# 移动安全

移动应用安全涵盖 Android 和 iOS 平台的应用渗透测试、逆向分析和漏洞挖掘。

## Android 安全

### 环境搭建
- Android Studio 和 SDK
- 模拟器（AVD、Genymotion）
- 真机调试
- Root 环境（Magisk）
- 抓包环境（Burp、Charles、mitmproxy）

### 静态分析

#### APK 结构
- AndroidManifest.xml
- classes.dex
- resources.arsc
- lib/（native 库）
- assets/

#### 反编译工具
- apktool（资源反编译）
- dex2jar + JD-GUI
- jadx
- Ghidra（native 分析）

#### 代码审计
- 硬编码敏感信息
- 不安全的加密
- 组件暴露
- WebView 漏洞
- 深度链接漏洞
- Intent 劫持

### 动态分析

#### 调试技术
- adb 调试
- JDWP 调试
- IDA/Ghidra 动态调试
- Frida Hook

#### Hook 框架
- Frida（JavaScript Hook）
- Xposed（系统级 Hook）
- LSPosed
- Magisk 模块

#### 常见 Hook 点
- 加密解密函数
- 网络请求
- 签名校验
- Root 检测
- SSL Pinning

### 组件安全

#### Activity
- Intent 注入
- 权限绕过
- 界面劫持

#### Service
- 未授权访问
- 权限提升
- 拒绝服务

#### Broadcast Receiver
- 广播劫持
- 敏感信息泄露
- 权限绕过

#### Content Provider
- SQL 注入
- 路径穿越
- 权限绕过
- 数据泄露

### 数据存储

#### 本地存储
- SharedPreferences
- SQLite 数据库
- 文件存储
- External Storage

#### 加密存储
- Android Keystore
- 加密算法分析
- 密钥管理

### 网络通信

#### 抓包技术
- HTTP/HTTPS 代理
- SSL Pinning 绕过
- VPN 抓包
- iptables 重定向

#### 协议分析
- 自定义协议
- 加密通信
- 签名验证
- 重放攻击

### Native 层

#### SO 库分析
- IDA Pro/Ghidra
- 反混淆
- 算法还原
- JNI 分析

#### Hook Native
- Frida Native Hook
- Substrate
- Inline Hook

## iOS 安全

### 环境搭建
- Xcode 和 iOS SDK
- 模拟器
- 越狱设备（unc0ver、checkra1n）
- SSH 访问
- 抓包环境

### 静态分析

#### IPA 结构
- Info.plist
- 可执行文件
- Frameworks
- PlugIns
- 资源文件

#### 反编译工具
- class-dump
- Hopper Disassembler
- IDA Pro/Ghidra
- Frida-ios-dump

#### 代码审计
- 硬编码敏感信息
- 不安全的数据存储
- URL Scheme 漏洞
- WebView 漏洞
- 加密实现

### 动态分析

#### 调试技术
- lldb 调试
- debugserver
- Xcode 调试

#### Hook 框架
- Frida
- Cycript
- Substrate
- Dobby

#### 常见 Hook 点
- 加密解密
- 网络请求
- 越狱检测
- SSL Pinning
- 签名校验

### 数据存储

#### 本地存储
- NSUserDefaults
- Keychain
- SQLite
- Plist 文件
- Core Data

#### 加密分析
- Keychain 访问
- 数据保护类
- 加密算法

### 网络通信
- Charles/Burp 抓包
- SSL Pinning 绕过
- 协议分析
- 证书信任

### 沙箱与权限
- 沙箱机制
- 权限申请
- URL Scheme
- Universal Links

## 通用测试

### 业务逻辑
- 认证授权
- 支付逻辑
- 优惠券滥用
- 条件竞争
- 重放攻击

### API 安全
- 认证绕过
- 越权访问
- 参数篡改
- 批量操作
- 速率限制

### 加密安全
- 弱加密算法
- 密钥硬编码
- 可预测的随机数
- 加密模式错误

### 隐私安全
- 敏感信息泄露
- 日志泄露
- 剪贴板泄露
- 截屏泄露

## 自动化测试

### 工具
- MobSF（静态+动态）
- Drozer（Android）
- Objection（Frida 封装）
- AppMon
- 自定义脚本

## 反幻觉机制

### 平台准确性
- Android/iOS 版本特性准确
- 工具使用方法正确
- 系统机制描述准确
- 不编造不存在的漏洞

### 实践可行性
- 命令可执行
- 代码可运行
- 环境要求明确
- 限制条件说明

---

移动安全需要对 Android/iOS 系统有深入理解，建议从基础开始逐步深入。
