# iOS 安全

iOS 是 Apple 的移动操作系统，以其严格的安全机制著称，但仍存在多种测试和攻击向量。

## 环境搭建

### 越狱设备
- checkra1n（硬件漏洞，支持A5-A11）
- unc0ver（软件漏洞）
- Taurine（软件漏洞）
- Dopamine（iOS 15-16）

### 工具安装
- Cydia/Sileo（包管理器）
- OpenSSH
- Frida
- Cycript
- SSL Kill Switch
- Flex 3

### 模拟器
- Xcode Simulator（有限功能）
- Corellium（商业云模拟器）

## 应用分析

### IPA 结构

**解包 IPA**：
```bash
# 解压
unzip app.ipa -d output

# 查看结构
cd output/Payload/App.app
ls -la
```

**IPA 组成**：
- Info.plist（应用配置）
- 可执行文件（Mach-O）
- _CodeSignature/（签名）
- Frameworks/（框架）
- PlugIns/（插件）
- Assets.car（资源）

### 静态分析

**反编译工具**：
- Hopper Disassembler
- IDA Pro
- Ghidra
- class-dump（导出类信息）

**Info.plist 分析**：
```xml
<!-- URL Scheme -->
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>myapp</string>
        </array>
    </dict>
</array>

<!-- 后台模式 -->
<key>UIBackgroundModes</key>
<array>
    <string>location</string>
    <string>audio</string>
</array>

<!-- ATS 配置 -->
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>
</dict>
```

**class-dump**：
```bash
class-dump -H App -o headers/
```

### 动态分析

**Frida Hook**：
```javascript
// Hook Objective-C 方法
if (ObjC.available) {
    var ViewController = ObjC.classes.ViewController;
    ViewController['- checkPassword:'].implementation = function(password) {
        console.log("Password: " + password);
        return true;
    };
}

// Hook Swift 方法
var moduleName = "App";
var addr = Module.findExportByName(moduleName, "$s3App14ViewControllerC13checkPasswordySbSSF");
Interceptor.attach(addr, {
    onEnter: function(args) {
        console.log("Swift method called");
    }
});

// Hook C 函数
var strcmp = Module.findExportByName(null, "strcmp");
Interceptor.attach(strcmp, {
    onEnter: function(args) {
        console.log("strcmp: " + Memory.readUtf8String(args[0]) + " vs " + Memory.readUtf8String(args[1]));
    }
});
```

**Cycript**：
```bash
# 连接进程
cycript -p App

# 列出类
ObjectiveC.classes

# 调用方法
[UIApplication sharedApplication]
[[UIApplication sharedApplication] keyWindow]

# 修改属性
var vc = [UIApplication sharedApplication].keyWindow.rootViewController
vc.view.backgroundColor = [UIColor redColor]
```

## 常见漏洞

### URL Scheme 劫持

**注册 URL Scheme**：
```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>myapp</string>
        </array>
    </dict>
</array>
```

**调用**：
```bash
# 通过 Safari
open myapp://action?param=value

# 通过命令行
xcrun simctl openurl booted myapp://action?param=value
```

**漏洞**：
- 参数注入
- 未验证来源
- 敏感操作未授权

### Keychain 数据泄露

**读取 Keychain**：
```bash
# 越狱设备
/var/Keychains/keychain-2.db

# 工具
keychain_dumper
```

**不安全的存储**：
```objc
// 不安全：kSecAttrAccessibleAlways
[keychain setObject:password forKey:@"password" 
    accessibility:kSecAttrAccessibleAlways];

// 安全：kSecAttrAccessibleWhenUnlockedThisDeviceOnly
[keychain setObject:password forKey:@"password"
    accessibility:kSecAttrAccessibleWhenUnlockedThisDeviceOnly];
```

### 数据存储

**NSUserDefaults**：
```bash
# 位置
/var/mobile/Containers/Data/Application/<UUID>/Library/Preferences/

# 读取
plutil -p com.example.app.plist
```

**文件存储**：
```bash
# Documents
/var/mobile/Containers/Data/Application/<UUID>/Documents/

# Library
/var/mobile/Containers/Data/Application/<UUID>/Library/

# Caches
/var/mobile/Containers/Data/Application/<UUID>/Library/Caches/
```

**Core Data**：
```bash
# SQLite 数据库
/var/mobile/Containers/Data/Application/<UUID>/Library/Application Support/

# 查看
sqlite3 app.sqlite
```

### SSL Pinning 绕过

**SSL Kill Switch**：
```bash
# Cydia 安装
# 自动绕过 SSL Pinning
```

**Frida 脚本**：
```javascript
// Hook NSURLSession
if (ObjC.available) {
    var NSURLSession = ObjC.classes.NSURLSession;
    var method = NSURLSession['- URLSession:didReceiveChallenge:completionHandler:'];
    
    Interceptor.attach(method.implementation, {
        onEnter: function(args) {
            var completionHandler = new ObjC.Block(args[4]);
            var origImpl = completionHandler.implementation;
            
            completionHandler.implementation = function(disposition, credential) {
                // 信任所有证书
                return origImpl(1, credential);
            };
        }
    });
}
```

