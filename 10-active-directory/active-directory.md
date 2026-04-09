# Active Directory 专项

Active Directory 是企业网络的核心，也是内网渗透的重点目标。

## 信息收集

### 域信息
- 域名和域控位置
- 域功能级别
- 域信任关系
- 站点和子网
- 组织单元（OU）

### 用户和组
- 域用户枚举
- 域管理员
- 特权组成员
- 服务账户
- 用户属性

### 计算机
- 域计算机列表
- 服务器和工作站
- 操作系统版本
- 最后登录时间

### 策略
- 组策略（GPO）
- 密码策略
- 审计策略
- 安全策略

## Kerberos 攻击

### AS-REP Roasting
- 禁用预认证的用户
- 请求 AS-REP
- 离线破解哈希

### Kerberoasting
- 服务主体名称（SPN）枚举
- 请求服务票据（TGS）
- 离线破解服务账户密码

### Golden Ticket
- 获取 krbtgt 哈希
- 伪造 TGT
- 域内任意用户权限
- 持久化访问

### Silver Ticket
- 获取服务账户哈希
- 伪造 TGS
- 特定服务访问
- 更隐蔽的攻击

### Overpass-the-Hash
- NTLM 哈希转 Kerberos 票据
- 请求 TGT
- 使用票据认证

### Pass-the-Ticket
- 导出 Kerberos 票据
- 注入票据
- 使用票据访问资源

## NTLM 攻击

### NTLM 中继
- SMB 中继
- HTTP 中继
- LDAP 中继
- 跨协议中继

### LLMNR/NBT-NS 投毒
- 响应欺骗
- 凭证捕获
- 中继攻击

### Pass-the-Hash
- 获取 NTLM 哈希
- 使用哈希认证
- 横向移动

## 域权限提升

### ACL 滥用
- GenericAll
- GenericWrite
- WriteDacl
- WriteOwner
- ForceChangePassword

### 委派攻击
- 非约束委派
- 约束委派
- 基于资源的约束委派（RBCD）

### GPO 滥用
- GPO 权限
- 计划任务
- 启动脚本
- 注册表修改

### LAPS 攻击
- LAPS 密码读取
- LAPS 权限滥用

### 证书服务攻击
- ESC1-ESC8 漏洞
- 证书模板滥用
- 证书请求

## 域控攻击

### DCSync
- 复制域控数据
- 获取所有用户哈希
- 需要特定权限

### DCShadow
- 临时域控
- 修改 AD 对象
- 隐蔽的持久化

### Zerologon
- CVE-2020-1472
- 域控权限获取
- 密码重置

### PrintNightmare
- CVE-2021-34527
- 远程代码执行
- 权限提升

## 域信任攻击

### 信任关系
- 父子域信任
- 林信任
- 外部信任
- 快捷方式信任

### SID History
- SID 历史注入
- 跨域权限
- Enterprise Admins

### 信任票据
- 跨域票据
- 林间攻击
- 信任密钥

## 持久化

### 域级持久化
- Golden Ticket
- Silver Ticket
- AdminSDHolder
- DSRM 密码
- SID History
- 恶意 GPO

### 隐蔽后门
- ACL 后门
- 委派后门
- SPN 后门
- 证书后门

## 防御规避

### 日志清理
- 事件日志
- PowerShell 日志
- Sysmon 日志
- 审计日志

### 检测规避
- 避免触发告警
- 使用合法工具
- 时间窗口选择
- 流量混淆

## 工具

### 信息收集
- BloodHound
- PowerView
- ADRecon
- ldapdomaindump

### 攻击工具
- Mimikatz
- Rubeus
- Impacket
- CrackMapExec
- PowerSploit

---

AD 攻击需要深入理解 Kerberos 和 NTLM 认证机制，以及 AD 的权限模型和信任关系。
