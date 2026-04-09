# 云安全

云平台的安全测试，包括 AWS、Azure、阿里云、腾讯云等，以及云原生安全。

## 云平台通用

### 信息收集
- 云服务识别
- 资源枚举
- 配置信息
- 权限信息
- 网络拓扑

### 常见漏洞
- 配置错误
- 权限过大
- 未授权访问
- 数据泄露
- API 滥用

## AWS 安全

### 信息收集
- S3 Bucket 枚举
- EC2 实例
- Lambda 函数
- RDS 数据库
- IAM 用户和角色

### 元数据服务
- IMDS v1/v2
- 凭证获取
- 角色窃取
- SSRF 利用

### S3 安全
- 公开 Bucket
- ACL 配置错误
- 策略配置错误
- 数据泄露

### IAM 安全
- 权限提升
- 角色假设
- 策略滥用
- 凭证泄露

## Azure 安全

### 信息收集
- 订阅枚举
- 资源组
- 虚拟机
- 存储账户
- Azure AD

### 元数据服务
- IMDS
- 托管标识
- 凭证获取

### 存储安全
- Blob 存储
- 文件共享
- 队列存储
- 表存储

## 阿里云/腾讯云

### 信息收集
- OSS/COS Bucket
- ECS/CVM 实例
- RDS 数据库
- RAM/CAM 权限

### 元数据服务
- 元数据 API
- 凭证获取
- 角色信息

## 云原生安全

### Kubernetes
- API Server 未授权
- Dashboard 暴露
- RBAC 配置错误
- 容器逃逸
- Secret 泄露

### Docker
- Docker API 未授权
- 镜像漏洞
- 容器逃逸
- 特权容器

### Service Mesh
- Istio
- Linkerd
- 配置错误
- 流量劫持

### Serverless
- Lambda/函数计算
- 权限配置
- 事件注入
- 依赖漏洞

## 云工具

### 枚举工具
- CloudMapper
- ScoutSuite
- Prowler
- Pacu

### 利用工具
- CloudGoat
- Stratus Red Team
- Leonidas

---

云安全需要深入理解各云平台的架构和安全机制。
