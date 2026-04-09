# 二进制安全

二进制安全涵盖逆向工程、漏洞开发、PWN、恶意软件分析等领域。

## 逆向工程

### 静态分析

#### 工具选择
- IDA Pro（反汇编和反编译）
- Ghidra（开源逆向平台）
- Binary Ninja
- Radare2/Cutter
- Hopper Disassembler

#### 分析技术
- 控制流分析
- 数据流分析
- 字符串和常量分析
- 函数识别和重命名
- 结构体恢复
- 算法识别

#### 反混淆
- 控制流反混淆
- 字符串解密
- 虚拟机还原
- 花指令去除

### 动态分析

#### 调试器
- x64dbg/x32dbg
- OllyDbg
- WinDbg
- GDB/PEDA/GEF
- LLDB

#### 调试技术
- 断点设置（软件断点、硬件断点、内存断点）
- 单步跟踪
- 内存监控
- API 监控
- 异常处理

#### 反调试对抗
- IsDebuggerPresent 绕过
- PEB 检测绕过
- 时间检测绕过
- 异常处理绕过
- 硬件断点检测

### 代码保护

#### 加壳识别
- PEiD/Detect It Easy
- 手工识别（熵值、节区特征）
- 壳的类型（压缩壳、加密壳、虚拟机壳）

#### 脱壳技术
- 单步脱壳
- ESP 定律
- 内存 dump
- OEP 查找
- IAT 修复

## 漏洞开发

### 漏洞类型

#### 内存破坏
- 栈溢出
- 堆溢出
- UAF（Use After Free）
- 双重释放
- 整数溢出
- 格式化字符串

#### 逻辑漏洞
- 条件竞争
- 类型混淆
- 未初始化变量
- 符号链接攻击

### Exploit 开发

#### 基础技术
- 返回地址覆盖
- Shellcode 编写
- 坏字符处理
- 空间限制处理

#### 高级技术
- ROP（Return-Oriented Programming）
- JOP（Jump-Oriented Programming）
- SROP（Sigreturn-Oriented Programming）
- 堆喷射（Heap Spray）
- 堆风水（Heap Feng Shui）

### 保护机制绕过

#### DEP/NX 绕过
- ROP 链构造
- ret2libc
- mprotect/VirtualProtect

#### ASLR 绕过
- 信息泄露
- 部分覆盖
- 爆破攻击
- 相对地址利用

#### Stack Canary 绕过
- Canary 泄露
- Canary 爆破
- 覆盖 TLS
- 格式化字符串泄露

#### PIE 绕过
- 地址泄露
- 相对偏移
- 部分覆盖

#### CFG/CET 绕过
- 间接调用
- 数据攻击
- JIT 代码

## 内核漏洞

### Windows 内核

#### 漏洞类型
- 池溢出
- UAF
- 类型混淆
- 竞争条件
- 任意地址写

#### 利用技术
- Token 窃取
- EPROCESS 操作
- 回调函数劫持
- 系统调用表修改

### Linux 内核

#### 漏洞类型
- 栈溢出
- 堆溢出
- UAF
- 竞争条件
- 整数溢出

#### 利用技术
- commit_creds(prepare_kernel_cred(0))
- modprobe_path 覆盖
- core_pattern 利用
- userfaultfd 利用

### macOS 内核
- XNU 内核特性
- IOKit 漏洞
- 沙箱逃逸
- 内核扩展

## 浏览器漏洞

### JavaScript 引擎
- V8（Chrome）
- SpiderMonkey（Firefox）
- JavaScriptCore（Safari）
- Chakra（Edge Legacy）

### 漏洞类型
- 类型混淆
- OOB（Out of Bounds）
- UAF
- JIT 漏洞
- 原型链污染

### 沙箱逃逸
- IPC 漏洞
- 渲染进程逃逸
- 浏览器进程提权
- 系统调用滥用

## Fuzzing

### Fuzzing 类型
- 黑盒 Fuzzing
- 白盒 Fuzzing
- 灰盒 Fuzzing
- 覆盖率引导 Fuzzing

### Fuzzing 工具
- AFL/AFL++
- LibFuzzer
- Honggfuzz
- Syzkaller（内核）
- Domato（浏览器）

### Fuzzing 技术
- 变异策略
- 语料库管理
- 覆盖率收集
- Crash 分析
- 去重和最小化

## 符号执行

### 工具
- angr
- Triton
- KLEE
- Manticore

### 应用场景
- 路径探索
- 约束求解
- 漏洞挖掘
- 自动化利用

## 反幻觉机制

### 技术准确性
- 不编造不存在的漏洞编号
- 不虚构工具的功能
- 不确定的技术细节应明确说明
- 引用真实的 CVE 和技术文档

### 实践验证
- 提供的命令应该是可执行的
- 代码示例应该是可编译的
- 工具使用应该符合实际
- 技术原理应该准确

### 诚实表达
- 不确定时说"需要根据具体情况"
- 复杂问题说"需要深入分析"
- 未知领域说"建议查阅相关文档"
- 避免绝对化表述

---

二进制安全需要扎实的底层知识和大量的实践经验，建议从基础开始逐步深入。
