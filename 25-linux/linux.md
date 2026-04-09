# Linux 安全

Linux 系统的安全测试、权限提升、持久化等技术。

## 信息收集

### 系统信息
- 发行版和版本
- 内核版本
- 系统架构
- 安装的软件
- 运行的服务
- 网络配置

### 用户和权限
- 当前用户
- /etc/passwd 和 /etc/shadow
- sudo 配置
- SUID/SGID 程序
- Capabilities

## 权限提升

### 内核漏洞
- 内核版本检测
- 已知漏洞利用
- Dirty COW
- 其他内核漏洞

### SUID/SGID
- SUID 程序查找
- 可利用的 SUID 程序
- GTFOBins
- 自定义利用

### Sudo 配置
- sudo -l 检查
- sudo 漏洞
- 配置错误利用
- LD_PRELOAD
- LD_LIBRARY_PATH

### Cron 任务
- 定时任务查找
- 路径劫持
- 通配符注入
- 权限配置错误

### Capabilities
- getcap 查找
- 能力滥用
- 权限提升

### 其他方法
- NFS 配置错误
- Docker 逃逸
- LXC/LXD 提权
- 环境变量

## 凭证获取

### 密码文件
- /etc/shadow
- /etc/passwd
- 备份文件
- 历史文件

### SSH 密钥
- ~/.ssh/id_rsa
- ~/.ssh/authorized_keys
- /etc/ssh/
- 其他用户的密钥

### 配置文件
- 数据库配置
- 应用配置
- 脚本文件
- 环境变量

### 历史命令
- .bash_history
- .zsh_history
- .mysql_history
- 其他历史文件

## 持久化

### 启动项
- /etc/rc.local
- systemd 服务
- cron 任务
- .bashrc/.profile
- /etc/profile.d/

### SSH 后门
- authorized_keys
- SSH 配置
- PAM 后门
- SSH wrapper

### 其他后门
- LD_PRELOAD
- 内核模块
- Rootkit
- 定时任务

## 容器逃逸

### Docker 逃逸
- 特权容器
- 挂载 Docker Socket
- 内核漏洞
- cgroup 逃逸

### 其他容器
- LXC/LXD
- Podman
- containerd

## 日志清理

### 日志文件
- /var/log/auth.log
- /var/log/syslog
- /var/log/messages
- 应用日志

### 命令历史
- .bash_history
- .zsh_history
- 其他历史文件

### 其他痕迹
- lastlog
- wtmp/utmp
- 审计日志

---

Linux 安全需要熟悉 Linux 系统机制和常见的配置错误。
