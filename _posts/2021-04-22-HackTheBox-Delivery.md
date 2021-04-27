---
layout:     post
title:      HackTheBox Delivery
subtitle:   HackTheBox
date:       2021-04-22
author:     heria
header-img: img/post-bg-009.jpeg
catalog: true
tags:
---



![image-20210421164504686](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210421164504686.png)

```
openvpn xxx.ovpn
```

通过vpn连接

```
ping 10.10.10.222
```

测试是否连通

![image-20210421164545132](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210421164545132.png)

## 渗透

扫描

```
nmap -sS -sV -sC -p- 10.10.10.222|tee nmap-scan.txt

-sS  SYN扫描

-sV 扫描端口服务版本信息

-sC 使用默认类别脚本扫描

-p- 指定所有端口
```

并将扫描结果保存到文件

![image-20210421172604283](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210421172604283.png)

发现开放80 8065端口

查看80端口

![image-20210421172744960](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210421172744960.png)

connect us

![image-20210421172849191](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210421172849191.png)

这里有两个超链接，MatterMost server就是8065端口的地址。根据提示信息，需要有一个@delivery.htb.email的邮箱地址才能登陆8065端口的控制台，而这个邮箱应该就是由HelpDesk超链接指向的地址来获取，但是访问发现这个链接地址：http://helpdesk.delivery.htb打开不了

![image-20210421173608390](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210421173608390.png)

修改hosts文件，在/etc/hosts中添加入下数据

```
10.10.10.225 helpdesk.delivery.htb
10.10.10.225 delivery.htb
```

![image-20210421174017129](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210421174017129.png)

open a new ticket

![image-20210422085712091](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210422085712091.png)

返回一个自己域的邮箱地址，如果你想添加更多信息到你的ticket,可以联系这个邮箱地址

![image-20210422085743736](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210422085743736.png)

输入邮箱及ticket id 查询状态，进入查看留言

![image-20210422085852239](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210422085852239.png)

![image-20210422085952641](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210422085952641.png)

现在我们有一个符合条件的邮箱用户，查看8065端口

![image-20210421174730452](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210421174730452.png)

我们只有一个邮箱账号，但是没有密码，点击create one now

注册页面使用9245698@delivery.lib来注册账户，注意有密码强度限制。123456789Test!

![image-20210422090222811](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210422090222811.png)

注册成功

![image-20210421175547262](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210421175547262.png)

返回ticket status的页面，发现收到了验证注册的邮件内容

![image-20210422090318817](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210422090318817.png)

去提示连接验证账户，登录后进到如下界面，发现有root的聊天记录，“Credentials to the server are maildeliverer:Youve_G0t_Mail!  ”.并且提示“PleaseSubscribe!maynot be in RockYou but if any hacker managers to get our hashes,they can use hashcat  rules to easily crack……”可能会使用到hashcat

使用凭据maildeliverer:Youve_G0t_Mail!登录ssh

![image-20210422091114657](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210422091114657.png)

成功登录,无法直接切到root 需要root密码

![image-20210422091732023](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210422091732023.png)

目前可以查看user.txt

![image-20210427142715653](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210427142715653.png)

## 提权

![image-20210422091754676](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210422091754676.png)

在/opt/mattermost/config目录下存在Readme.md，

![image-20210422092254732](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210422092254732.png)

根据提示连接找到官方手册，在里面找到数据库的链接字段

```
https://about.mattermost.com/default-config-docs/
```

![image-20210422092443683](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210422092443683.png)

在同级目录下的config.json文件中，我们发现了对应的字段信息

mmuser:Crack_The_MM_Admin_PW

![image-20210422092642524](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210422092642524.png)



连接数据库

```
mysql -u mmuser -D mattermost -p
```

```
show tables;
```

发现User表

![image-20210422093306655](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210422093306655.png)

```
select username,password from users where username='root';
```

![image-20210422093628303](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210422093628303.png)

发现password

```
$2a$10$VM6EeymRxJ29r8Wjkr8Dtev0O.1STWb4.4ScG.anuu7v0EFJwgjjO
```



根据之前的提示,需要使用hashcat

先分别将需要破解的hash和使用到的“PleaseSubscribe!”字符串写入hash.txt和wordlist文件

![image-20210422100314457](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210422100314457.png)

识别hash类型,使用hashid,判断应该使用了bcrypt,

![image-20210422100945112](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210422100945112.png)

在hashcat中该模式对应的id为3200

![image-20210422101145889](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210422101145889.png)

更多信息可以通过hashcat --help查看

hashcat附带一些可以使用的预定义规则，也可使用github上的

```
git clone git@github.com:praetorian-inc/Hob0Rules.git
```

或者直接访问下载

https://github.com/praetorian-inc/Hob0Rules   

![image-20210422101648121](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210422101648121.png)



执行

```bash
hashcat -a 0 -m 3200 hash.txt wordlist.txt -r Hob0Rules-master/d3adhob0.rule -o cracked.txt -w 3 -O
```

开始破解

![image-20210422102537864](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210422102537864.png)

参数简介

```
-a 0指定攻击模式，零为直接模式。即要使用列表中的所有单词。
-m 3200指定哈希类型。即之前讲的那样，这是一个bcrypt哈希，对于hashcat来说是3200。
hash.txt 包含要破解的哈希的文件
wordlist.txt是我们用于直接模式攻击的“词典”。
-r Hob0Rules-master/d3adhob0.rule 是我们要使用的规则，根据文件中的规则处理字典中的单词。
-o cracked.txt告诉hashcat将破解的哈希值写入一个名为的文件cracked.txt。
-w 3设置工作负载配置文件。将此设置为3可以在桌面上提供更优化的体验，但是它也可能更慢。要利用更多的GPU，请使用的工作负载设置1，但您的桌面可能会滞后。默认设置为2。
-O是的简写--optimized-kernel-enable，这限制了密码的长度。
```

获得root密码 PleaseSubscribe!21

![image-20210422110152024](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210422110152024.png)

su - root切到root输入密码

![image-20210427142734090](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210427142734090.png)

获取root.txt

结束