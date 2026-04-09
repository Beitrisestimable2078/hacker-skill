# 中间件与框架安全

中间件和框架是 Web 应用的基础组件，存在大量已知漏洞和配置问题。

## Web 服务器

### Apache
- 解析漏洞
- .htaccess 利用
- 目录遍历
- CGI 漏洞
- mod_rewrite 绕过

### Nginx
- 解析漏洞
- 配置错误
- 目录穿越
- CRLF 注入
- 整数溢出

### IIS
- 短文件名泄露
- PUT 漏洞
- 解析漏洞
- WebDAV 利用
- 权限配置错误

### Tomcat
- 管理后台弱口令
- 文件上传
- 远程代码执行
- AJP 协议漏洞
- 信息泄露

## 应用服务器

### WebLogic
- 反序列化漏洞
- SSRF 漏洞
- 任意文件上传
- 弱密码和默认密码
- T3 协议漏洞

### JBoss
- JMX Console 未授权访问
- 反序列化漏洞
- 文件上传
- 远程代码执行

### WebSphere
- 控制台弱口令
- 反序列化漏洞
- 信息泄露
- SSRF 漏洞

### GlassFish
- 管理控制台
- 远程代码执行
- 目录遍历
- 信息泄露

## Web 框架

### Spring
- Spring Boot Actuator 信息泄露
- SpEL 表达式注入
- Spring Cloud Gateway RCE
- Spring Data REST 漏洞
- 反序列化漏洞

### Struts2
- OGNL 表达式注入
- 远程代码执行
- 文件上传绕过
- S2 系列漏洞

### Django
- Debug 模式信息泄露
- SQL 注入
- SSTI 模板注入
- 反序列化漏洞
- CSRF 绕过

### Flask
- Debug 模式 RCE
- SSTI 模板注入
- Session 伪造
- 配置错误

### Laravel
- Debug 模式信息泄露
- 反序列化漏洞
- SQL 注入
- 文件包含

### ThinkPHP
- 远程代码执行
- SQL 注入
- 文件包含
- 目录遍历

## 容器与编排

### Docker
- 未授权访问
- 容器逃逸
- 镜像漏洞
- 特权容器

### Kubernetes
- API Server 未授权
- Dashboard 暴露
- RBAC 配置错误
- 容器逃逸

## 消息队列

### RabbitMQ
- 管理后台弱口令
- 未授权访问
- Erlang Cookie 泄露

### Kafka
- 未授权访问
- JMX 暴露
- ZooKeeper 未授权

### ActiveMQ
- 管理后台弱口令
- 反序列化漏洞
- 文件上传

## 缓存系统

### Redis
- 未授权访问
- 弱密码
- 主从复制 RCE
- 模块加载

### Memcached
- 未授权访问
- SSRF 利用
- 数据泄露

## 搜索引擎

### Elasticsearch
- 未授权访问
- 代码执行
- 目录遍历
- 信息泄露

### Solr
- 未授权访问
- 远程代码执行
- XXE 漏洞
- 文件读取

## 漏洞利用

### 信息收集
- 版本识别
- 指纹识别
- 配置信息
- 默认路径

### 漏洞检测
- 已知 CVE 漏洞
- 配置错误
- 弱口令
- 未授权访问

### 漏洞利用
- 远程代码执行
- 文件上传
- 任意文件读取
- 权限提升

---

中间件和框架漏洞通常影响范围广，需要及时关注最新的漏洞公告和补丁。