### 越狱检测绕过

**常见检测方法**：
- 检查 Cydia
- 检查越狱文件
- 检查系统调用
- 检查沙盒

**绕过方法**：
```javascript
// Hook 文件检查
var fopen = Module.findExportByName(null, "fopen");
Interceptor.attach(fopen, {
    onEnter: function(args) {
        var path = Memory.readUtf8String(args[0]);
        if (path.indexOf("cydia") !== -1 || 
            path.indexOf("substrate") !== -1) {
            args[0] = Memory.allocUtf8String("/dev/null");
        }
    }
});

// Hook stat
var stat = Module.findExportByName(null, "stat");
Interceptor.attach(stat, {
    onEnter: function(args) {
        var path = Memory.readUtf8String(args[0]);
        if (path.indexOf("cydia") !== -1) {
            args[0] = Memory.allocUtf8String("/dev/null");
        }
    }
});
```

**工具**：
- Liberty Lite
- Shadow
- A-Bypass
- FlyJB X

## 逆向保护绕过

### 代码混淆

**Swift/Objective-C 混淆**：
- 符号混淆
- 字符串加密
- 控制流混淆

**分析方法**：
- 动态调试
- Frida Hook
- 符号恢复

### 反调试

**检测方法**：
```c
// ptrace
ptrace(PT_DENY_ATTACH, 0, 0, 0);

// sysctl
int mib[4] = {CTL_KERN, KERN_PROC, KERN_PROC_PID, getpid()};
struct kinfo_proc info;
sysctl(mib, 4, &info, &size, NULL, 0);
if (info.kp_proc.p_flag & P_TRACED) {
    // 检测到调试器
}
```

**绕过**：
```javascript
// Hook ptrace
var ptrace = Module.findExportByName(null, "ptrace");
Interceptor.attach(ptrace, {
    onEnter: function(args) {
        if (args[0].toInt32() === 31) { // PT_DENY_ATTACH
            args[0] = ptr(0);
        }
    }
});

// Hook sysctl
var sysctl = Module.findExportByName(null, "sysctl");
Interceptor.attach(sysctl, {
    onLeave: function(retval) {
        // 修改 p_flag
    }
});
```

## 网络流量分析

### 抓包配置

**代理设置**：
```bash
# WiFi 设置中配置代理
# 或使用 USB 代理
```

**证书安装**：
- 设置 -> 通用 -> 描述文件
- 设置 -> 通用 -> 关于本机 -> 证书信任设置

**工具**：
- Burp Suite
- Charles
- mitmproxy
- Proxyman

### 协议分析

**HTTP/HTTPS**：
- 请求分析
- 响应分析
- 认证机制

**WebSocket**：
- 实时通信
- 消息格式

**自定义协议**：
- 协议逆向
- 加密分析

## 权限滥用

### 隐私权限

**位置**：
- NSLocationWhenInUseUsageDescription
- NSLocationAlwaysUsageDescription

**相机/麦克风**：
- NSCameraUsageDescription
- NSMicrophoneUsageDescription

**照片**：
- NSPhotoLibraryUsageDescription
- NSPhotoLibraryAddUsageDescription

**联系人**：
- NSContactsUsageDescription

### 后台活动

**后台模式**：
- location（位置）
- audio（音频）
- voip（VoIP）
- fetch（后台获取）

## 自动化测试

### Objection

**安装**：
```bash
pip install objection
```

**使用**：
```bash
# 连接
objection -g com.example.app explore

# 常用命令
ios info binary
ios hooking list classes
ios hooking search classes MainActivity
ios keychain dump
ios plist cat Info.plist
ios nsurlcredentialstorage dump
ios cookies get
```

### MobSF

**iOS 支持**：
- 静态分析
- IPA 解包
- 二进制分析
- 配置检查

## 实战技巧

### 应用砸壳

**工具**：
- frida-ios-dump
- Clutch
- bfinject
- CrackerXI+

**frida-ios-dump**：
```bash
# 安装
pip install frida-tools
git clone https://github.com/AloneMonkey/frida-ios-dump
cd frida-ios-dump

# 砸壳
python dump.py com.example.app
```

### 动态调试

**lldb**：
```bash
# 连接
debugserver *:1234 -a App

# 本地连接
lldb
process connect connect://localhost:1234

# 断点
b objc_msgSend
b -[ViewController checkPassword:]
```

### 符号恢复

**restore-symbol**：
```bash
restore-symbol App -o App_restored
```

---

**AI 发挥空间**：
- 根据iOS版本选择合适的测试方法
- 识别和利用iOS特有的漏洞
- 开发自动化测试脚本
- 绕过各种保护机制

**反幻觉提示**：
- 不要假设所有应用都能越狱测试
- 验证目标应用的iOS版本和保护措施
- 测试漏洞利用的实际效果
- 注意不同iOS版本的API差异
- 某些保护机制可能需要特定的绕过方法
- 越狱检测和反调试技术在不断演进
