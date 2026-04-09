# 业务逻辑漏洞审计

业务逻辑漏洞是由于业务流程设计缺陷或实现错误导致的安全问题，往往比技术漏洞更难发现但危害更大。

## 核心概念

### 业务逻辑漏洞特点
- **难以自动化发现**：需要理解业务流程
- **危害往往更大**：直接影响业务和资金
- **修复成本高**：可能需要重构业务流程
- **容易被忽视**：传统扫描器无法发现

### 审计思路
1. 深入理解业务流程
2. 识别关键业务节点
3. 分析状态转换逻辑
4. 检查边界条件处理
5. 验证权限控制
6. 测试异常场景

## 常见漏洞类型

### 支付逻辑漏洞

**价格篡改**：
```javascript
// 不安全：前端传递价格
app.post('/order', (req, res) => {
    const { productId, price, quantity } = req.body;
    const total = price * quantity;  // 价格可被篡改
    processPayment(total);
});

// 安全：后端查询价格
app.post('/order', (req, res) => {
    const { productId, quantity } = req.body;
    const product = db.getProduct(productId);
    const total = product.price * quantity;  // 价格由后端控制
    processPayment(total);
});
```

**金额溢出**：
```python
# 整数溢出
amount = int(request.form['amount'])
if amount > 0:
    # amount = -1 会绕过检查
    user.balance -= amount
```

**支付回调伪造**：
```php
// 不安全：未验证签名
if ($_POST['status'] == 'success') {
    updateOrderStatus($_POST['order_id'], 'paid');
}

// 安全：验证签名
$sign = md5($order_id . $amount . $secret);
if ($_POST['sign'] == $sign && $_POST['status'] == 'success') {
    updateOrderStatus($_POST['order_id'], 'paid');
}
```

**重复支付**：
```java
// 不安全：未检查订单状态
public void processPayment(String orderId) {
    // 直接扣款，可能重复扣款
    deductBalance(orderId);
}

// 安全：检查订单状态
public synchronized void processPayment(String orderId) {
    Order order = getOrder(orderId);
    if (order.getStatus() == OrderStatus.PENDING) {
        deductBalance(orderId);
        order.setStatus(OrderStatus.PAID);
    }
}
```

**审计要点**：
- 价格是否由后端控制
- 是否存在整数溢出
- 支付回调是否验证
- 是否防止重复支付
- 退款逻辑是否安全
- 优惠叠加是否合理

### 优惠券/折扣漏洞

**重复使用**：
```python
# 不安全：未标记已使用
def apply_coupon(user_id, coupon_code):
    coupon = Coupon.query.filter_by(code=coupon_code).first()
    if coupon:
        return coupon.discount
    return 0

# 安全：标记使用状态
def apply_coupon(user_id, coupon_code):
    coupon = Coupon.query.filter_by(
        code=coupon_code,
        used=False
    ).first()
    if coupon:
        coupon.used = True
        coupon.used_by = user_id
        db.session.commit()
        return coupon.discount
    return 0
```

**条件绕过**：
```javascript
// 不安全：前端验证
if (orderAmount >= 100) {
    applyCoupon(couponCode);
}

// 安全：后端验证
app.post('/apply-coupon', (req, res) => {
    const order = getOrder(req.body.orderId);
    const coupon = getCoupon(req.body.couponCode);
    
    if (order.amount >= coupon.minAmount) {
        applyCoupon(order, coupon);
    }
});
```

**叠加使用**：
```java
// 检查优惠叠加规则
public boolean canStackCoupons(List<Coupon> coupons) {
    for (Coupon c1 : coupons) {
        for (Coupon c2 : coupons) {
            if (c1 != c2 && !c1.canStackWith(c2)) {
                return false;
            }
        }
    }
    return true;
}
```

**审计要点**：
- 是否防止重复使用
- 使用条件是否后端验证
- 叠加规则是否正确
- 过期时间是否检查
- 使用次数是否限制
- 用户资格是否验证

### 积分/余额漏洞

**负数绕过**：
```python
# 不安全：未检查负数
def transfer(from_user, to_user, amount):
    from_user.balance -= amount  # amount=-100 会增加余额
    to_user.balance += amount

# 安全：检查负数和余额
def transfer(from_user, to_user, amount):
    if amount <= 0:
        raise ValueError("Amount must be positive")
    if from_user.balance < amount:
        raise ValueError("Insufficient balance")
    
    from_user.balance -= amount
    to_user.balance += amount
```

