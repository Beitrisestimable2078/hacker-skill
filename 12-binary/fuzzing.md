# 模糊测试 (Fuzzing)

模糊测试是一种自动化漏洞发现技术，通过向程序输入大量随机或变异的数据来触发异常。

## Fuzzing 基础

### 核心概念

**Fuzzer 组成**：
- 输入生成器
- 执行引擎
- 监控机制
- 崩溃分析

**覆盖率**：
- 代码覆盖率
- 路径覆盖率
- 分支覆盖率

**变异策略**：
- 随机变异
- 基于语法
- 基于反馈

### Fuzzing 类型

**黑盒 Fuzzing**：
- 无需源码
- 随机输入
- 效率较低

**白盒 Fuzzing**：
- 需要源码
- 符号执行
- 路径探索

**灰盒 Fuzzing**：
- 插桩获取反馈
- 覆盖率引导
- 效率最高

## AFL (American Fuzzy Lop)

### 基本使用

**编译目标**：
```bash
# C/C++
afl-gcc -o target target.c
afl-g++ -o target target.cpp

# LLVM 模式（推荐）
afl-clang-fast -o target target.c
```

**启动 Fuzzing**：
```bash
# 创建输入目录
mkdir input output
echo "test" > input/seed

# 开始 fuzzing
afl-fuzz -i input -o output ./target @@
```

**多核 Fuzzing**：
```bash
# Master
afl-fuzz -i input -o output -M fuzzer1 ./target @@

# Slave
afl-fuzz -i input -o output -S fuzzer2 ./target @@
afl-fuzz -i input -o output -S fuzzer3 ./target @@
```

### AFL 工作原理

**插桩**：
- 编译时插入代码
- 记录分支覆盖
- 生成覆盖率图

**变异策略**：
- Bit flip
- Byte flip
- Arithmetic
- Interest values
- Dictionary
- Havoc
- Splice

**调度算法**：
- 优先选择高覆盖率输入
- 能量分配
- 路径探索

### AFL 优化

**字典**：
```bash
# 创建字典文件
echo 'keyword1="GET"' > dict.txt
echo 'keyword2="POST"' >> dict.txt

# 使用字典
afl-fuzz -i input -o output -x dict.txt ./target @@
```

**持久化模式**：
```c
__AFL_FUZZ_INIT();

while (__AFL_LOOP(1000)) {
    // 读取输入
    // 处理输入
}
```

**QEMU 模式**：
```bash
# 无源码二进制
afl-fuzz -Q -i input -o output ./binary @@
```

## AFL++

### 增强特性

**更多变异策略**：
- MOpt mutator
- Redqueen
- CMPLOG

**更好的插桩**：
- LLVM 插桩
- QEMU 插桩
- Frida 插桩

**使用**：
```bash
# 编译
afl-clang-fast++ -o target target.cpp

# CMPLOG
afl-clang-fast -o target.cmplog target.c -g -O0
afl-fuzz -i input -o output -c target.cmplog ./target @@

# MOpt
afl-fuzz -i input -o output -L 0 ./target @@
```

## LibFuzzer

### 基本使用

**编写 Fuzz Target**：
```cpp
#include <stdint.h>
#include <stddef.h>

extern "C" int LLVMFuzzerTestOneInput(const uint8_t *data, size_t size) {
    // 处理输入
    if (size > 0) {
        process_data(data, size);
    }
    return 0;
}
```

**编译**：
```bash
clang++ -g -O1 -fsanitize=fuzzer,address target.cpp -o fuzzer
```

**运行**：
```bash
./fuzzer corpus/ -max_len=1024 -timeout=10
```

### 高级特性

**自定义变异**：
```cpp
extern "C" size_t LLVMFuzzerCustomMutator(
    uint8_t *Data, size_t Size, size_t MaxSize, unsigned int Seed) {
    // 自定义变异逻辑
    return new_size;
}
```

**字典**：
```bash
./fuzzer corpus/ -dict=dict.txt
```

**结构化 Fuzzing**：
```cpp
#include <libfuzzer/libfuzzer_macro.h>

DEFINE_PROTO_FUZZER(const MyProto& input) {
    // 处理 protobuf 输入
}
```

## Sanitizers

### AddressSanitizer (ASan)

**编译**：
```bash
clang -fsanitize=address -g -O1 target.c -o target
```

**检测**：
- 堆溢出
- 栈溢出
- Use-after-free
- Double free
- 内存泄漏

### MemorySanitizer (MSan)

**编译**：
```bash
clang -fsanitize=memory -g -O1 target.c -o target
```

**检测**：
- 未初始化内存读取

### UndefinedBehaviorSanitizer (UBSan)

**编译**：
```bash
clang -fsanitize=undefined -g -O1 target.c -o target
```

**检测**：
- 整数溢出
- 空指针解引用
- 未定义行为

### ThreadSanitizer (TSan)

