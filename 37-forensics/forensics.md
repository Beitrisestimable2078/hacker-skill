# 数字取证

数字取证技术，包括磁盘取证、内存取证、网络取证、反取证等。

## 磁盘取证

### 镜像获取
- dd/dcfldd
- FTK Imager
- EnCase
- 写保护设备

### 文件系统分析
- NTFS/FAT/EXT/HFS+
- 文件恢复
- 删除文件恢复
- 时间线分析

### 工具
- Autopsy
- Sleuth Kit
- FTK
- EnCase

## 内存取证

### 内存获取
- DumpIt
- FTK Imager
- WinPmem
- LiME（Linux）

### 内存分析
- Volatility
- Rekall
- 进程分析
- 网络连接
- 注册表 Hive
- 恶意代码检测

## 网络取证

### 流量捕获
- Wireshark
- tcpdump
- NetworkMiner

### 流量分析
- 协议分析
- 会话重组
- 文件提取
- 恶意流量识别

## Windows 取证

### 注册表分析
- 用户活动
- 程序执行
- USB 设备
- 网络配置

### 事件日志
- 系统日志
- 安全日志
- 应用日志
- PowerShell 日志

### 工件分析
- Prefetch
- ShimCache
- AmCache
- Jump Lists
- LNK 文件
- USN Journal

## Linux 取证

### 日志分析
- /var/log/
- auth.log
- syslog
- 审计日志

### 工件分析
- .bash_history
- cron 任务
- systemd 日志
- 用户活动

## 移动取证

### Android 取证
- ADB 提取
- 逻辑提取
- 物理提取
- 应用数据

### iOS 取证
- iTunes 备份
- 逻辑提取
- 物理提取
- Keychain

## 反取证

### 数据销毁
- 安全删除
- 覆盖写入
- 物理销毁

### 反内存取证
- 内存加密
- 内存清理
- 冷启动攻击防护

### 反磁盘取证
- 全盘加密
- 隐藏卷
- 可否认加密
- 时间戳伪造

### 日志清理
- 事件日志清理
- 命令历史清理
- 文件时间戳修改
- 工件清理

## 时间线分析

### 时间线构建
- 文件系统时间线
- 注册表时间线
- 事件日志时间线
- 网络活动时间线

### 工具
- log2timeline/plaso
- Timesketch

---

数字取证需要系统的方法论和对操作系统的深入理解。
