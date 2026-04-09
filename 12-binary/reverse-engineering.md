# 逆向工程

逆向工程是分析二进制程序以理解其功能、发现漏洞或绕过保护的技术。

## 基础知识

### 汇编语言

**x86/x64 架构**：
- 寄存器（EAX, EBX, ECX, EDX, ESP, EBP, EIP）
- 指令集（MOV, PUSH, POP, CALL, JMP, CMP, TEST）
- 调用约定（cdecl, stdcall, fastcall）

**ARM 架构**：
- 寄存器（R0-R15, SP, LR, PC）
- Thumb 指令集
- AAPCS 调用约定

### 文件格式

**PE (Portable Executable)**：
- Windows 可执行文件
- DOS Header, NT Headers, Sections
- Import/Export Table
- Resource Section

**ELF (Executable and Linkable Format)**：
- Linux 可执行文件
- ELF Header, Program Headers, Section Headers
- .text, .data, .bss, .plt, .got

**Mach-O**：
- macOS/iOS 可执行文件
- Header, Load Commands, Segments

## 静态分析

### 反汇编工具

**IDA Pro**：
- 行业标准
- 强大的反汇编和反编译
- 插件生态系统
- F5 反编译

**Ghidra**：
- NSA 开源工具
- 反编译器
- 脚本支持（Python, Java）
- 协作功能

**Binary Ninja**：
- 现代化界面
- 中级语言（BNIL）
- Python API
- 插件系统

**Radare2/Cutter**：
- 开源框架
- 命令行和 GUI
- 强大的脚本能力

### 字符串分析

**提取字符串**：
```bash
strings binary
strings -e l binary  # Unicode
rabin2 -z binary
```

**关键字搜索**：
- 错误信息
- 调试信息
- API 名称
- 文件路径
- URL

### 符号和导入

**查看导入函数**：
```bash
# Linux
objdump -T binary
readelf -s binary

# Windows
dumpbin /imports binary.exe
```

**分析**：
- 识别功能模块
- 发现敏感 API
- 理解程序行为

### 控制流分析

**函数识别**：
- 函数序言（prologue）
- 函数尾声（epilogue）
- 调用图

**基本块**：
- 顺序执行的指令序列
- 控制流图（CFG）
- 循环和分支

## 动态分析

### 调试器

**GDB**：
```bash
gdb ./binary
break main
run
step / next / continue
info registers
x/10x $esp
```

**WinDbg**：
```
bp main
g
p
t
r
dd esp
```

**x64dbg/x32dbg**：
- 现代化 Windows 调试器
- 插件支持
- 脚本功能

**LLDB**：
- macOS/iOS 调试
- LLVM 项目

### 断点技术

**软件断点**：
- INT3 指令（0xCC）
- 修改代码

**硬件断点**：
- 调试寄存器（DR0-DR7）
- 不修改代码
- 数量有限（4个）

**条件断点**：
- 满足条件时触发
- 减少中断次数

### 动态插桩

**Frida**：
```javascript
// Hook 函数
Interceptor.attach(Module.findExportByName(null, "strcmp"), {
    onEnter: function(args) {
        console.log("strcmp called");
        console.log("arg1: " + Memory.readUtf8String(args[0]));
        console.log("arg2: " + Memory.readUtf8String(args[1]));
    },
    onLeave: function(retval) {
        console.log("return: " + retval);
    }
});
```

**Pin**：
- Intel 动态插桩框架
- 指令级插桩
- 性能分析

**DynamoRIO**：
- 运行时代码操作
- 跨平台

## 反混淆

### 代码混淆技术

**控制流平坦化**：
- 使用状态机
- 隐藏真实控制流

**虚假控制流**：
- 添加永不执行的代码
- 混淆分析

**指令替换**：
- 等价指令替换
- 增加复杂度

**字符串加密**：
- 运行时解密
- 隐藏敏感信息

### 去混淆方法

**符号执行**：
- angr
- Triton
- Manticore

**动态跟踪**：
- 记录实际执行路径
- 忽略虚假分支