**编译**：
```bash
clang -fsanitize=thread -g -O1 target.c -o target
```

**检测**：
- 数据竞争
- 死锁

## Honggfuzz

### 基本使用

**编译**：
```bash
hfuzz-gcc target.c -o target
```

**运行**：
```bash
honggfuzz -i input -o output -- ./target ___FILE___
```

### 特性

**硬件辅助**：
- Intel PT
- BTS

**反馈驱动**：
- 代码覆盖率
- 比较覆盖率

**持久化模式**：
```c
int main() {
    HF_ITER(&buf, &len) {
        process(buf, len);
    }
}
```

## 网络 Fuzzing

### Boofuzz

**示例**：
```python
from boofuzz import *

session = Session(target=Target(connection=TCPSocketConnection("127.0.0.1", 8080)))

s_initialize("HTTP Request")
s_string("GET", fuzzable=False)
s_delim(" ", fuzzable=False)
s_string("/")
s_delim(" ", fuzzable=False)
s_string("HTTP/1.1", fuzzable=False)
s_static("\r\n")
s_string("Host")
s_delim(":", fuzzable=False)
s_delim(" ", fuzzable=False)
s_string("localhost")
s_static("\r\n\r\n")

session.connect(s_get("HTTP Request"))
session.fuzz()
```

### AFL for Network

**preeny**：
```bash
# 将网络输入转为 stdin
LD_PRELOAD=desock.so afl-fuzz -i input -o output ./server
```

**AFLNet**：
```bash
# 专门用于网络协议
aflnet -i input -o output -N tcp://127.0.0.1/8080 ./server
```

## 内核 Fuzzing

### syzkaller

**配置**：
```json
{
    "target": "linux/amd64",
    "http": "127.0.0.1:56741",
    "workdir": "/workdir",
    "kernel_obj": "/kernel",
    "image": "/image/stretch.img",
    "sshkey": "/image/stretch.id_rsa",
    "syzkaller": "/gopath/src/github.com/google/syzkaller",
    "procs": 8,
    "type": "qemu",
    "vm": {
        "count": 4,
        "kernel": "/kernel/arch/x86/boot/bzImage",
        "cpu": 2,
        "mem": 2048
    }
}
```

**运行**：
```bash
./bin/syz-manager -config=my.cfg
```

### kAFL

**Intel PT 加速**：
- 硬件追踪
- 高性能
- 内核覆盖率

## 崩溃分析

### 崩溃分类

**AFL 崩溃目录**：
```
output/
├── crashes/
│   ├── id:000000,sig:11,src:000000,op:havoc,rep:2
│   └── id:000001,sig:06,src:000001,op:splice,rep:4
└── hangs/
```

**去重**：
```bash
# AFL
afl-cmin -i output/crashes -o unique_crashes ./target @@

# 手动分析
for crash in output/crashes/*; do
    gdb -batch -ex "r < $crash" -ex "bt" ./target
done
```

### 可利用性分析

**Exploitable GDB 插件**：
```bash
gdb ./target
source exploitable.py
exploitable
```

**分类**：
- EXPLOITABLE
- PROBABLY_EXPLOITABLE
- PROBABLY_NOT_EXPLOITABLE
- UNKNOWN

### 自动化分析

**脚本**：
```python
import subprocess

for crash_file in crashes:
    result = subprocess.run(
        ['gdb', '-batch', '-ex', f'r < {crash_file}', '-ex', 'bt', './target'],
        capture_output=True
    )
    analyze_crash(result.stdout)
```

## 高级技术

### 符号执行结合

**angr + AFL**：
```python
import angr
import driller

# 符号执行探索新路径
# AFL fuzzing 快速变异
```

### 语法感知 Fuzzing

**Protobuf**：
```bash
# libprotobuf-mutator
./fuzzer corpus/ -dict=proto.dict
```

**自定义语法**：
```python
# 使用 grammar-based fuzzer
# 生成符合语法的输入
```

### 差分 Fuzzing

**对比多个实现**：
```python
# 相同输入
# 不同实现
# 对比输出差异
```

## 实战技巧

### 种子选择
- 有效输入样本
- 覆盖不同代码路径
- 最小化种子集

### 性能优化
- 多核并行
- 持久化模式
- 减少初始化开销

### 目标选择
- 解析器
- 文件格式处理
- 网络协议
- 复杂输入处理

---

**AI 发挥空间**：
- 根据目标类型选择合适的 Fuzzer
- 设计有效的种子和字典
- 分析崩溃并评估可利用性
- 开发自动化 Fuzzing 和分析流程

**反幻觉提示**：
- 不要假设所有程序都适合 Fuzzing
- 验证 Fuzzer 对目标的支持
- 测试 Fuzzing 配置的有效性
- 注意 Fuzzing 需要大量时间和资源
- 某些漏洞可能需要特定的输入格式或条件
