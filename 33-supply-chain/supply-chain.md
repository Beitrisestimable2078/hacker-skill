# 供应链安全

软件供应链攻击，包括依赖投毒、CI/CD 攻击等。

## 依赖攻击

### 依赖投毒
- npm/PyPI/RubyGems 投毒
- 恶意包上传
- 包名抢注（Typosquatting）
- 依赖混淆

### 依赖劫持
- 账户接管
- 包维护者攻击
- 域名过期

## CI/CD 攻击

### CI/CD 平台
- Jenkins
- GitLab CI
- GitHub Actions
- Travis CI
- CircleCI

### 攻击向量
- 配置文件注入
- Secret 泄露
- 权限提升
- 代码注入
- 构建劫持

## 源码仓库

### Git 攻击
- .git 目录泄露
- Git Hook 劫持
- Commit 签名伪造
- 分支保护绕过

### 代码审查
- Pull Request 攻击
- 代码审查绕过
- 恶意代码隐藏

## 构建系统

### 构建工具
- Maven
- Gradle
- npm
- pip
- Cargo

### 攻击技术
- 构建脚本注入
- 依赖下载劫持
- 构建环境污染
- 缓存投毒

## 软件分发

### 更新机制
- 更新劫持
- 中间人攻击
- 签名绕过
- 降级攻击

### 镜像仓库
- Docker Hub
- 私有镜像仓库
- 镜像投毒
- 镜像劫持

## 开源组件

### 漏洞管理
- 已知漏洞（CVE）
- 依赖扫描
- SCA（Software Composition Analysis）
- SBOM（Software Bill of Materials）

### 工具
- Snyk
- Dependabot
- OWASP Dependency-Check
- Trivy

---

供应链安全需要关注整个软件开发和分发流程的每个环节。
