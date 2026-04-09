# Python 代码审计

Python 在 Web 开发和数据处理中广泛使用，其代码审计涉及框架漏洞、模板注入、反序列化等。

## 常见漏洞类型

### 代码执行

**危险函数**：
```python
# eval
eval(user_input)

# exec
exec(user_input)

# compile + exec
code = compile(user_input, '<string>', 'exec')
exec(code)

# __import__
__import__(user_input)
```

**审计要点**：
- 搜索 `eval`、`exec`
- 检查动态导入
- 验证输入过滤

### 命令注入

**危险函数**：
```python
# os.system
os.system(user_input)

# os.popen
os.popen(user_input)

# subprocess
subprocess.call(user_input, shell=True)
subprocess.Popen(user_input, shell=True)

# commands (Python 2)
commands.getoutput(user_input)
```

**安全方式**：
```python
# 使用列表参数
subprocess.call(['ls', '-la'])
subprocess.Popen(['ping', '-c', '1', ip])
```

**审计要点**：
- 搜索命令执行函数
- 检查 shell=True 参数
- 验证参数列表使用

### SQL 注入

**字符串拼接**：
```python
# 不安全的SQL
query = "SELECT * FROM users WHERE id = " + user_id
cursor.execute(query)

query = f"SELECT * FROM users WHERE name = '{username}'"
cursor.execute(query)
```

**参数化查询**：
```python
# 安全的方式
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
cursor.execute("SELECT * FROM users WHERE name = ?", (username,))
```

**ORM 注入**：
```python
# Django ORM - 不安全的extra
User.objects.extra(where=["id = " + user_id])

# SQLAlchemy - 不安全的text
session.query(User).filter(text("id = " + user_id))
```

**审计要点**：
- 搜索字符串拼接的 SQL
- 检查 f-string 在 SQL 中使用
- 验证参数化查询

### SSTI (模板注入)

**Jinja2**：
```python
# 不安全的模板渲染
from jinja2 import Template
template = Template(user_input)
template.render()
```

**利用 Payload**：
```python
# RCE payload
{{''.__class__.__mro__[1].__subclasses__()[104].__init__.__globals__['sys'].modules['os'].popen('whoami').read()}}

# 简化版
{{config.__class__.__init__.__globals__['os'].popen('whoami').read()}}
```

**审计要点**：
- 搜索 `Template(` 构造
- 检查用户输入是否进入模板
- 验证模板沙箱配置

### 反序列化

**Pickle**：
```python
# 不安全的反序列化
import pickle
data = pickle.loads(user_input)
```

**利用**：
```python
# 恶意pickle对象
import pickle
import os

class Evil:
    def __reduce__(self):
        return (os.system, ('whoami',))

payload = pickle.dumps(Evil())
```

**YAML**：
```python
# 不安全的YAML加载
import yaml
data = yaml.load(user_input)  # 使用Loader=yaml.FullLoader
```

**审计要点**：
- 搜索 `pickle.loads`
- 检查 `yaml.load` 使用
- 验证 `yaml.safe_load` 使用

### 文件操作

**路径穿越**：
```python
# 不安全的文件读取
filename = request.args.get('file')
with open(filename, 'r') as f:
    content = f.read()
```

**文件上传**：
```python
# 不安全的文件保存
file = request.files['file']
file.save(os.path.join('uploads', file.filename))
```

**审计要点**：
- 搜索 `open(` 函数
- 检查路径拼接
- 验证路径规范化

### XXE

**XML 解析**：
```python
# 不安全的XML解析
import xml.etree.ElementTree as ET
tree = ET.parse(user_input)

from xml.dom import minidom
dom = minidom.parse(user_input)
```

**安全配置**：
```python
# 使用defusedxml
from defusedxml import ElementTree as ET
tree = ET.parse(user_input)
```

**审计要点**：
- 搜索 XML 解析库
- 检查外部实体配置
- 验证 defusedxml 使用

## Django 审计

### 常见漏洞

**SQL 注入**：
```python
# 不安全的extra
User.objects.extra(where=["id = %s" % user_id])

# 不安全的raw
User.objects.raw("SELECT * FROM users WHERE id = " + user_id)
```

**XSS**：
```python
# 模板中使用safe
{{ user_input|safe }}

# mark_safe
from django.utils.safestring import mark_safe
html = mark_safe(user_input)
```

**CSRF**：
```python
# 禁用CSRF保护
from django.views.decorators.csrf import csrf_exempt

@csrf_exempt
def view(request):
    pass
```

**审计要点**：
- 检查 `extra`、`raw` 使用
- 搜索 `safe`、`mark_safe`
- 验证 CSRF 保护

### 配置安全

**settings.py**：
```python
# 危险配置
DEBUG = True
SECRET_KEY = 'hardcoded-key'
ALLOWED_HOSTS = ['*']
```

**审计要点**：
- 检查 DEBUG 模式
- 验证 SECRET_KEY
- 查看 ALLOWED_HOSTS

## Flask 审计

### 常见漏洞

**SSTI**：
```python
# 不安全的模板渲染
@app.route('/hello')
def hello():
    name = request.args.get('name')
    return render_template_string('Hello ' + name)
```

**SQL 注入**：
```python
# 不安全的SQL
@app.route('/user')
def user():
    user_id = request.args.get('id')
    query = f"SELECT * FROM users WHERE id = {user_id}"
    cursor.execute(query)
```

**文件读取**：
```python
# 不安全的send_file
@app.route('/download')
def download():
    filename = request.args.get('file')
    return send_file(filename)
```

**审计要点**：
- 搜索 `render_template_string`
- 检查 SQL 字符串拼接
- 验证文件路径过滤

### 配置安全

**app.py**：
```python
# 危险配置
app.debug = True
app.secret_key = 'hardcoded-key'
```

## 审计工具

### 静态分析

**Bandit**：
```bash
# Python安全检查
bandit -r /path/to/project
bandit -r . -f json -o report.json
```

**Pylint**：
```bash
pylint --load-plugins=pylint_django project/
```

**Semgrep**：
```bash
semgrep --config=python project/
```

### 依赖检查

**Safety**：
```bash
# 检查依赖漏洞
safety check
safety check --json
```

**pip-audit**：
```bash
pip-audit
```

## 审计清单

### 危险函数
- [ ] eval、exec、compile
- [ ] os.system、subprocess
- [ ] pickle.loads
- [ ] yaml.load
- [ ] Template()

### 输入验证
- [ ] request.args
- [ ] request.form
- [ ] request.json
- [ ] 过滤和验证

### 输出编码
- [ ] HTML 转义
- [ ] JSON 编码
- [ ] SQL 参数化
- [ ] 命令参数列表

### 配置安全
- [ ] DEBUG = False
- [ ] SECRET_KEY 随机
- [ ] ALLOWED_HOSTS 限制
- [ ] 依赖版本更新

---

**AI 发挥空间**：
- 根据框架特性识别漏洞
- 分析代码逻辑发现注入点
- 追踪数据流发现安全问题
- 开发自动化审计脚本

**反幻觉提示**：
- 不要假设所有Python版本行为一致
- 验证具体的框架版本
- 测试漏洞的实际可利用性
- 注意Python 2和Python 3的差异
- 某些漏洞可能需要特定的库版本
