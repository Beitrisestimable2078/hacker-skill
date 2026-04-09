# Windows 安全

Windows 系统的安全测试、权限提升、持久化等技术。

## 信息收集

### 系统信息
- 操作系统版本和补丁
- 系统架构
- 安装的软件
- 运行的服务
- 网络配置
- 防护软件

### 用户和权限
- 当前用户权限
- 本地用户和组
- 域用户信息
- 权限配置
- Token 信息

## 权限提升

### 系统漏洞
- 内核漏洞
- 补丁缺失
- Exploit 利用
- 稳定性考虑

### 配置错误
- 不安全的服务权限
- 不安全的文件权限
- 注册表权限
- 计划任务劫持
- DLL 劫持
- 路径劫持

### Token 操纵
- Token 窃取
- Token 模拟
- 进程注入
- PPID Spoofing

### UAC 绕过
- 白名单程序
- COM 接口
- 注册表劫持
- DLL 劫持
- 环境变量

## 凭证获取

### 内存凭证
- LSASS dump
- Mimikatz
- Procdump
- Comsvcs.dll

### 注册表凭证
- SAM 数据库
- LSA Secrets
- Cached Credentials
- DPAPI

### 文件凭证
- 配置文件
- 脚本文件
- 浏览器密码
- WiFi 密码
- RDP 凭证

## 持久化

### 启动项
- 注册表启动项
- 启动文件夹
- 计划任务
- 服务
- WMI 事件订阅

### 劫持技术
- DLL 劫持
- COM 劫持
- 映像劫持
- 快捷方式劫持
- Accessibility Features

### 后门账户
- 隐藏账户
- 克隆账户
- RID 劫持

## Active Directory

### 域渗透
- Kerberos 攻击
- NTLM 中继
- ACL 滥用
- GPO 滥用
- 委派攻击

### 域权限提升
- DCSync
- DCShadow
- Zerologon
- PrintNightmare

## 横向移动

### 远程执行
- PsExec
- WMI
- DCOM
- WinRM
- PowerShell Remoting
- SC
- AT/SCHTASKS

### 凭证传递
- Pass-the-Hash
- Pass-the-Ticket
- Overpass-the-Hash

## 防御绕过

### 杀软绕过
- 静态检测绕过
- 行为检测绕过
- 内存扫描绕过
- AMSI 绕过

### EDR 绕过
- Hook 绕过
- ETW 绕过
- 日志清理
- 进程保护

## PowerShell

### PowerShell 攻击
- PowerShell Empire
- PowerSploit
- Nishang
- Invoke-Obfuscation

### PowerShell 绕过
- Execution Policy 绕过
- AMSI 绕过
- Constrained Language Mode 绕过
- AppLocker 绕过

---

Windows 安全需要深入理解 Windows 内部机制和安全特性。
