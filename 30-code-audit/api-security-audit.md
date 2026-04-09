# API安全审计

API (Application Programming Interface) 是现代应用的核心，API安全审计涉及认证授权、输入验证、速率限制等多个方面。

## API类型

### REST API
**特点**：
- 基于HTTP协议
- 无状态
- 资源导向
- 标准HTTP方法

**安全关注点**：
- 认证授权机制
- HTTPS使用
- CORS配置
- 输入验证
- 速率限制

### GraphQL API
**特点**：
- 灵活的查询语言
- 单一端点
- 类型系统
- 实时订阅

**安全关注点**：
- 查询深度限制
- 查询复杂度限制
- 批量查询限制
- 字段级权限控制
- 内省查询限制

### gRPC API
**特点**：
- 基于HTTP/2
- Protocol Buffers
- 双向流
- 高性能

**安全关注点**：
- TLS配置
- 认证机制
- 消息验证
- 流量控制

### WebSocket API
**特点**：
- 全双工通信
- 持久连接
- 实时数据

**安全关注点**：
- 连接认证
- 消息验证
- 速率限制
- 连接管理

## 认证与授权

### API密钥认证

**实现**：
```python
# 不安全：密钥在URL中
@app.route('/api/data')
def get_data():
    api_key = request.args.get('api_key')
    if api_key == VALID_API_KEY:
        return jsonify(data)

# 安全：密钥在Header中
@app.route('/api/data')
def get_data():
    api_key = request.headers.get('X-API-Key')
    if not validate_api_key(api_key):
        return jsonify({'error': 'Invalid API key'}), 401
    return jsonify(data)
```

**审计要点**：
- 密钥是否在Header中传输
- 是否使用HTTPS
- 密钥是否有过期时间
- 是否有密钥轮换机制
- 是否记录密钥使用

### OAuth 2.0

**常见漏洞**：

**授权码劫持**：
```javascript
// 不安全：未验证redirect_uri
app.get('/oauth/authorize', (req, res) => {
    const { client_id, redirect_uri } = req.query;
    const code = generateAuthCode();
    res.redirect(`${redirect_uri}?code=${code}`);
});

// 安全：验证redirect_uri
app.get('/oauth/authorize', (req, res) => {
    const { client_id, redirect_uri } = req.query;
    const client = getClient(client_id);
    
    if (!client.redirect_uris.includes(redirect_uri)) {
        return res.status(400).send('Invalid redirect_uri');
    }
    
    const code = generateAuthCode();
    res.redirect(`${redirect_uri}?code=${code}`);
});
```

**CSRF攻击**：
```javascript
// 使用state参数防止CSRF
app.get('/oauth/authorize', (req, res) => {
    const state = req.query.state;
    // 验证state参数
    if (!validateState(state)) {
        return res.status(400).send('Invalid state');
    }
    // ...
});
```

**审计要点**：
- redirect_uri是否严格验证
- 是否使用state参数
- 授权码是否一次性
- 是否使用PKCE (移动应用)
- token是否有过期时间
- refresh_token是否安全存储

### JWT (JSON Web Token)

**常见漏洞**：

**算法混淆**：
```javascript
// 不安全：未指定算法
jwt.verify(token, publicKey);

// 安全：指定算法
jwt.verify(token, publicKey, { algorithms: ['RS256'] });
```

**弱密钥**：
```python
# 不安全：弱密钥
SECRET_KEY = 'secret'
token = jwt.encode(payload, SECRET_KEY, algorithm='HS256')

# 安全：强密钥
import secrets
SECRET_KEY = secrets.token_urlsafe(32)
token = jwt.encode(payload, SECRET_KEY, algorithm='HS256')
```

**敏感信息泄露**：
```javascript
// 不安全：JWT中包含敏感信息
const payload = {
    userId: user.id,
    password: user.password,  // 不应包含密码
    ssn: user.ssn  // 不应包含敏感信息
};

// 安全：只包含必要信息
const payload = {
    userId: user.id,
    role: user.role,
    exp: Math.floor(Date.now() / 1000) + (60 * 60)  // 1小时过期
};
```

