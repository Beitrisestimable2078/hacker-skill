# 数据库安全

数据库是存储核心数据的系统，是渗透测试的重要目标。

## 数据库类型

### 关系型数据库
- MySQL/MariaDB
- Microsoft SQL Server
- Oracle
- PostgreSQL
- SQLite

### NoSQL 数据库
- MongoDB
- Redis
- CouchDB
- Cassandra
- Elasticsearch

## 攻击向量

### 未授权访问
- 默认凭证
- 弱密码
- 空密码
- 匿名访问

### 注入攻击
- SQL 注入
- NoSQL 注入
- ORM 注入
- 存储过程注入

### 权限提升
- 数据库用户提权
- 操作系统命令执行
- UDF 利用
- 存储过程滥用

## MySQL/MariaDB

### 信息收集
- 版本识别
- 用户枚举
- 权限查询
- 数据库结构

### 攻击技术
- UDF 提权
- MOF 提权
- 文件读写（load_file、into outfile）
- 日志文件写入
- 慢查询日志

### 后门技术
- 触发器后门
- 存储过程后门
- UDF 后门
- 账户后门

## Microsoft SQL Server

### 信息收集
- 版本和补丁
- 实例枚举
- 链接服务器
- 数据库角色

### 攻击技术
- xp_cmdshell 命令执行
- sp_oacreate COM 对象
- CLR 程序集
- Agent Job 利用

### 权限提升
- 模拟登录（EXECUTE AS）
- 链接服务器提权
- 服务账户提权

## Oracle

### 信息收集
- SID 枚举
- 用户和角色
- 权限查询
- TNS 监听器

### 攻击技术
- TNS 投毒
- Java 存储过程
- DBMS_SCHEDULER
- UTL_HTTP/UTL_FILE

### 权限提升
- DBA 权限获取
- SYSDBA 提权
- 操作系统认证

## PostgreSQL

### 信息收集
- 版本和配置
- 用户和角色
- 扩展和函数

### 攻击技术
- COPY 命令文件读写
- 大对象利用
- 自定义函数
- 扩展利用

### 权限提升
- 超级用户提权
- 命令执行
- 文件系统访问

## Redis

### 未授权访问
- 无密码访问
- 弱密码爆破
- 协议利用

### 攻击技术
- 写 SSH 公钥
- 写 Crontab
- 写 Webshell
- 主从复制 RCE
- 模块加载

## MongoDB

### 未授权访问
- 无认证访问
- 默认配置
- 端口暴露

### 攻击技术
- NoSQL 注入
- JavaScript 注入
- 数据导出
- 权限提升

## 数据提取

### 敏感数据
- 用户凭证
- 个人信息
- 业务数据
- 配置信息

### 数据导出
- SQL 查询导出
- 备份文件
- 日志文件
- 二进制数据

## 持久化

### 后门植入
- 数据库账户
- 触发器
- 存储过程
- UDF 函数
- 定时任务

### 隐蔽技术
- 隐藏账户
- 权限维持
- 日志清理

---

数据库渗透需要深入了解各类数据库的特性和安全机制，灵活运用各种攻击技术。
