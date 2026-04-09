# 容器安全

Docker、Kubernetes 等容器技术的安全测试。

## Docker 安全

### Docker API
- 未授权访问
- 远程代码执行
- 容器管理
- 镜像操作

### 容器逃逸
- 特权容器
- 挂载 Docker Socket
- 内核漏洞
- cgroup 逃逸
- Capabilities 滥用

### 镜像安全
- 镜像扫描
- 漏洞检测
- 恶意镜像
- 供应链攻击

### 配置安全
- 不安全的配置
- 敏感信息泄露
- 网络配置
- 存储配置

## Kubernetes 安全

### 信息收集
- API Server 探测
- 节点信息
- Pod 信息
- Service 信息
- Secret 和 ConfigMap

### 未授权访问
- API Server 未授权
- Dashboard 未授权
- Kubelet 未授权
- etcd 未授权

### RBAC 配置
- 权限配置错误
- 角色绑定
- 服务账户
- 权限提升

### 容器逃逸
- 特权 Pod
- hostPath 挂载
- hostNetwork/hostPID
- 内核漏洞

### 网络安全
- Network Policy
- Service 暴露
- Ingress 配置
- 东西向流量

### Secret 管理
- Secret 泄露
- ConfigMap 泄露
- 环境变量
- 挂载文件

## 容器运行时

### containerd
- 配置安全
- 运行时漏洞
- 逃逸技术

### CRI-O
- 配置安全
- 运行时漏洞

## 工具

### 扫描工具
- Trivy
- Clair
- Anchore
- Grype

### 安全工具
- Falco
- Aqua Security
- Sysdig

### 渗透工具
- kubectl
- kube-hunter
- kube-bench
- Peirates

---

容器安全需要理解容器技术的原理和 Kubernetes 的架构。