**并发问题**：
```java
// 不安全：存在竞态条件
public void withdraw(User user, int amount) {
    if (user.getBalance() >= amount) {
        // 并发时可能多次通过检查
        user.setBalance(user.getBalance() - amount);
    }
}

// 安全：使用事务和锁
@Transactional
public synchronized void withdraw(User user, int amount) {
    User lockedUser = userRepository.findByIdForUpdate(user.getId());
    if (lockedUser.getBalance() >= amount) {
        lockedUser.setBalance(lockedUser.getBalance() - amount);
        userRepository.save(lockedUser);
    }
}
```

**溢出问题**：
```javascript
// 检查整数溢出
function addPoints(user, points) {
    const MAX_POINTS = Number.MAX_SAFE_INTEGER;
    if (user.points + points > MAX_POINTS) {
        throw new Error("Points overflow");
    }
    user.points += points;
}
```

**审计要点**：
- 是否检查负数
- 是否防止并发问题
- 是否检查溢出
- 转账逻辑是否原子
- 是否有操作日志
- 异常回滚是否正确

### 条件竞争

**订单状态竞争**：
```python
# 不安全：检查和更新不原子
def cancel_order(order_id):
    order = Order.query.get(order_id)
    if order.status == 'pending':
        # 并发时可能取消已支付订单
        order.status = 'cancelled'
        db.session.commit()

# 安全：使用乐观锁
def cancel_order(order_id):
    order = Order.query.filter_by(
        id=order_id,
        status='pending',
        version=order.version
    ).first()
    if order:
        order.status = 'cancelled'
        order.version += 1
        db.session.commit()
```

**库存竞争**：
```java
// 不安全：检查和扣减不原子
public boolean purchaseProduct(int productId, int quantity) {
    Product product = getProduct(productId);
    if (product.getStock() >= quantity) {
        product.setStock(product.getStock() - quantity);
        return true;
    }
    return false;
}

// 安全：使用数据库原子操作
public boolean purchaseProduct(int productId, int quantity) {
    int updated = jdbcTemplate.update(
        "UPDATE products SET stock = stock - ? " +
        "WHERE id = ? AND stock >= ?",
        quantity, productId, quantity
    );
    return updated > 0;
}
```

**审计要点**：
- 识别TOCTOU (Time-of-Check-Time-of-Use)
- 检查是否使用锁机制
- 验证事务隔离级别
- 测试并发场景
- 检查幂等性设计

### 重放攻击

**请求重放**：
```javascript
// 不安全：可重放请求
app.post('/transfer', (req, res) => {
    const { from, to, amount } = req.body;
    transfer(from, to, amount);  // 可重复提交
});

// 安全：使用nonce或timestamp
app.post('/transfer', (req, res) => {
    const { from, to, amount, nonce, timestamp } = req.body;
    
    // 检查nonce是否已使用
    if (isNonceUsed(nonce)) {
        return res.status(400).send('Request already processed');
    }
    
    // 检查时间戳
    if (Date.now() - timestamp > 60000) {
        return res.status(400).send('Request expired');
    }
    
    markNonceUsed(nonce);
    transfer(from, to, amount);
});
```

**验证码重放**：
```python
# 不安全：验证码可重复使用
def verify_code(phone, code):
    stored_code = redis.get(f'code:{phone}')
    return stored_code == code

# 安全：验证后删除
def verify_code(phone, code):
    stored_code = redis.get(f'code:{phone}')
    if stored_code == code:
        redis.delete(f'code:{phone}')  # 验证后删除
        return True
    return False
```

**审计要点**：
- 是否使用nonce/token
- 是否检查时间戳
- 验证码是否一次性
- 是否有请求签名
- 是否记录请求历史

### 批量操作漏洞

**批量注册**：
```python
# 不安全：无限制批量注册
@app.route('/register', methods=['POST'])
def register():
    username = request.form['username']
    create_user(username)

# 安全：限制频率和验证
@app.route('/register', methods=['POST'])
@rate_limit(max_requests=5, window=3600)  # 每小时5次
def register():
    username = request.form['username']
    captcha = request.form['captcha']
    
    if not verify_captcha(captcha):
        return 'Invalid captcha', 400
    
    create_user(username)
```

**批量查询**：
```java
// 不安全：无限制批量查询
@GetMapping("/users")
public List<User> getUsers(@RequestParam List<Long> ids) {
    return userService.findByIds(ids);  // 可能查询大量数据
}

// 安全：限制数量
@GetMapping("/users")
public List<User> getUsers(@RequestParam List<Long> ids) {
    if (ids.size() > 100) {
        throw new IllegalArgumentException("Too many IDs");
    }
    return userService.findByIds(ids);
}
```

