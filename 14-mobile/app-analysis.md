# 移动应用分析

移动应用分析涵盖跨平台的通用测试方法、业务逻辑漏洞和API安全。

## 通用测试方法

### 业务逻辑漏洞

**支付逻辑**：
- 金额篡改
- 订单重放
- 优惠券滥用
- 积分操作

**认证授权**：
- 会话管理
- Token 泄露
- 权限提升
- 越权访问

**数据验证**：
- 客户端验证绕过
- 服务端验证缺失
- 参数污染

### API 安全

**API 测试**：
```bash
# 抓包分析
# 识别 API 端点
# 测试认证机制
# 参数 Fuzzing
```

**常见问题**：
- 未授权访问
- 敏感信息泄露
- 批量操作
- 速率限制缺失
- IDOR（不安全的直接对象引用）

**GraphQL**：
- 内省查询
- 批量查询
- 深度查询
- 字段建议

### 加密分析

**识别加密**：
- 高熵数据
- 加密库调用
- 密钥管理

**常见问题**：
- 硬编码密钥
- 弱加密算法
- ECB 模式
- 自定义加密

**分析方法**：
- 静态分析找密钥
- 动态 Hook 加密函数
- 流量分析识别模式

## 跨平台框架

### React Native

**调试**：
```bash
# 开启调试菜单
adb shell input keyevent 82  # Android
# iOS: Cmd+D

# Chrome DevTools
chrome://inspect
```

**代码位置**：
- Android: assets/index.android.bundle
- iOS: main.jsbundle

**Hook**：
```javascript
// Frida Hook React Native
Java.perform(function() {
    var ReactNativeHost = Java.use("com.facebook.react.ReactNativeHost");
    ReactNativeHost.getJSBundleFile.implementation = function() {
        console.log("Loading JS bundle");
        return this.getJSBundleFile();
    };
});
```

### Flutter

**分析难点**：
- Dart 编译为原生代码
- 符号被剥离
- 难以反编译

**工具**：
- reFlutter（解包和分析）
- Doldrums（Frida 脚本）

**Hook**：
```javascript
// Hook Flutter 函数
var base = Module.findBaseAddress("libflutter.so");
var offset = 0x123456;
Interceptor.attach(base.add(offset), {
    onEnter: function(args) {
        console.log("Flutter function called");
    }
});
```

### Xamarin

**分析**：
- .NET DLL 文件
- 使用 dnSpy 反编译
- C# 代码

**位置**：
- Android: assemblies/
- iOS: Frameworks/

### Cordova/Ionic

**代码位置**：
- www/ 目录
- HTML/CSS/JavaScript

**分析**：
- 直接查看源码
- 修改 JavaScript
- Hook Cordova 插件

## 自动化测试

### Appium

**安装**：
```bash
npm install -g appium
appium
```

**测试脚本**：
```python
from appium import webdriver

caps = {
    "platformName": "Android",
    "deviceName": "emulator-5554",
    "app": "/path/to/app.apk",
    "automationName": "UiAutomator2"
}

driver = webdriver.Remote("http://localhost:4723/wd/hub", caps)
driver.find_element_by_id("username").send_keys("admin")
driver.find_element_by_id("password").send_keys("password")
driver.find_element_by_id("login").click()
```

### MobSF API

**自动化扫描**：
```python
import requests

# 上传 APK
files = {'file': open('app.apk', 'rb')}
r = requests.post('http://localhost:8000/api/v1/upload', files=files)
scan_hash = r.json()['hash']

# 扫描
r = requests.post('http://localhost:8000/api/v1/scan', data={'hash': scan_hash})

# 获取报告
r = requests.get(f'http://localhost:8000/api/v1/report_json?hash={scan_hash}')
print(r.json())
```

## 实战场景

### 支付绕过

**测试点**：
- 修改支付金额
- 重放支付请求
- 修改商品ID
- 绕过支付验证

**方法**：
```bash
# 抓包修改
# 重放攻击
# 条件竞争
```

### 会话劫持

**Token 泄露**：
- 日志输出
- 本地存储
- 网络传输

**会话固定**：
- Token 不更新
- 可预测 Token

### 数据泄露

**敏感数据**：
- 用户信息
- 支付信息
- 位置数据
- 聊天记录

**泄露途径**：
- 日志
- 截图
- 剪贴板
- 备份

### 逻辑漏洞

**条件竞争**：
- 并发请求
- 重复操作
- 资源竞争

**业务流程**：
- 跳过步骤
- 修改顺序
- 重复执行

## 报告编写

### 漏洞分类

**严重**：
- 支付绕过
- 数据泄露
- 远程代码执行

**高危**：
- 权限提升
- 敏感信息泄露
- 认证绕过

**中危**：
- 信息泄露
- 会话管理
- 加密问题

**低危**：
- 信息收集
- 配置问题

### 报告内容

**漏洞描述**：
- 漏洞位置
- 触发条件
- 影响范围

**复现步骤**：
- 详细步骤
- 截图证明
- 视频演示

**修复建议**：
- 具体方案
- 代码示例
- 最佳实践

---

**AI 发挥空间**：
- 根据应用类型设计测试方案
- 识别业务逻辑漏洞
- 分析加密和混淆
- 开发自动化测试脚本

**反幻觉提示**：
- 不要假设所有应用都有相同的架构
- 验证测试方法对目标应用的适用性
- 测试漏洞的实际影响
- 注意不同框架和平台的差异
- 某些漏洞可能需要特定的业务场景