**审计要点**：
- 是否指定算法
- 密钥强度是否足够
- 是否包含敏感信息
- 是否设置过期时间
- 是否验证签名
- 是否有token撤销机制

### API网关认证

**多层认证**：
```yaml
# Kong配置示例
plugins:
  - name: key-auth
    config:
      key_names: [apikey]
  - name: rate-limiting
    config:
      minute: 100
  - name: jwt
    config:
      secret_is_base64: false
```

**审计要点**：
- 网关认证配置
- 后端服务是否再次验证
- 认证信息传递方式
- 认证失败处理

## 输入验证

### 参数验证

**类型验证**：
```python
# 不安全：未验证类型
@app.route('/api/user/<user_id>')
def get_user(user_id):
    user = User.query.get(user_id)  # user_id可能不是整数
    return jsonify(user)

# 安全：验证类型
from flask import request
from marshmallow import Schema, fields, ValidationError

class UserSchema(Schema):
    user_id = fields.Int(required=True, validate=lambda x: x > 0)

@app.route('/api/user/<int:user_id>')
def get_user(user_id):
    try:
        schema = UserSchema()
        data = schema.load({'user_id': user_id})
        user = User.query.get(data['user_id'])
        return jsonify(user)
    except ValidationError as err:
        return jsonify(err.messages), 400
```

**范围验证**：
```java
// 不安全：未验证范围
@GetMapping("/api/users")
public List<User> getUsers(@RequestParam int page, @RequestParam int size) {
    return userService.findAll(page, size);
}

// 安全：验证范围
@GetMapping("/api/users")
public List<User> getUsers(
    @RequestParam @Min(0) int page,
    @RequestParam @Min(1) @Max(100) int size
) {
    return userService.findAll(page, size);
}
```

**格式验证**：
```javascript
// 使用JSON Schema验证
const Ajv = require('ajv');
const ajv = new Ajv();

const schema = {
    type: 'object',
    properties: {
        email: { type: 'string', format: 'email' },
        age: { type: 'integer', minimum: 0, maximum: 150 },
        phone: { type: 'string', pattern: '^\\d{11}$' }
    },
    required: ['email', 'age'],
    additionalProperties: false
};

app.post('/api/user', (req, res) => {
    const validate = ajv.compile(schema);
    const valid = validate(req.body);
    
    if (!valid) {
        return res.status(400).json(validate.errors);
    }
    
    // 处理请求
});
```

### 批量操作限制

**数组大小限制**：
```python
# 不安全：无限制批量操作
@app.route('/api/users/batch', methods=['POST'])
def batch_create_users():
    users = request.json.get('users', [])
    for user_data in users:
        create_user(user_data)
    return jsonify({'created': len(users)})

# 安全：限制数量
@app.route('/api/users/batch', methods=['POST'])
def batch_create_users():
    users = request.json.get('users', [])
    
    if len(users) > 100:
        return jsonify({'error': 'Too many users'}), 400
    
    if len(users) == 0:
        return jsonify({'error': 'No users provided'}), 400
    
    created = []
    for user_data in users:
        user = create_user(user_data)
        created.append(user)
    
    return jsonify({'created': len(created)})
```

### GraphQL特定验证

**查询深度限制**：
```javascript
const depthLimit = require('graphql-depth-limit');

const server = new ApolloServer({
    typeDefs,
    resolvers,
    validationRules: [depthLimit(5)]  // 最大深度5
});
```

**查询复杂度限制**：
```javascript
const { createComplexityLimitRule } = require('graphql-validation-complexity');

const server = new ApolloServer({
    typeDefs,
    resolvers,
    validationRules: [
        createComplexityLimitRule(1000)  // 最大复杂度1000
    ]
});
```