**审计要点**：
- 是否限制批量操作数量
- 是否有频率限制
- 是否需要验证码
- 是否有资源消耗保护
- 是否记录异常行为

### 状态机漏洞

**订单状态跳转**：
```python
# 不安全：未验证状态转换
def update_order_status(order_id, new_status):
    order = Order.query.get(order_id)
    order.status = new_status  # 可以任意跳转状态
    db.session.commit()

# 安全：验证状态转换
VALID_TRANSITIONS = {
    'pending': ['paid', 'cancelled'],
    'paid': ['shipped', 'refunded'],
    'shipped': ['delivered', 'returned'],
    'delivered': ['completed'],
}

def update_order_status(order_id, new_status):
    order = Order.query.get(order_id)
    if new_status not in VALID_TRANSITIONS.get(order.status, []):
        raise ValueError(f"Invalid transition from {order.status} to {new_status}")
    order.status = new_status
    db.session.commit()
```

**审计要点**：
- 绘制状态转换图
- 验证所有转换路径
- 检查是否可跳过状态
- 验证回退逻辑
- 检查终态处理

### 权限提升

**水平越权**：
```javascript
// 不安全：未验证资源所有权
app.get('/order/:id', (req, res) => {
    const order = getOrder(req.params.id);
    res.json(order);  // 可以查看他人订单
});

// 安全：验证所有权
app.get('/order/:id', auth, (req, res) => {
    const order = getOrder(req.params.id);
    if (order.userId !== req.user.id) {
        return res.status(403).send('Forbidden');
    }
    res.json(order);
});
```

**垂直越权**：
```java
// 不安全：未验证操作权限
@PostMapping("/user/delete")
public void deleteUser(@RequestParam Long userId) {
    userService.delete(userId);  // 普通用户可删除
}

// 安全：验证角色权限
@PostMapping("/user/delete")
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser(@RequestParam Long userId) {
    userService.delete(userId);
}
```

**审计要点**：
- 是否验证资源所有权
- 是否检查操作权限
- 是否有角色控制
- 是否有细粒度权限
- 是否记录敏感操作

## 审计方法论

### 业务流程分析

**步骤**：
1. 获取业务文档和流程图
2. 识别关键业务节点
3. 分析数据流和状态转换
4. 识别信任边界
5. 列出假设和约束条件

**关注点**：
- 金钱相关操作
- 权限变更操作
- 状态转换逻辑
- 批量操作
- 异常处理

### 威胁建模

**STRIDE模型**：
- **S**poofing (伪造)
- **T**ampering (篡改)
- **R**epudiation (抵赖)
- **I**nformation Disclosure (信息泄露)
- **D**enial of Service (拒绝服务)
- **E**levation of Privilege (权限提升)

**应用**：
对每个业务流程应用STRIDE分析

### 边界条件测试

**测试用例**：
- 零值/负值
- 最大值/最小值
- 空值/null
- 特殊字符
- 并发请求
- 异常顺序

### 时序分析

**关注点**：
- 操作顺序是否可颠倒
- 是否可跳过某些步骤
- 并发操作的影响
- 超时处理
- 重试机制

## 实战案例

### 案例1：优惠券叠加漏洞

**场景**：
电商平台允许使用多张优惠券，但未正确验证叠加规则

**漏洞代码**：
```python
def calculate_total(cart, coupons):
    total = cart.subtotal
    for coupon in coupons:
        if coupon.type == 'percentage':
            total *= (1 - coupon.value / 100)
        elif coupon.type == 'fixed':
            total -= coupon.value
    return max(total, 0)
```

**问题**：
- 未检查优惠券是否可叠加
- 未验证最低消费金额
- 可能导致负价格

**修复**：
```python
def calculate_total(cart, coupons):
    # 验证优惠券叠加规则
    if not can_stack_coupons(coupons):
        raise ValueError("Coupons cannot be stacked")
    
    total = cart.subtotal
    for coupon in coupons:
        # 验证最低消费
        if total < coupon.min_amount:
            continue
            
        if coupon.type == 'percentage':
            discount = total * coupon.value / 100
            total -= min(discount, coupon.max_discount)
        elif coupon.type == 'fixed':
            total -= coupon.value
    
    return max(total, 0.01)  # 最低1分钱
```

