# 代码审计

源代码安全审计是通过分析源代码发现安全漏洞的技术。相比黑盒测试，白盒审计能够发现更深层次的逻辑漏洞和隐蔽的安全问题。

## 子模块导航

本模块包含以下深度子模块：

### 语言专项审计
- [Java代码审计](java-audit.md) - Java/Spring/Struts2框架审计，反序列化，JNDI注入
- [PHP代码审计](php-audit.md) - PHP/ThinkPHP/Laravel审计，文件操作，反序列化
- [Python代码审计](python-audit.md) - Python/Django/Flask审计，SSTI，Pickle反序列化
- [Node.js代码审计](nodejs-audit.md) - Node.js/Express审计，原型链污染，npm安全

### 专项审计
- [业务逻辑审计](business-logic-audit.md) - 支付/优惠券/积分/权限/状态机等业务漏洞
- [API安全审计](api-security-audit.md) - REST/GraphQL/gRPC API安全，认证授权，速率限制

### 工具与流程
- [静态分析工具](static-analysis-tools.md) - SonarQube/CodeQL/Semgrep等工具实践
- [代码审查流程](code-review-process.md) - 系统化审查方法论与最佳实践

## 核心概念

### 污点分析 (Taint Analysis)
追踪不可信数据从输入点到危险函数的传播路径：
- **污点源 (Source)**：用户输入、外部数据
- **污点传播 (Propagation)**：数据流动和转换
- **污点汇聚 (Sink)**：危险函数、敏感操作
- **净化函数 (Sanitizer)**：过滤、验证、编码

### 数据流分析
- **前向分析**：从输入点追踪到危险函数
- **后向分析**：从危险函数反推输入来源
- **双向分析**：结合前向和后向分析

### 控制流分析
- 分析程序执行路径
- 识别条件分支
- 发现死代码
- 检测逻辑漏洞

## 审计方法

### 手工审计
**适用场景**：
- 复杂业务逻辑
- 框架特定漏洞
- 0day挖掘
- 深度安全评估

**方法**：
- 代码走查
- 关键路径分析
- 危险函数定位
- 数据流追踪

### 自动化审计
**适用场景**：
- 大规模代码库
- 持续集成
- 已知漏洞模式
- 快速初筛

**方法**：
- 静态分析工具
- 模式匹配
- AST分析
- 符号执行

### 混合审计
结合手工和自动化的优势：
- 工具初筛 → 人工验证
- 人工发现 → 工具扩展
- 迭代优化审计规则

## 审计维度

### 按漏洞类型
- 注入类：SQL/命令/代码/LDAP/XPath/NoSQL
- XSS类：反射/存储/DOM/SSTI
- 文件类：上传/包含/下载/穿越
- 反序列化：Java/PHP/Python/.NET
- 逻辑类：认证/授权/业务/竞态

### 按技术栈
- 前端：JavaScript/TypeScript/React/Vue
- 后端：Java/PHP/Python/Node.js/Go/Rust
- 移动：Android/iOS/Flutter/React Native
- 框架：Spring/Django/Laravel/Express

### 按安全域
- 输入验证
- 输出编码
- 认证授权
- 会话管理
- 加密存储
- 错误处理
- 日志审计

## 漏洞分类体系

### OWASP Top 10 视角
1. **Broken Access Control** - 访问控制失效
2. **Cryptographic Failures** - 加密机制失效
3. **Injection** - 注入攻击
4. **Insecure Design** - 不安全设计
5. **Security Misconfiguration** - 安全配置错误
6. **Vulnerable Components** - 易受攻击的组件
7. **Authentication Failures** - 认证机制失效
8. **Data Integrity Failures** - 数据完整性失效
9. **Logging Failures** - 日志和监控失效
10. **SSRF** - 服务端请求伪造

### CWE 常见弱点
- **CWE-79**: XSS跨站脚本
- **CWE-89**: SQL注入
- **CWE-78**: OS命令注入
- **CWE-22**: 路径穿越
- **CWE-502**: 不安全的反序列化
- **CWE-94**: 代码注入
- **CWE-611**: XXE外部实体注入
- **CWE-798**: 硬编码凭证
- **CWE-918**: SSRF
- **CWE-352**: CSRF