**批量查询限制**：
```javascript
// 限制批量查询数量
const batchingMiddleware = (req, res, next) => {
    if (Array.isArray(req.body)) {
        if (req.body.length > 10) {
            return res.status(400).json({
                error: 'Too many queries in batch'
            });
        }
    }
    next();
};

app.use('/graphql', batchingMiddleware, graphqlHTTP({...}));
```

## 速率限制

### 基于IP的限制

**实现**：
```python
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

limiter = Limiter(
    app,
    key_func=get_remote_address,
    default_limits=["200 per day", "50 per hour"]
)

@app.route('/api/data')
@limiter.limit("10 per minute")
def get_data():
    return jsonify(data)
```

### 基于用户的限制

**实现**：
```javascript
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');

const limiter = rateLimit({
    store: new RedisStore({
        client: redisClient
    }),
    windowMs: 15 * 60 * 1000,  // 15分钟
    max: 100,  // 最多100个请求
    keyGenerator: (req) => req.user.id,  // 基于用户ID
    handler: (req, res) => {
        res.status(429).json({
            error: 'Too many requests'
        });
    }
});

app.use('/api/', limiter);
```

### 动态速率限制

**根据用户等级**：
```python
def get_rate_limit(user):
    if user.is_premium:
        return "1000 per hour"
    elif user.is_verified:
        return "500 per hour"
    else:
        return "100 per hour"

@app.route('/api/data')
@limiter.limit(lambda: get_rate_limit(current_user))
def get_data():
    return jsonify(data)
```

### 成本限制

**GraphQL成本计算**：
```javascript
const costAnalysis = require('graphql-cost-analysis').default;

const server = new ApolloServer({
    typeDefs,
    resolvers,
    validationRules: [
        costAnalysis({
            maximumCost: 1000,
            defaultCost: 1,
            onComplete: (cost) => {
                console.log('Query cost:', cost);
            }
        })
    ]
});
```

## 输出安全

### 敏感信息过滤

**响应过滤**：
```python
# 不安全：返回所有字段
@app.route('/api/user/<int:user_id>')
def get_user(user_id):
    user = User.query.get(user_id)
    return jsonify(user.__dict__)  # 可能包含密码等敏感信息

# 安全：只返回必要字段
@app.route('/api/user/<int:user_id>')
def get_user(user_id):
    user = User.query.get(user_id)
    return jsonify({
        'id': user.id,
        'username': user.username,
        'email': user.email,
        # 不返回password, ssn等敏感字段
    })
```

**使用序列化器**：
```python
from marshmallow import Schema, fields

class UserSchema(Schema):
    id = fields.Int()
    username = fields.Str()
    email = fields.Email()
    # 不包含password字段

    class Meta:
        fields = ('id', 'username', 'email')

@app.route('/api/user/<int:user_id>')
def get_user(user_id):
    user = User.query.get(user_id)
    schema = UserSchema()
    return jsonify(schema.dump(user))
```

### 错误信息处理

**不安全的错误信息**：
```javascript
// 不安全：暴露内部信息
app.post('/api/login', (req, res) => {
    try {
        const user = findUser(req.body.username);
        if (!user) {
            return res.status(404).json({ error: 'User not found' });
        }
        if (!checkPassword(user, req.body.password)) {
            return res.status(401).json({ error: 'Invalid password' });
        }
        // ...
    } catch (err) {
        res.status(500).json({ error: err.message, stack: err.stack });
    }
});

// 安全：统一错误信息
app.post('/api/login', (req, res) => {
    try {
        const user = findUser(req.body.username);
        if (!user || !checkPassword(user, req.body.password)) {
            return res.status(401).json({ 
                error: 'Invalid credentials' 
            });
        }
        // ...
    } catch (err) {
        logger.error(err);  // 记录详细错误
        res.status(500).json({ 
            error: 'Internal server error',
            requestId: req.id  // 只返回请求ID
        });
    }
});
```

