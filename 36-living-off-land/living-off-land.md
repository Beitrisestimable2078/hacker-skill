# 白利用技术

Living off the Land - 利用系统自带工具和合法程序进行攻击。

## Windows LOLBins

### 常用工具
- PowerShell
- cmd.exe
- wmic.exe
- mshta.exe
- regsvr32.exe
- rundll32.exe
- certutil.exe
- bitsadmin.exe

### 文件下载
- certutil -urlcache
- bitsadmin /transfer
- PowerShell Invoke-WebRequest
- mshta vbscript:Download

### 代码执行
- PowerShell -EncodedCommand
- mshta javascript/vbscript
- regsvr32 /u /n /s /i:http://
- rundll32 javascript:

### 持久化
- schtasks
- sc.exe
- reg.exe
- wmic

### 凭证获取
- reg save
- vaultcmd
- cmdkey

## Linux 工具

### 常用工具
- bash/sh
- python/perl/ruby
- curl/wget
- nc/netcat
- ssh
- cron

### 文件下载
- curl/wget
- scp/sftp
- nc
- /dev/tcp

### 代码执行
- bash -c
- python -c
- perl -e
- awk/sed

### 持久化
- crontab
- systemd
- .bashrc/.profile

## 合法工具滥用

### 系统管理工具
- PsExec
- WinRM
- WMI
- PowerShell Remoting

### 远程工具
- TeamViewer
- AnyDesk
- RDP
- VNC

### 开发工具
- Visual Studio
- MSBuild
- InstallUtil
- RegAsm

## GTFOBins

### 文件读取
- less/more/cat
- tail/head
- vi/vim/nano

### 文件写入
- tee
- dd
- cp

### SUID 利用
- find
- nmap
- vim
- python

---

白利用技术能够规避检测，因为使用的都是系统合法工具。
