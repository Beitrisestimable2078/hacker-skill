# Webshell 免杀

Webshell 是 Web 渗透中的重要工具，免杀技术能够绕过 WAF、杀软和安全检测。

## 免杀原则

### 核心思路
- 避免使用明显的危险函数
- 使用间接调用和回调函数
- 字符串拆分和动态拼接
- 编码和加密混淆
- 业务逻辑伪装

### 检测机制
- 静态特征检测（关键字、函数名、代码结构）
- 动态行为检测（函数调用、文件操作、网络连接）
- 语义分析（AST 分析、污点追踪）
- 机器学习检测

## PHP Webshell

### 危险函数
需要避免或混淆的函数：
- 命令执行：system、exec、shell_exec、passthru、popen、proc_open
- 代码执行：eval、assert、create_function、preg_replace /e
- 文件操作：file_put_contents、file_get_contents、fwrite

### 替代方案
- 回调函数：array_map、array_filter、usort、uasort、array_walk
- 可变函数：$func()
- 反射：ReflectionFunction、ReflectionMethod
- 文件函数组合：fopen + fwrite + fclose

### 混淆技术
- 字符串拆分：str_rot13、base64、异或、字符串拼接
- 变量函数：动态函数名
- 类和对象：封装在类中
- 编码变换：多层编码嵌套

### 伪装技术
- 业务逻辑伪装（日志类、缓存类、配置类）
- 正常功能混合
- 大量无用代码
- 注释和文档

## JSP Webshell

### 危险类
- Runtime.getRuntime().exec()
- ProcessBuilder
- ScriptEngineManager
- ClassLoader

### 替代方案
- 反射调用
- 表达式引擎
- JNI 调用
- JNDI 注入

### 混淆技术
- 类名和方法名混淆
- 字符串加密
- 字节码混淆
- 动态类加载

## ASP/ASPX Webshell

### 危险函数
- eval
- execute
- CreateObject("WScript.Shell")
- Process.Start

### 替代方案
- 反射调用
- 动态编译
- COM 对象
- PowerShell 调用

### 混淆技术
- 字符串编码
- 变量名混淆
- 代码加密
- 动态生成

## 内存马

### 类型
- Filter 型内存马
- Servlet 型内存马
- Listener 型内存马
- Agent 型内存马
- WebSocket 型内存马

### 注入方式
- 反序列化注入
- 文件上传注入
- JNDI 注入
- 表达式注入

### 隐蔽技术
- 无文件落地
- 动态注册
- 进程注入
- 内存加密

## 流量加密

### 通信加密
- 对称加密（AES、DES、RC4）
- 非对称加密（RSA）
- 自定义加密算法
- 流量混淆

### 协议伪装
- 正常 HTTP 请求
- Cookie 隧道
- Header 隧道
- WebSocket 通信

## 检测绕过

### 静态检测绕过
- 关键字变换
- 代码结构变换
- 编码混淆
- 加密保护

### 动态检测绕过
- 延迟执行
- 条件触发
- 环境检测
- 沙箱对抗

### WAF 绕过
- 请求方法变换
- 参数污染
- 编码绕过
- 分块传输

## 工具与生成

根据目标环境和检测强度，生成不同混淆级别的 Webshell：
- 轻度混淆：基本的字符串拆分和函数替换
- 中度混淆：多层编码和回调函数
- 重度混淆：完全加密和业务伪装
- 内存马：无文件落地的内存驻留

---

Webshell 免杀需要不断更新技术，对抗不断升级的检测机制。
