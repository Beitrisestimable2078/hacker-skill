# 代码审查流程与方法

代码安全审查的系统化流程和实践方法。

## 审查流程

### 1. 前期准备
- 获取完整源代码
- 了解业务逻辑
- 确定技术栈
- 识别框架版本
- 收集文档资料

### 2. 架构分析
- 绘制系统架构图
- 识别关键组件
- 分析数据流向
- 确定信任边界
- 识别外部依赖

### 3. 入口点识别
- Web路由/控制器
- API端点
- 消息队列消费者
- 定时任务
- WebSocket处理器
- 文件上传点

### 4. 危险函数定位
- 命令执行函数
- 代码执行函数
- 文件操作函数
- 反序列化函数
- 数据库查询函数
- 网络请求函数

### 5. 数据流追踪
- 识别污点源（用户输入）
- 追踪数据传播路径
- 分析过滤和验证
- 定位污点汇聚点
- 评估安全控制

### 6. 漏洞验证
- 构造PoC
- 本地测试
- 评估影响
- 确认可利用性
- 记录复现步骤

### 7. 报告编写
- 漏洞描述
- 影响分析
- 复现步骤
- 修复建议
- 风险评级

## 审查方法

### 自顶向下
- 从业务逻辑入手
- 分析关键功能
- 识别高风险点
- 深入代码细节
- 适合业务逻辑漏洞

### 自底向上
- 从危险函数入手
- 反向追踪调用链
- 寻找可控输入
- 构造利用路径
- 适合技术类漏洞

### 混合方法
- 结合两种方法
- 先识别关键功能
- 再定位危险函数
- 双向追踪验证
- 最全面的方法

## 检查清单

### 注入漏洞
- [ ] SQL注入
- [ ] 命令注入
- [ ] 代码注入
- [ ] LDAP注入
- [ ] XPath注入
- [ ] NoSQL注入
- [ ] ORM注入

### XSS漏洞
- [ ] 反射型XSS
- [ ] 存储型XSS
- [ ] DOM型XSS
- [ ] 模板注入
- [ ] 输出编码检查

### 文件操作
- [ ] 文件上传
- [ ] 文件包含
- [ ] 文件下载
- [ ] 路径穿越
- [ ] 文件删除
- [ ] ZIP解压

### 反序列化
- [ ] Java反序列化
- [ ] PHP反序列化
- [ ] Python Pickle
- [ ] .NET反序列化
- [ ] JSON反序列化

### 认证授权
- [ ] 认证绕过
- [ ] 弱密码策略
- [ ] 会话管理
- [ ] 权限检查
- [ ] IDOR
- [ ] 越权访问

### 业务逻辑
- [ ] 支付逻辑
- [ ] 优惠券逻辑
- [ ] 积分逻辑
- [ ] 条件竞争
- [ ] 重放攻击
- [ ] 批量操作

### 配置安全
- [ ] 敏感信息泄露
- [ ] 调试模式
- [ ] 默认凭证
- [ ] 不安全配置
- [ ] CORS配置
- [ ] CSP配置

### 加密安全
- [ ] 弱加密算法
- [ ] 硬编码密钥
- [ ] 不安全随机数
- [ ] 密码存储
- [ ] 传输加密

## 优先级评估

### 严重（Critical）
- 远程代码执行
- SQL注入（敏感数据）
- 认证绕过
- 任意文件读取（敏感文件）
- 反序列化RCE

### 高危（High）
- 存储型XSS
- 任意文件上传
- SSRF（内网访问）
- 越权访问（敏感操作）
- 命令注入

### 中危（Medium）
- 反射型XSS
- 路径穿越
- 信息泄露
- CSRF
- 弱密码策略

### 低危（Low）
- 自XSS
- 信息泄露（非敏感）
- 配置问题
- 缺少安全头

## 审查技巧

### 关键字搜索
```bash
# 命令执行
grep -r "exec\|system\|shell_exec\|passthru" .
grep -r "Runtime.getRuntime\|ProcessBuilder" .
grep -r "subprocess\|os.system\|eval" .

# SQL查询
grep -r "query\|execute\|prepare" .
grep -r "SELECT.*FROM\|INSERT.*INTO" .

# 文件操作
grep -r "file_get_contents\|fopen\|readfile" .
grep -r "FileInputStream\|FileReader" .
grep -r "open\|read\|write" .

# 反序列化
grep -r "unserialize\|deserialize" .
grep -r "pickle.loads\|yaml.load" .
grep -r "ObjectInputStream\|readObject" .
```

### 正则表达式
```python
# SQL注入模式
r"SELECT.*\+.*FROM"
r"query\([^)]*\$"
r"execute\([^)]*\+"

# XSS模式
r"innerHTML.*=.*\+"
r"document\.write\("
r"eval\([^)]*\+"

# 命令注入模式
r"exec\([^)]*\$"
r"system\([^)]*\+"
r"shell_exec\([^)]*\."
```

### AST分析
- 使用语言特定的AST工具
- 分析函数调用链
- 识别数据流
- 检测污点传播
- 自动化漏洞发现

### 差异对比
- 对比不同版本
- 识别安全修复
- 发现新增功能
- 检查回归问题

## 自动化工具集成

### 静态分析
```bash
# SonarQube
sonar-scanner \
  -Dsonar.projectKey=myproject \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://localhost:9000

# Semgrep
semgrep --config=auto .
semgrep --config=p/security-audit .

# CodeQL
codeql database create mydb --language=java
codeql database analyze mydb --format=sarif-latest --output=results.sarif
```

### IDE集成
- 使用安全插件
- 实时代码检查
- 快速定位问题
- 提供修复建议

### CI/CD集成
```yaml
# GitLab CI示例
security_scan:
  stage: test
  script:
    - semgrep --config=auto --json -o semgrep.json .
    - sonar-scanner
  artifacts:
    reports:
      sast: semgrep.json
```

## 报告模板

### 漏洞标题
简洁明确的漏洞描述

### 漏洞描述
- 漏洞类型
- 影响范围
- 触发条件
- 技术细节

### 复现步骤
1. 详细的操作步骤
2. 请求和响应示例
3. PoC代码
4. 截图证明

### 影响分析
- 机密性影响
- 完整性影响
- 可用性影响
- 业务影响

### 修复建议
- 短期修复方案
- 长期修复方案
- 代码示例
- 最佳实践

### 风险评级
- CVSS评分
- 风险等级
- 利用难度
- 修复优先级

## 修复验证

### 代码审查
- 检查修复代码
- 验证修复逻辑
- 确认无副作用
- 检查相似问题

### 功能测试
- 正常功能验证
- 边界条件测试
- 异常情况测试
- 性能影响评估

### 安全测试
- 原PoC验证
- 绕过尝试
- 回归测试
- 相关漏洞检查

### 文档更新
- 更新安全文档
- 记录修复方案
- 总结经验教训
- 分享最佳实践

## 最佳实践

### 审查准备
- 充分了解业务
- 熟悉技术栈
- 准备工具环境
- 制定审查计划

### 审查执行
- 系统化方法
- 详细记录
- 及时沟通
- 持续学习

### 质量保证
- 交叉验证
- 同行评审
- 工具辅助
- 经验积累

### 持续改进
- 总结经验
- 更新检查清单
- 优化流程
- 分享知识

---

**反幻觉提示**：
- 审查流程应根据项目实际情况调整
- 不同语言和框架有不同的审查重点
- 自动化工具只是辅助，人工审查不可替代
- 漏洞评级应结合实际业务场景
- 修复建议应具体可行，避免泛泛而谈

代码审查是一个系统化的过程，需要方法论、工具和经验的结合。