### 案例2：条件竞争导致超卖

**场景**：
秒杀活动中，库存检查和扣减不原子，导致超卖

**漏洞代码**：
```java
public boolean purchase(Long productId, int quantity) {
    Product product = productRepository.findById(productId);
    if (product.getStock() >= quantity) {
        // 并发时多个请求都可能通过检查
        product.setStock(product.getStock() - quantity);
        productRepository.save(product);
        return true;
    }
    return false;
}
```

**修复方案1：数据库层面**：
```java
@Transactional
public boolean purchase(Long productId, int quantity) {
    int updated = jdbcTemplate.update(
        "UPDATE products SET stock = stock - ? " +
        "WHERE id = ? AND stock >= ?",
        quantity, productId, quantity
    );
    return updated > 0;
}
```

**修复方案2：分布式锁**：
```java
public boolean purchase(Long productId, int quantity) {
    String lockKey = "product:lock:" + productId;
    try {
        if (redisLock.tryLock(lockKey, 10, TimeUnit.SECONDS)) {
            Product product = productRepository.findById(productId);
            if (product.getStock() >= quantity) {
                product.setStock(product.getStock() - quantity);
                productRepository.save(product);
                return true;
            }
        }
    } finally {
        redisLock.unlock(lockKey);
    }
    return false;
}
```

### 案例3：支付回调伪造

**场景**：
支付回调未验证签名，攻击者可伪造支付成功

**漏洞代码**：
```php
// 支付回调处理
if ($_POST['status'] == 'success') {
    $order_id = $_POST['order_id'];
    updateOrderStatus($order_id, 'paid');
    deliverGoods($order_id);
}
```

**攻击**：
```bash
curl -X POST http://example.com/payment/callback \
  -d "order_id=12345&status=success"
```

**修复**：
```php
// 验证签名
$sign = md5($order_id . $amount . $timestamp . $secret_key);
if ($_POST['sign'] !== $sign) {
    die('Invalid signature');
}

// 验证订单状态
$order = getOrder($_POST['order_id']);
if ($order['status'] !== 'pending') {
    die('Invalid order status');
}

// 验证金额
if ($_POST['amount'] != $order['amount']) {
    die('Amount mismatch');
}

// 更新订单
updateOrderStatus($_POST['order_id'], 'paid');
deliverGoods($_POST['order_id']);
```

## 审计清单

### 支付相关
- [ ] 价格是否由后端控制
- [ ] 是否防止整数溢出
- [ ] 支付回调是否验证签名
- [ ] 是否防止重复支付
- [ ] 退款逻辑是否安全
- [ ] 是否记录支付日志

### 优惠相关
- [ ] 优惠券是否可重复使用
- [ ] 叠加规则是否正确
- [ ] 使用条件是否后端验证
- [ ] 是否检查过期时间
- [ ] 是否限制使用次数

### 积分/余额
- [ ] 是否检查负数
- [ ] 是否防止并发问题
- [ ] 是否检查溢出
- [ ] 转账是否原子操作
- [ ] 是否有操作日志

### 权限控制
- [ ] 是否验证资源所有权
- [ ] 是否检查操作权限
- [ ] 是否有角色控制
- [ ] 是否防止越权
- [ ] 是否记录敏感操作

### 状态管理
- [ ] 状态转换是否合法
- [ ] 是否可跳过状态
- [ ] 是否可回退状态
- [ ] 并发状态变更是否安全
- [ ] 终态是否正确处理

### 批量操作
- [ ] 是否限制操作数量
- [ ] 是否有频率限制
- [ ] 是否需要额外验证
- [ ] 是否有资源保护
- [ ] 是否记录异常行为

---

**AI发挥空间**：
- 深入理解业务逻辑和流程
- 识别业务流程中的安全风险点
- 分析复杂的状态转换逻辑
- 设计针对性的测试用例
- 评估业务漏洞的实际影响
- 提供符合业务需求的修复方案
- 进行威胁建模和风险评估

**反幻觉提示**：
- 业务逻辑漏洞高度依赖具体业务场景
- 不要假设所有业务流程都有相同的安全问题
- 需要深入理解业务规则和约束条件
- 某些看似异常的行为可能是业务需求
- 修复方案需要平衡安全性和业务可用性
- 不同行业和场景有不同的风险容忍度
- 业务逻辑审计需要与业务人员充分沟通
- 测试时注意不要影响生产环境

业务逻辑漏洞审计需要技术能力和业务理解的完美结合。
