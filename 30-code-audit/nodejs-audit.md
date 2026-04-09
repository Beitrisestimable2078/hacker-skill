# Node.js 代码审计

Node.js 是服务端 JavaScript 运行环境，其代码审计涉及原型链污染、命令注入、NoSQL注入等。

## 常见漏洞类型

### 原型链污染

**原理**：
```javascript
// 污染Object.prototype
let obj = {};
obj.__proto__.polluted = 'yes';

// 所有对象都被污染
let newObj = {};
console.log(newObj.polluted); // 'yes'
```

**危险场景**：
```javascript
// 不安全的merge
function merge(target, source) {
    for (let key in source) {
        target[key] = source[key];
    }
    return target;
}

// 攻击payload
let payload = JSON.parse('{"__proto__":{"polluted":"yes"}}');
merge({}, payload);
```

**利用**：
```javascript
// RCE via prototype pollution
{"__proto__":{"shell":"/bin/bash","argv0":"bash"}}
```

**审计要点**：
- 搜索对象合并函数
- 检查 `__proto__`、`constructor`、`prototype`
- 验证递归合并逻辑

### 命令注入

**危险函数**：
```javascript
// child_process
const { exec } = require('child_process');
exec(userInput);

// shell: true
const { spawn } = require('child_process');
spawn('sh', ['-c', userInput]);
```

**安全方式**：
```javascript
// 使用参数数组
const { execFile } = require('child_process');
execFile('ls', ['-la']);

// spawn without shell
spawn('ping', ['-c', '1', ip]);
```

**审计要点**：
- 搜索 `exec`、`spawn`
- 检查 `shell: true` 参数
- 验证参数数组使用

### NoSQL 注入

**MongoDB**：
```javascript
// 不安全的查询
User.find({ username: req.body.username, password: req.body.password });

// 攻击payload
// {"username": {"$ne": null}, "password": {"$ne": null}}
```

**安全方式**：
```javascript
// 类型验证
if (typeof req.body.username !== 'string') {
    return res.status(400).send('Invalid input');
}

// 使用$eq
User.find({ 
    username: { $eq: req.body.username },
    password: { $eq: req.body.password }
});
```

**审计要点**：
- 搜索 MongoDB 查询
- 检查输入类型验证
- 验证操作符使用

### 路径穿越

**文件读取**：
```javascript
// 不安全的文件读取
const fs = require('fs');
app.get('/file', (req, res) => {
    const filename = req.query.file;
    fs.readFile(filename, (err, data) => {
        res.send(data);
    });
});
```

**安全方式**：
```javascript
const path = require('path');
const basePath = '/var/www/files';

app.get('/file', (req, res) => {
    const filename = path.basename(req.query.file);
    const filepath = path.join(basePath, filename);
    
    // 验证路径
    if (!filepath.startsWith(basePath)) {
        return res.status(400).send('Invalid path');
    }
    
    fs.readFile(filepath, (err, data) => {
        res.send(data);
    });
});
```

**审计要点**：
- 搜索文件操作函数
- 检查路径拼接
- 验证路径规范化

### SSRF

**HTTP 请求**：
```javascript
// 不安全的HTTP请求
const axios = require('axios');
app.get('/fetch', async (req, res) => {
    const url = req.query.url;
    const response = await axios.get(url);
    res.send(response.data);
});
```

**审计要点**：
- 搜索 HTTP 客户端
- 检查 URL 白名单
- 验证内网访问限制

### 代码注入

**eval**：
```javascript
// 不安全的eval
eval(userInput);

// Function构造函数
new Function(userInput)();

// vm模块
const vm = require('vm');
vm.runInThisContext(userInput);
```

**审计要点**：
- 搜索 `eval`、`Function`
- 检查 `vm` 模块使用
- 验证沙箱配置

### XSS

**模板引擎**：
```javascript
// EJS - 不安全的输出
<%- userInput %>  // 不转义

// Pug - 不安全的输出
div!= userInput

// Handlebars - 不安全的输出
{{{ userInput }}}
```

**审计要点**：
- 搜索不转义的输出
- 检查模板语法
- 验证输入过滤

## Express 审计

### 常见漏洞

**参数污染**：
```javascript
// 不安全的参数处理
app.get('/user', (req, res) => {
    const id = req.query.id;  // ?id=1&id=2 返回数组
    User.findById(id);
});
```

**中间件绕过**：
```javascript
// 路由顺序问题
app.use('/admin', authMiddleware);
app.get('/admin/../public/file', (req, res) => {
    // 绕过认证
});
```

**CSRF**：
```javascript
// 缺少CSRF保护
app.post('/transfer', (req, res) => {
    // 没有CSRF token验证
});
```

**审计要点**：
- 检查参数类型验证
- 验证路由顺序
- 查看 CSRF 保护

### 配置安全

**app.js**：
```javascript
// 危险配置
app.set('trust proxy', true);  // 信任X-Forwarded-*
app.disable('x-powered-by');   // 隐藏Express
```

## JWT 安全

### 常见问题

**算法混淆**：
```javascript
// 不验证算法
jwt.verify(token, publicKey);  // 可以使用HS256绕过RS256
```

**弱密钥**：
```javascript
// 弱密钥
const secret = 'secret';
jwt.sign(payload, secret);
```

**空密钥**：
```javascript
// none算法
jwt.verify(token, '', { algorithms: ['none'] });
```

**审计要点**：
- 检查算法验证
- 验证密钥强度
- 查看过期时间

## npm 包安全

### 依赖漏洞

**检查工具**：
```bash
# npm audit
npm audit
npm audit fix

# Snyk
snyk test
snyk monitor

# yarn
yarn audit
```

### 恶意包

**审计要点**：
- 检查 package.json
- 验证包来源
- 查看 postinstall 脚本
- 审查依赖树

## 审计工具

### 静态分析

**ESLint Security**：
```bash
npm install --save-dev eslint-plugin-security
```

```json
{
  "plugins": ["security"],
  "extends": ["plugin:security/recommended"]
}
```

**NodeJsScan**：
```bash
nodejsscan -d /path/to/project
```

**Semgrep**：
```bash
semgrep --config=javascript project/
```

### 代码搜索

**正则搜索**：
```bash
# 搜索危险函数
grep -r "eval\|exec\|spawn" .

# 搜索原型链污染
grep -r "__proto__\|constructor\|prototype" .
```

## 审计清单

### 危险函数
- [ ] eval、Function
- [ ] exec、spawn
- [ ] vm.runInThisContext
- [ ] require(userInput)

### 输入验证
- [ ] req.query
- [ ] req.body
- [ ] req.params
- [ ] 类型验证
- [ ] 长度限制

### 输出编码
- [ ] HTML 转义
- [ ] JSON 编码
- [ ] URL 编码
- [ ] 模板引擎配置

### 配置安全
- [ ] 环境变量
- [ ] 密钥管理
- [ ] CORS 配置
- [ ] 安全头设置

### 依赖安全
- [ ] npm audit
- [ ] 版本固定
- [ ] 定期更新
- [ ] 最小权限

---

**AI 发挥空间**：
- 根据框架特性识别漏洞
- 分析异步代码逻辑
- 追踪原型链污染路径
- 开发自动化审计脚本

**反幻觉提示**：
- 不要假设所有Node.js版本行为一致
- 验证具体的框架和库版本
- 测试漏洞的实际可利用性
- 注意异步代码的安全问题
- 某些漏洞可能需要特定的npm包版本
