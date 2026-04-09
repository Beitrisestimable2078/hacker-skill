# 内存马

内存马是一种无文件落地的Webshell，驻留在内存中，隐蔽性极强。

## 内存马类型

### Filter型内存马
- 注册恶意Filter
- 拦截所有HTTP请求
- 优先级可控
- 适用于Servlet容器

### Servlet型内存马
- 动态注册Servlet
- 创建新的URL映射
- 独立的访问路径
- 持久化访问

### Listener型内存马
- 注册ServletRequestListener
- 监听请求生命周期
- 隐蔽性最强
- 难以检测

### Valve型内存马（Tomcat）
- Tomcat特有
- Pipeline机制
- 优先级最高
- 拦截所有请求

### Agent型内存马
- Java Agent技术
- 字节码注入
- 全局拦截
- 最强隐蔽性

## 注入方式

### 反序列化注入
```java
// 通过反序列化漏洞
// 注入内存马代码
// 无需文件上传
```

### 文件上传注入
```java
// 上传JSP文件
// 执行注入代码
// 删除JSP文件
// 仅留内存马
```

### JNDI注入
```java
// 通过JNDI注入
// 远程加载恶意类
// 注入内存马
```

### 表达式注入
```java
// SpEL、OGNL等
// 执行注入代码
// 动态注册
```

## Filter型内存马实现

### 基本结构
```java
// 创建恶意Filter
// 实现doFilter方法
// 动态注册到容器
// 设置URL映射
```

### 注册方法
```java
// Tomcat
ServletContext servletContext = request.getServletContext();
FilterRegistration.Dynamic filterRegistration = 
    servletContext.addFilter("evilFilter", new EvilFilter());
filterRegistration.addMappingForUrlPatterns(...);

// Spring Boot
// 通过ApplicationContext注册
```

### 优先级控制
```java
// 设置Filter顺序
// 确保优先执行
// 避免被其他Filter拦截
```

## Servlet型内存马实现

### 基本结构
```java
// 创建恶意Servlet
// 实现service方法
// 动态注册
// 设置URL映射
```

### 注册方法
```java
// 获取ServletContext
// 调用addServlet
// 设置URL映射
// 启动Servlet
```

## Listener型内存马实现

### 基本结构
```java
// 实现ServletRequestListener
// 在requestInitialized中处理
// 动态注册
// 监听所有请求
```

### 隐蔽性
```java
// 不创建新的URL
// 不修改现有映射
// 难以发现
// 难以清除
```

## Valve型内存马（Tomcat）

### Pipeline机制
```java
// Tomcat的Pipeline机制
// 添加自定义Valve
// 最高优先级
// 拦截所有请求
```

### 注入方法
```java
// 获取StandardContext
// 获取Pipeline
// 添加自定义Valve
// 设置优先级
```

## 通信方式

### HTTP Header
```java
// 通过特定Header传递命令
// 响应也通过Header返回
// 隐蔽性强
```

### Cookie
```java
// 通过Cookie传递命令
// 加密Cookie内容
// 正常流量伪装
```

### POST Body
```java
// 通过POST数据传递
// 加密传输
// 支持大数据量
```

### 自定义协议
```java
// 自定义请求格式
// 自定义响应格式
// 完全控制
```

## 加密通信

### AES加密
```java
// 对称加密
// 密钥协商
// 快速加解密
```

### RSA加密
```java
// 非对称加密
// 密钥交换
// 安全性高
```

## 持久化

### 定时任务
```java
// 创建定时任务
// 定期检查
// 自动恢复
```

### 多重注册
```java
// 注册多个内存马
// 互相备份
// 提高存活率
```

## 检测对抗

### 反调试
```java
// 检测调试器
// 检测分析工具
// 自毁机制
```

### 环境检测
```java
// 检测沙箱
// 检测虚拟机
// 条件触发
```

### 混淆保护
```java
// 代码混淆
// 字符串加密
// 控制流混淆
```

## 清除方法

### 重启应用
```
// 最简单的方法
// 内存马失效
// 但影响业务
```

### 动态清除
```java
// 遍历Filter/Servlet
// 识别恶意组件
// 动态移除
```

### 内存dump分析
```
// dump内存
// 分析恶意代码
// 定位注入点
```

## 防御建议

### 输入验证
- 严格的输入验证
- 防止注入攻击
- 限制反序列化

### 运行时监控
- 监控动态注册
- 检测异常Filter/Servlet
- 告警机制

### 安全加固
- 禁用不必要的功能
- 限制反射使用
- 代码签名验证

---

内存马是高级攻击技术，需要深入理解Java Web容器的内部机制。
