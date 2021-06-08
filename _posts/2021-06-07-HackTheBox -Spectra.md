---
layout:     post
title:      HackTheBox Spectra
subtitle:   HackTheBox
date:       2021-06-07
author:     heria
header-img: img/post-bg-026.jpg
catalog: true
tags:
---



![image-20210607114521718](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607114521718.png)

## 信息收集

nmap -sS -sV -sC 10.10.10.229

![image-20210607114808191](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607114808191.png)

开放22,80,3306,8081端口

## 渗透

访问80正常，但是超链接跳转失败

![image-20210607130914396](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607130914396.png)

在/etc/hosts中添加spectra.htb

![image-20210607131022014](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607131022014.png)

查看main页面

![image-20210607131943618](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607131943618.png)

点击后跳转到登录页面，发现存在用户名枚举

![image-20210607132111313](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607132111313.png)



![image-20210607132138958](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607132138958.png)



gobuster扫目录

![image-20210607131754019](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607131754019.png)

扫出的两个目录正好对应两个链接

![image-20210607132227148](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607132227148.png)

```
curl http://spectra.htb/testing/wp-config.php.save
```

![image-20210607132352979](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607132352979.png)

发现数据库连接信息

使用获得的devtest 和devteam01尝试登录失败，用户名改为administrator成功登入

![image-20210607133309060](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607133309060.png)

之后发现wordpress5.4.2版本

使用metasploit中利用模块设置参数

![image-20210607133739578](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607133739578.png)

run 执行后获取到meterpreter

![image-20210607133842533](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607133842533.png)

shell，执行id,当前是nginx用户权限，并没找到user.txt

![image-20210607134136937](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607134136937.png)

根据/etc/passwd和找到的内容，发现还有一个katie的用户

![image-20210607134441774](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607134441774.png)

![image-20210607134351659](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607134351659.png)

在/opt下找到一个名为autologin.conf.orig的文件，里面提示存在/etc/autologin目录

![image-20210607134817262](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607134817262.png)

之后到该目录，cat passwd找到密码

![image-20210607134952602](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607134952602.png)

SSH用katie和获取到的密码进行登录，拿到第一个flag

![image-20210607135150565](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607135150565.png)



## 提权

sudo -l发现可以执行/sbin/initctl

![image-20210607135404167](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607135404167.png)

initctl是作为系统来控制启动和停止的过程

在/etc/init下发现多个test.conf文件

![image-20210607141343528](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607141343528.png)

![image-20210607141450534](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607141450534.png)

替换内容，之后便可用root权限运行

首先停止test服务

![image-20210607141620164](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607141620164.png)

之后更改文件内容

```
vim test.conf
```

![image-20210607141736770](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607141736770.png)

重启运行test

```
sudo /sbin/initctl start test
/bin/bash -p
```

![image-20210607142027634](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607142027634.png)

在/root/root.txt中找到第二个flag

![image-20210607142702554](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607142702554.png)