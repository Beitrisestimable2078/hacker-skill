# 静态分析工具与实践

静态分析工具可以自动化发现代码中的安全漏洞，提高审计效率。

## 通用工具

### SonarQube

**安装部署**：
```bash
# Docker部署
docker run -d --name sonarqube -p 9000:9000 sonarqube:latest

# 访问
http://localhost:9000
```

**项目扫描**：
```bash
# Maven
mvn sonar:sonar -Dsonar.host.url=http://localhost:9000

# Gradle
./gradlew sonarqube

# 通用扫描器
sonar-scanner \
  -Dsonar.projectKey=myproject \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://localhost:9000
```

**规则配置**：
- Security Hotspots
- Vulnerabilities
- Code Smells
- 自定义规则

### Semgrep

**安装**：
```bash
pip install semgrep
```

**使用**：
```bash
# 使用预设规则
semgrep --config=auto .

# 指定语言
semgrep --config=p/java .
semgrep --config=p/python .

# 自定义规则
semgrep --config=rules.yaml .
```

**自定义规则**：
```yaml
rules:
  - id: sql-injection
    pattern: |
      $STMT.executeQuery($SQL)
    message: Potential SQL injection
    languages: [java]
    severity: ERROR
    
  - id: command-injection
    pattern: |
      Runtime.getRuntime().exec($CMD)
    message: Command injection vulnerability
    languages: [java]
    severity: ERROR
```

### CodeQL

**安装**：
```bash
# 下载CodeQL CLI
wget https://github.com/github/codeql-cli-binaries/releases/latest/download/codeql-linux64.zip
unzip codeql-linux64.zip
```

**创建数据库**：
```bash
# Java
codeql database create java-db --language=java

# Python
codeql database create python-db --language=python

# JavaScript
codeql database create js-db --language=javascript
```

**运行查询**：
```bash
# 使用预设查询
codeql database analyze java-db java-security-and-quality.qls --format=sarif-latest --output=results.sarif

# 自定义查询
codeql query run custom-query.ql -d java-db
```

**自定义查询**：
```ql
import java

from MethodAccess ma
where ma.getMethod().getName() = "executeQuery"
  and ma.getArgument(0).toString().matches("%+%")
select ma, "Potential SQL injection"
```

## 语言特定工具

### Java

**SpotBugs + FindSecBugs**：
```xml
<!-- Maven -->
<plugin>
    <groupId>com.github.spotbugs</groupId>
    <artifactId>spotbugs-maven-plugin</artifactId>
    <version>4.7.3.0</version>
    <configuration>
        <plugins>
            <plugin>
                <groupId>com.h3xstream.findsecbugs</groupId>
                <artifactId>findsecbugs-plugin</artifactId>
                <version>1.12.0</version>
            </plugin>
        </plugins>
    </configuration>
</plugin>
```

```bash
mvn spotbugs:check
```

**Checkmarx**：
- 商业工具
- 深度数据流分析
- 支持多种语言

### PHP

**RIPS**：
```bash
# 开源版本
git clone https://github.com/ripsscanner/rips.git
php -S localhost:8080 -t rips/
```

**Psalm**：
```bash
# 安装
composer require --dev vimeo/psalm

# 初始化
./vendor/bin/psalm --init

# 运行
./vendor/bin/psalm
```

**PHPStan**：
```bash
# 安装
composer require --dev phpstan/phpstan

# 运行
./vendor/bin/phpstan analyse src
```

### Python

**Bandit**：
```bash
# 安装
pip install bandit

# 运行
bandit -r /path/to/project
bandit -r . -f json -o report.json

# 配置文件
bandit -r . -c .bandit
```

**.bandit 配置**：
```yaml
exclude_dirs:
  - /test
  - /venv

tests:
  - B201  # flask_debug_true
  - B301  # pickle
  - B302  # marshal
  - B303  # md5
  - B304  # ciphers
  - B305  # cipher_modes
  - B306  # mktemp_q
  - B307  # eval
```

### JavaScript/Node.js

**ESLint Security**：
```bash
npm install --save-dev eslint eslint-plugin-security
```

**.eslintrc.json**：
```json
{
  "plugins": ["security"],
  "extends": ["plugin:security/recommended"],
  "rules": {
    "security/detect-object-injection": "error",
    "security/detect-non-literal-regexp": "error",
    "security/detect-unsafe-regex": "error"
  }
}
```

**NodeJsScan**：
```bash
# 安装
pip install nodejsscan

# 运行
nodejsscan -d /path/to/project
```

## CI/CD 集成

### GitHub Actions

```yaml
name: Security Scan

on: [push, pull_request]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Run Semgrep
        uses: returntocorp/semgrep-action@v1
        with:
          config: auto
      
      - name: Run CodeQL
        uses: github/codeql-action/analyze@v2
        with:
          languages: java, python
```

### GitLab CI

```yaml
security_scan:
  stage: test
  image: returntocorp/semgrep
  script:
    - semgrep --config=auto . --json -o semgrep-report.json
  artifacts:
    reports:
      sast: semgrep-report.json
```

### Jenkins

```groovy
pipeline {
    agent any
    stages {
        stage('Security Scan') {
            steps {
                sh 'semgrep --config=auto .'
                sh 'bandit -r . -f json -o bandit-report.json'
            }
        }
    }
}
```

## 自定义工具开发

### 正则搜索脚本

```python
import re
import os

# 危险函数模式
patterns = {
    'eval': r'eval\s*\(',
    'exec': r'exec\s*\(',
    'system': r'system\s*\(',
    'sql_concat': r'SELECT.*\+.*\$',
}

def scan_file(filepath):
    with open(filepath, 'r') as f:
        content = f.read()
        for name, pattern in patterns.items():
            matches = re.finditer(pattern, content)
            for match in matches:
                line_num = content[:match.start()].count('\n') + 1
                print(f"{filepath}:{line_num} - {name}")

def scan_directory(directory):
    for root, dirs, files in os.walk(directory):
        for file in files:
            if file.endswith(('.php', '.py', '.java', '.js')):
                scan_file(os.path.join(root, file))

scan_directory('/path/to/project')
```

### AST 分析

```python
import ast

class SecurityVisitor(ast.NodeVisitor):
    def visit_Call(self, node):
        # 检查危险函数调用
        if isinstance(node.func, ast.Name):
            if node.func.id in ['eval', 'exec', '__import__']:
                print(f"Line {node.lineno}: Dangerous function {node.func.id}")
        self.generic_visit(node)

with open('script.py', 'r') as f:
    tree = ast.parse(f.read())
    visitor = SecurityVisitor()
    visitor.visit(tree)
```

## 最佳实践

### 规则配置

**优先级设置**：
- Critical：立即修复
- High：尽快修复
- Medium：计划修复
- Low：可选修复

**误报处理**：
- 标记误报
- 添加注释说明
- 调整规则配置

### 报告分析

**关注重点**：
1. 高危漏洞优先
2. 可利用性评估
3. 影响范围分析
4. 修复成本评估

**趋势分析**：
- 漏洞数量变化
- 新增漏洞类型
- 修复效率
- 代码质量趋势

### 持续改进

**流程优化**：
- 定期扫描
- 自动化修复
- 开发培训
- 规则更新

---

**AI 发挥空间**：
- 根据项目特点选择合适的工具
- 配置和优化扫描规则
- 分析扫描结果并评估风险
- 开发自定义分析工具

**反幻觉提示**：
- 不要完全依赖工具结果
- 验证工具报告的准确性
- 结合手工审计
- 注意工具的局限性
- 某些漏洞需要人工分析才能发现