### 注入类漏洞
**SQL注入**：
- 数字型/字符型/搜索型
- Union/Boolean/Time-based/Error-based
- 二次注入/宽字节注入
- ORM注入

**命令注入**：
- 直接命令执行
- 参数注入
- 环境变量注入
- 管道符利用

**代码注入**：
- eval/assert/create_function
- 模板注入(SSTI)
- 表达式注入(SpEL/OGNL/EL)
- 动态代码执行

**其他注入**：
- LDAP注入
- XPath注入
- NoSQL注入
- XML注入
- CRLF注入

### XSS类漏洞
**反射型XSS**：
- GET/POST参数
- HTTP头注入
- URL路径注入

**存储型XSS**：
- 数据库存储
- 文件存储
- 缓存存储

**DOM型XSS**：
- innerHTML/outerHTML
- document.write
- eval/setTimeout/setInterval
- location/src/href赋值

**高级XSS**：
- mXSS (Mutation XSS)
- UXSS (Universal XSS)
- Self-XSS利用
- XSS Worm

### 文件操作漏洞
**文件上传**：
- 类型检测绕过
- 文件名绕过
- 解析漏洞
- 竞态条件
- 二次渲染绕过

**文件包含**：
- 本地文件包含(LFI)
- 远程文件包含(RFI)
- 伪协议利用
- 日志包含
- Session文件包含

**路径穿越**：
- ../绕过
- 编码绕过
- 双写绕过
- 绝对路径

**文件操作**：
- 任意文件读取
- 任意文件删除
- 任意文件写入
- ZIP解压漏洞

### 反序列化漏洞
**Java**：
- Commons Collections
- Spring Framework
- Fastjson/Jackson
- JNDI注入
- RMI/LDAP利用

**PHP**：
- POP链构造
- Phar反序列化
- Session反序列化
- 魔术方法利用

**Python**：
- Pickle反序列化
- YAML反序列化
- Marshal反序列化

**.NET**：
- BinaryFormatter
- DataContractSerializer
- JavaScriptSerializer

### 逻辑漏洞
**认证授权**：
- 认证绕过
- 弱密码策略
- 会话固定
- 会话劫持
- JWT漏洞
- OAuth漏洞
- IDOR
- 越权访问

**业务逻辑**：
- 支付逻辑漏洞
- 优惠券滥用
- 积分/余额篡改
- 条件竞争
- 重放攻击
- 批量操作
- 数量负数
- 价格篡改

**加密问题**：
- 弱加密算法
- 硬编码密钥
- 不安全随机数
- ECB模式
- 密码明文存储
- 传输未加密

## 语言与框架特性

### Java生态
**框架**：
- Spring Boot/Cloud/Security
- Struts2
- MyBatis/Hibernate
- Shiro/JWT

**特性漏洞**：
- 反序列化RCE
- SpEL/OGNL表达式注入
- JNDI注入
- XXE
- Fastjson/Jackson反序列化
- Log4j2 (Log4Shell)

### PHP生态
**框架**：
- Laravel
- ThinkPHP
- Symfony
- CodeIgniter
- Yii

**特性漏洞**：
- 弱类型比较
- 变量覆盖
- 文件包含
- 反序列化
- 伪协议利用
- disable_functions绕过

### Python生态
**框架**：
- Django
- Flask
- FastAPI
- Tornado

**特性漏洞**：
- SSTI模板注入
- Pickle反序列化
- YAML反序列化
- ORM注入
- 命令注入

### Node.js生态
**框架**：
- Express
- Koa
- NestJS
- Fastify

**特性漏洞**：
- 原型链污染
- 命令注入
- NoSQL注入
- 路径穿越
- npm包漏洞
- JWT漏洞

### Go生态
**框架**：
- Gin
- Echo
- Beego
- Fiber