### CORS配置

**不安全的CORS**：
```javascript
// 不安全：允许所有来源
app.use(cors({
    origin: '*',
    credentials: true
}));

// 安全：白名单
const whitelist = ['https://example.com', 'https://app.example.com'];
app.use(cors({
    origin: (origin, callback) => {
        if (whitelist.indexOf(origin) !== -1 || !origin) {
            callback(null, true);
        } else {
            callback(new Error('Not allowed by CORS'));
        }
    },
    credentials: true,
    methods: ['GET', 'POST', 'PUT', 'DELETE'],
    allowedHeaders: ['Content-Type', 'Authorization']
}));
```

## API版本管理

### 版本策略

**URL版本**：
```
GET /api/v1/users
GET /api/v2/users
```

**Header版本**：
```
GET /api/users
Accept: application/vnd.api+json; version=1
```

**查询参数版本**：
```
GET /api/users?version=1
```

### 废弃处理

**响应头提示**：
```javascript
app.get('/api/v1/users', (req, res) => {
    res.set('X-API-Deprecated', 'true');
    res.set('X-API-Sunset', '2026-12-31');
    res.set('Link', '</api/v2/users>; rel="successor-version"');
    // ...
});
```

## API文档安全

### Swagger/OpenAPI

**敏感信息**：
```yaml
# 不安全：暴露内部端点
paths:
  /api/admin/debug:
    get:
      summary: Debug endpoint
      # 不应在生产环境暴露

# 安全：根据环境配置
paths:
  /api/users:
    get:
      summary: Get users
      security:
        - bearerAuth: []
```

**认证配置**：
```yaml
components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

security:
  - bearerAuth: []
```

### GraphQL内省

**禁用内省**：
```javascript
const server = new ApolloServer({
    typeDefs,
    resolvers,
    introspection: process.env.NODE_ENV !== 'production'
});
```

## 审计清单

### 认证授权
- [ ] 是否使用HTTPS
- [ ] 认证机制是否安全
- [ ] token是否有过期时间
- [ ] 是否有token撤销机制
- [ ] 权限控制是否细粒度
- [ ] 是否防止越权访问

### 输入验证
- [ ] 是否验证参数类型
- [ ] 是否验证参数范围
- [ ] 是否验证参数格式
- [ ] 是否限制批量操作
- [ ] GraphQL是否限制查询深度
- [ ] GraphQL是否限制查询复杂度

### 速率限制
- [ ] 是否有速率限制
- [ ] 限制策略是否合理
- [ ] 是否基于用户限制
- [ ] 是否有成本限制
- [ ] 超限响应是否正确

### 输出安全
- [ ] 是否过滤敏感信息
- [ ] 错误信息是否安全
- [ ] CORS配置是否正确
- [ ] 响应头是否安全
- [ ] 是否有数据脱敏

### 日志审计
- [ ] 是否记录API调用
- [ ] 是否记录认证失败
- [ ] 是否记录异常行为
- [ ] 日志是否包含敏感信息
- [ ] 是否有日志分析

### 文档安全
- [ ] 文档是否需要认证
- [ ] 是否暴露敏感端点
- [ ] 内省是否在生产环境禁用
- [ ] 示例是否包含真实数据

---

**AI发挥空间**：
- 分析API设计的安全性
- 识别认证授权漏洞
- 评估速率限制策略
- 发现输入验证问题
- 检查输出安全配置
- 提供API安全最佳实践建议
- 根据API类型提供针对性审计

**反幻觉提示**：
- 不同类型的API有不同的安全关注点
- 认证机制的选择取决于具体场景
- 速率限制策略需要平衡安全和可用性
- CORS配置需要根据实际需求调整
- 不要假设所有API都需要相同的安全措施
- 某些看似不安全的配置可能是业务需求
- API安全需要与业务需求平衡
- 测试时注意不要影响生产环境

API安全审计需要理解API设计模式和安全最佳实践。