**模式识别**：
- 识别混淆器特征
- 自动化去混淆

## 反调试技术

### 检测方法

**IsDebuggerPresent**：
```c
if (IsDebuggerPresent()) {
    exit(1);
}
```

**PEB 检查**：
```c
// Windows
if (NtCurrentPeb()->BeingDebugged) {
    exit(1);
}
```

**时间检测**：
```c
DWORD start = GetTickCount();
// 一些操作
DWORD end = GetTickCount();
if (end - start > threshold) {
    // 检测到调试器
}
```

**异常处理**：
- 触发异常
- 检查处理方式

### 绕过方法

**修改标志**：
- 清除 BeingDebugged
- 修改返回值

**Hook API**：
- 拦截反调试 API
- 返回正常值

**插件**：
- ScyllaHide
- TitanHide
- 自动化绕过

## 加壳与脱壳

### 常见壳

**UPX**：
- 开源压缩壳
- 容易脱壳

**Themida/VMProtect**：
- 商业保护
- 虚拟化保护
- 难以脱壳

**ASPack/PECompact**：
- 压缩壳
- 中等难度

### 脱壳方法

**自动脱壳**：
```bash
upx -d packed.exe
```

**手动脱壳**：
1. 找到 OEP（Original Entry Point）
2. Dump 内存
3. 修复 Import Table
4. 重建 PE

**工具**：
- OllyDumpEx
- Scylla
- ImpREC

## 漏洞分析

### 漏洞类型

**内存破坏**：
- 缓冲区溢出
- 堆溢出
- Use-After-Free
- Double Free

**逻辑漏洞**：
- 整数溢出
- 格式化字符串
- 条件竞争

### 分析方法

**代码审计**：
- 危险函数（strcpy, sprintf, gets）
- 边界检查
- 整数运算

**Diff 分析**：
- 补丁对比
- 识别修复的漏洞
- 1-day 利用

**Fuzzing 结合**：
- 触发崩溃
- 逆向分析崩溃原因
- 开发利用

## 协议逆向

### 网络协议

**抓包分析**：
- Wireshark
- tcpdump
- 识别协议结构

**重放攻击**：
- 捕获数据包
- 修改和重放
- 测试验证

### 加密协议

**识别加密**：
- 高熵数据
- 加密库调用
- 密钥管理

**破解方法**：
- 提取密钥
- 识别算法
- 实现解密

## 自动化分析

### 脚本化

**IDA Python**：
```python
import idaapi
import idc

# 遍历所有函数
for func_ea in Functions():
    func_name = idc.get_func_name(func_ea)
    print(f"Function: {func_name} at {hex(func_ea)}")
```

**Ghidra Script**：
```python
from ghidra.program.model.block import BasicBlockModel

bbm = BasicBlockModel(currentProgram)
for bb in bbm.getCodeBlocks(monitor):
    print(bb)
```

### 批量分析

**YARA 规则**：
```yara
rule suspicious_api {
    strings:
        $api1 = "VirtualAlloc"
        $api2 = "WriteProcessMemory"
        $api3 = "CreateRemoteThread"
    condition:
        all of them
}
```

**自动化框架**：
- Cuckoo Sandbox
- CAPE Sandbox
- 批量样本分析

## 实战技巧

### 快速定位关键代码
- 从字符串入手
- 追踪关键 API
- 交叉引用分析

### 理解程序逻辑
- 绘制流程图
- 注释关键代码
- 重命名函数和变量

### 文档记录
- 记录分析过程
- 标注重要发现
- 便于后续利用

---

**AI 发挥空间**：
- 根据二进制特征选择合适的分析工具和方法
- 分析汇编代码理解程序逻辑
- 识别和绕过反调试和混淆技术
- 开发自动化逆向分析脚本

**反幻觉提示**：
- 不要假设所有二进制都使用相同的编译器或保护
- 验证分析工具对目标文件格式的支持
- 测试分析结果的准确性
- 注意不同架构和平台的差异
- 某些保护技术可能需要专门的绕过方法