**特性漏洞**：
- SQL注入
- 命令注入
- 路径穿越
- SSRF
- 模板注入

### Rust生态
**框架**：
- Actix-web
- Rocket
- Warp

**安全特性**：
- 内存安全
- 类型安全
- 并发安全

**仍需关注**：
- 逻辑漏洞
- 不安全代码块
- 依赖漏洞

### C/C++
**常见漏洞**：
- 缓冲区溢出
- 格式化字符串
- UAF (Use After Free)
- 整数溢出
- 空指针解引用
- 双重释放

### .NET生态
**框架**：
- ASP.NET Core
- Entity Framework

**特性漏洞**：
- 反序列化
- XXE
- SQL注入
- ViewState利用

## 审计工具生态

### 商业工具
**SAST (静态应用安全测试)**：
- Checkmarx
- Fortify SCA
- Veracode
- Coverity
- Klocwork

**优势**：
- 深度分析能力
- 企业级支持
- 合规报告
- 误报率低

### 开源工具
**通用工具**：
- SonarQube - 代码质量和安全
- Semgrep - 模式匹配
- CodeQL - 语义分析
- LGTM - 持续安全分析

**语言特定**：
- Java: SpotBugs + FindSecBugs
- PHP: RIPS, Psalm, PHPStan
- Python: Bandit, Pyre
- JavaScript: ESLint Security, NodeJsScan
- Go: Gosec, StaticCheck
- Rust: Clippy, Cargo-audit

### IDE集成
**插件**：
- IntelliJ IDEA: Security Code Scan
- VS Code: Snyk, SonarLint
- Eclipse: SpotBugs
- PyCharm: Security插件

### 依赖检查
**工具**：
- OWASP Dependency-Check
- Snyk
- npm audit / yarn audit
- pip-audit / safety
- bundler-audit (Ruby)
- cargo-audit (Rust)

### 自定义工具
**技术栈**：
- 正则表达式搜索
- AST (抽象语法树) 分析
- 符号执行
- 污点分析引擎
- 模糊测试

**开发框架**：
- Tree-sitter - 通用AST解析
- Joern - 代码属性图
- Soot - Java分析框架
- LLVM - 编译器基础设施

## 高级审计技术

### 污点分析实践
**手工污点追踪**：
1. 标记所有输入点 (GET/POST/Cookie/Header)
2. 追踪数据流动路径
3. 识别过滤和验证函数
4. 定位危险函数调用
5. 评估可利用性

**自动化污点分析**：
- 静态污点分析工具
- 动态污点追踪
- 符号执行
- 混合分析

### 数据流分析
**前向切片**：
从输入点开始，追踪所有可能影响的代码

**后向切片**：
从危险点开始，反推所有可能的输入来源

**程序依赖图 (PDG)**：
- 控制依赖
- 数据依赖
- 调用依赖

### 符号执行
**概念**：
使用符号值代替具体值，探索所有可能的执行路径

**应用**：
- 路径覆盖
- 约束求解
- 漏洞触发条件生成
- PoC自动生成

**工具**：
- KLEE (C/C++)
- Angr (二进制)
- Triton (二进制)
- Jalangi (JavaScript)

### 模糊测试辅助
**灰盒模糊测试**：
结合代码覆盖率引导模糊测试

**工具**：
- AFL/AFL++
- LibFuzzer
- Honggfuzz
- Jazzer (Java)

### 代码属性图 (CPG)
**概念**：
结合AST、CFG、PDG的统一图表示

**查询**：
使用图查询语言发现漏洞模式

**工具**：
- Joern
- CodeQL
- Ocular

### 机器学习辅助
**应用场景**：
- 漏洞模式识别
- 误报过滤
- 相似代码检测
- 补丁分析

**技术**：
- 代码向量化
- 图神经网络
- 深度学习模型

---

代码审计需要深入理解编程语言特性和常见的安全漏洞模式。


## 实战审计策略

### 快速审计 (1-3天)
**目标**：发现高危漏洞
**方法**：
- 工具自动扫描
- 危险函数搜索
- 已知漏洞模式匹配
- 关键功能快速审查

