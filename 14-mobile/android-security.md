# Android 安全

Android 是全球最流行的移动操作系统，其安全测试涉及应用分析、系统漏洞、权限滥用等多个方面。

## 环境搭建

### 模拟器
- Android Studio Emulator
- Genymotion
- 夜神、雷电等国产模拟器

### 真机测试
- Root 设备（Magisk）
- 解锁 Bootloader
- 刷入自定义 ROM

### 工具安装
- ADB（Android Debug Bridge）
- Frida
- Objection
- Xposed Framework
- Magisk

## 应用分析

### APK 结构

**解包 APK**：
```bash
# apktool
apktool d app.apk -o output

# jadx
jadx app.apk -d output

# 手动解压
unzip app.apk -d output
```

**APK 组成**：
- AndroidManifest.xml（应用配置）
- classes.dex（Dalvik字节码）
- resources.arsc（资源索引）
- res/（资源文件）
- lib/（原生库）
- META-INF/（签名信息）

### 静态分析

**反编译工具**：
- jadx（Java源码）
- JEB（商业工具）
- Ghidra（原生库）
- IDA Pro（原生库）

**代码审计重点**：
- 硬编码密钥和密码
- 不安全的加密算法
- SQL注入
- 路径遍历
- WebView漏洞
- Intent劫持

**AndroidManifest.xml 分析**：
```xml
<!-- 危险权限 -->
<uses-permission android:name="android.permission.READ_SMS" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />

<!-- 导出组件 -->
<activity android:name=".MainActivity" android:exported="true">
<service android:name=".MyService" android:exported="true">
<receiver android:name=".MyReceiver" android:exported="true">
<provider android:name=".MyProvider" android:exported="true">

<!-- 调试模式 -->
<application android:debuggable="true">

<!-- 备份允许 -->
<application android:allowBackup="true">
```

### 动态分析

**Frida Hook**：
```javascript
// Hook 方法
Java.perform(function() {
    var MainActivity = Java.use("com.example.MainActivity");
    MainActivity.checkPassword.implementation = function(password) {
        console.log("Password: " + password);
        return true;  // 绕过验证
    };
});

// Hook 加密
var Cipher = Java.use("javax.crypto.Cipher");
Cipher.doFinal.overload('[B').implementation = function(data) {
    console.log("Encrypting: " + bytesToHex(data));
    var result = this.doFinal(data);
    console.log("Encrypted: " + bytesToHex(result));
    return result;
};

// Hook 网络请求
var URL = Java.use("java.net.URL");
URL.$init.overload('java.lang.String').implementation = function(url) {
    console.log("URL: " + url);
    return this.$init(url);
};
```

**Objection**：
```bash
# 启动
objection -g com.example.app explore

# 常用命令
android hooking list activities
android hooking list services
android hooking watch class com.example.MainActivity
android intent launch_activity com.example.MainActivity
android sslpinning disable
```

## 常见漏洞

### 组件暴露

**Activity 劫持**：
```bash
# 启动导出的 Activity
adb shell am start -n com.example.app/.SecretActivity

# 传递数据
adb shell am start -n com.example.app/.MainActivity -e key value
```

**Service 滥用**：
```bash
# 启动 Service
adb shell am startservice -n com.example.app/.MyService

# 发送数据
adb shell am startservice -n com.example.app/.MyService --es key value
```

**Broadcast 劫持**：
```bash
# 发送广播
adb shell am broadcast -a com.example.ACTION -e key value
```

**Content Provider 注入**：
```bash
# 查询
adb shell content query --uri content://com.example.provider/table

# 插入
adb shell content insert --uri content://com.example.provider/table --bind name:s:value

# SQL注入
content://com.example.provider/table?id=1' OR '1'='1
```

### WebView 漏洞

**JavaScript 接口**：
```java
// 危险代码
webView.addJavascriptInterface(new JsInterface(), "Android");

// 利用
<script>
Android.sensitiveMethod();
</script>
```

**文件访问**：
```java
// 允许文件访问
webView.getSettings().setAllowFileAccess(true);
webView.getSettings().setAllowFileAccessFromFileURLs(true);

// 利用
file:///data/data/com.example.app/databases/sensitive.db
```

**XSS**：
```java
// 不安全的加载
webView.loadUrl("javascript:" + userInput);
```

### 数据存储

**SharedPreferences**：
```bash
# 读取
adb shell cat /data/data/com.example.app/shared_prefs/config.xml

# 修改（需要root）
adb shell
su
vi /data/data/com.example.app/shared_prefs/config.xml
```

**SQLite 数据库**：
```bash
# 导出数据库
adb pull /data/data/com.example.app/databases/app.db

# 查看
sqlite3 app.db
.tables
SELECT * FROM users;
```

**文件存储**：
```bash
# 检查权限
adb shell ls -la /data/data/com.example.app/files/

# 读取文件
adb shell cat /data/data/com.example.app/files/secret.txt
```

### SSL Pinning 绕过

