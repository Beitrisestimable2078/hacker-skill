# macOS 安全

macOS 系统的安全测试、权限提升、持久化等技术。

## 信息收集

### 系统信息
- macOS 版本
- 内核版本
- 系统架构
- 安装的应用
- 运行的服务
- 安全特性（SIP、Gatekeeper、XProtect）

### 用户和权限
- 当前用户
- 管理员用户
- sudo 配置
- 权限配置

## 权限提升

### 系统漏洞
- 内核漏洞
- 已知漏洞利用
- 补丁缺失

### 配置错误
- sudo 配置错误
- SUID 程序
- 文件权限
- 服务权限

### TCC 绕过
- Transparency, Consent, and Control
- 权限绕过
- 数据库操纵

### SIP 绕过
- System Integrity Protection
- 绕过技术
- 内核扩展

## 凭证获取

### Keychain
- Keychain 访问
- 密码提取
- 证书提取

### 配置文件
- 应用配置
- 数据库配置
- 脚本文件

### 浏览器数据
- Safari
- Chrome
- Firefox

## 持久化

### Launch Agents/Daemons
- ~/Library/LaunchAgents
- /Library/LaunchAgents
- /Library/LaunchDaemons
- /System/Library/LaunchDaemons

### 登录项
- Login Items
- 启动应用

### 其他方法
- .bash_profile/.zshrc
- Cron 任务
- Kernel Extensions
- 应用劫持

## 沙箱逃逸

### 应用沙箱
- 沙箱机制
- 逃逸技术
- 权限提升

### 浏览器沙箱
- Safari 沙箱
- Chrome 沙箱
- 逃逸漏洞

---

macOS 安全需要理解 Apple 的安全架构和保护机制。