### 深度审计 (1-2周)
**目标**：全面安全评估
**方法**：
- 完整代码走查
- 业务逻辑分析
- 数据流追踪
- 架构安全评估
- 配置审查

### 专项审计
**认证授权专项**：
- 认证机制分析
- 会话管理审查
- 权限控制检查
- RBAC/ABAC实现

**加密专项**：
- 加密算法选择
- 密钥管理
- 随机数生成
- 证书验证

**API安全专项**：
- REST/GraphQL API
- 认证授权
- 速率限制
- 输入验证

### 持续审计
**CI/CD集成**：
- 每次提交自动扫描
- PR审查自动化
- 安全门禁
- 趋势分析

**定期审计**：
- 季度深度审计
- 新功能专项审计
- 依赖更新审计
- 配置变更审计

## 审计报告

### 报告结构
1. **执行摘要**
   - 审计范围
   - 主要发现
   - 风险评估
   - 修复建议

2. **详细发现**
   - 漏洞描述
   - 影响分析
   - 复现步骤
   - 修复方案
   - 参考资料

3. **统计分析**
   - 漏洞分布
   - 严重程度统计
   - 趋势分析
   - 对比分析

4. **附录**
   - 工具配置
   - 扫描日志
   - 代码片段
   - 测试数据

### 风险评级
**CVSS评分体系**：
- 基础评分 (Base Score)
- 时间评分 (Temporal Score)
- 环境评分 (Environmental Score)

**自定义评级**：
- 技术影响
- 业务影响
- 利用难度
- 修复成本

### 修复优先级
1. **P0 - 紧急**：立即修复
2. **P1 - 高优先级**：1周内修复
3. **P2 - 中优先级**：1月内修复
4. **P3 - 低优先级**：计划修复

## 最佳实践

### 开发阶段
**安全编码规范**：
- 输入验证
- 输出编码
- 参数化查询
- 最小权限原则

**安全设计**：
- 威胁建模
- 安全架构
- 纵深防御
- 失败安全

### 审计阶段
**方法论**：
- 系统化流程
- 工具辅助
- 人工验证
- 持续改进

**质量保证**：
- 交叉验证
- 同行评审
- 漏洞复现
- 修复验证

### 修复阶段
**修复原则**：
- 根本原因修复
- 防御性编程
- 安全加固
- 回归测试

**知识沉淀**：
- 漏洞库建设
- 案例分享
- 培训教育
- 规范更新

## 进阶主题

### 供应链安全
- 依赖漏洞管理
- 开源组件审计
- 第三方库评估
- 软件物料清单 (SBOM)

### 容器安全
- Dockerfile审计
- 镜像扫描
- 运行时安全
- Kubernetes配置

### 云原生安全
- 微服务安全
- API网关
- 服务网格
- 无服务器安全

### DevSecOps
- 安全左移
- 自动化安全测试
- 安全即代码
- 持续合规

### AI/ML安全
- 模型投毒
- 对抗样本
- 数据投毒
- 模型窃取

---

**AI发挥空间**：
- 根据项目特点制定审计策略
- 识别框架和语言特定的漏洞模式
- 分析复杂的业务逻辑漏洞
- 开发自动化审计工具和脚本
- 评估漏洞的实际风险和影响
- 提供针对性的修复建议
- 持续学习新的漏洞类型和利用技术

**反幻觉提示**：
- 代码审计需要深入理解业务逻辑，不能仅依赖工具
- 不同版本的框架和库有不同的漏洞特征
- 漏洞的可利用性需要结合实际环境评估
- 某些漏洞需要特定条件才能触发
- 修复建议应该具体可行，考虑兼容性和性能
- 不要假设所有相似代码都有相同的漏洞
- 验证工具报告的准确性，避免误报和漏报
- 关注新出现的漏洞类型和攻击技术
- 代码审计是持续的过程，不是一次性活动

代码审计是一门需要技术深度、业务理解和实战经验的综合性技能。