**Frida 脚本**：
```javascript
Java.perform(function() {
    // TrustManager
    var TrustManager = Java.use("javax.net.ssl.X509TrustManager");
    TrustManager.checkServerTrusted.implementation = function() {
        console.log("SSL Pinning bypassed");
    };
    
    // HostnameVerifier
    var HostnameVerifier = Java.use("javax.net.ssl.HostnameVerifier");
    HostnameVerifier.verify.overload('java.lang.String', 'javax.net.ssl.SSLSession').implementation = function() {
        return true;
    };
});
```

**Objection**：
```bash
android sslpinning disable
```

**Xposed 模块**：
- JustTrustMe
- SSLUnpinning
- TrustMeAlready

### Root 检测绕过

**常见检测方法**：
- 检查 su 文件
- 检查 Magisk/SuperSU
- 检查系统属性
- 检查已安装应用

**绕过方法**：
```javascript
// Frida Hook
Java.perform(function() {
    var File = Java.use("java.io.File");
    File.exists.implementation = function() {
        var path = this.getAbsolutePath();
        if (path.indexOf("su") !== -1 || path.indexOf("magisk") !== -1) {
            return false;
        }
        return this.exists();
    };
});
```

**Magisk Hide**：
- 隐藏 Magisk
- 重命名包名
- 隐藏应用

## 逆向保护绕过

### 代码混淆

**ProGuard/R8**：
- 类名、方法名混淆
- 字符串加密
- 控制流混淆

**分析方法**：
- 动态调试
- Frida Hook
- 日志输出

### 加壳

**常见壳**：
- 梆梆加固
- 360加固
- 腾讯乐固
- 爱加密

**脱壳方法**：
- FART（ART环境脱壳）
- FRIDA-DEXDump
- Dump DEX from memory
- 手动修复

### 反调试

**检测方法**：
- 检查 TracerPid
- 检查调试器端口
- 时间检测
- 异常检测

**绕过**：
```javascript
// Hook TracerPid
var fopen = Module.findExportByName("libc.so", "fopen");
Interceptor.attach(fopen, {
    onEnter: function(args) {
        var path = Memory.readUtf8String(args[0]);
        if (path.indexOf("status") !== -1) {
            // 修改返回值
        }
    }
});
```

## 原生库分析

### SO 文件逆向

**工具**：
- IDA Pro
- Ghidra
- Binary Ninja
- Radare2

**分析重点**：
- JNI 函数
- 加密算法
- 关键逻辑
- 漏洞点

### JNI Hook

**Frida**：
```javascript
// Hook native 函数
var nativeFunc = Module.findExportByName("libnative.so", "Java_com_example_Native_encrypt");
Interceptor.attach(nativeFunc, {
    onEnter: function(args) {
        console.log("Native function called");
        console.log("Arg: " + Memory.readUtf8String(args[2]));
    },
    onLeave: function(retval) {
        console.log("Return: " + retval);
    }
});
```

## 网络流量分析

### 抓包

**工具**：
- Burp Suite
- Charles
- mitmproxy
- Wireshark

**配置代理**：
```bash
# 设置代理
adb shell settings put global http_proxy 192.168.1.100:8080

# 清除代理
adb shell settings put global http_proxy :0
```

**证书安装**：
- 系统证书（需要root）
- 用户证书（Android 7+需要配置）

### 协议分析

**HTTP/HTTPS**：
- 请求参数
- 响应数据
- 认证机制
- 加密方式

**WebSocket**：
- 实时通信
- 消息格式
- 认证流程

**自定义协议**：
- 协议逆向
- 数据格式
- 加密解密

## 权限滥用

### 危险权限

**位置**：
- ACCESS_FINE_LOCATION
- ACCESS_COARSE_LOCATION

**联系人**：
- READ_CONTACTS
- WRITE_CONTACTS

**短信**：
- READ_SMS
- SEND_SMS
- RECEIVE_SMS

**存储**：
- READ_EXTERNAL_STORAGE
- WRITE_EXTERNAL_STORAGE

### 权限提升

**系统漏洞**：
- Dirty COW
- 提权漏洞
- 内核漏洞

**ADB 提权**：
- adb root
- adb remount

## 自动化测试

### MobSF

**安装**：
```bash
docker pull opensecurity/mobile-security-framework-mobsf
docker run -it -p 8000:8000 opensecurity/mobile-security-framework-mobsf
```

**功能**：
- 静态分析
- 动态分析
- 恶意软件检测
- API分析

### Drozer

**安装**：
```bash
pip install drozer
```

**使用**：
```bash
# 连接
drozer console connect

# 枚举组件
run app.package.list
run app.package.info -a com.example.app
run app.activity.info -a com.example.app
run app.service.info -a com.example.app

# 攻击
run app.activity.start --component com.example.app .SecretActivity
run scanner.provider.injection -a com.example.app
```

---

**AI 发挥空间**：
- 根据应用类型选择合适的分析方法
- 识别和利用Android特有的漏洞
- 开发自动化测试和分析脚本
- 绕过各种保护机制

**反幻觉提示**：
- 不要假设所有应用都有相同的保护机制
- 验证目标应用的Android版本和保护措施
- 测试漏洞利用的实际效果
- 注意不同Android版本的API差异
- 某些保护机制可能需要特定的绕过方法
