---
layout:     post
title:      CVE-2021-3156漏洞复现
subtitle:   Linux sudo权限提升漏洞
date:       2020-02-01
author:     heria
header-img: img/post_bg_wa.jpg
catalog: true
tags:
---



## 0X01漏洞简介

Sudo是一个功能强大的工具，其允许普通用户执行root权限命令，大多数基于Unix和Linux的操作系统都包含sudo。

当在类Unix的操作系统上执行命令时，非root用户可以使用sudo命令来以root用户身份执行命令。由于sudo错误地在参数中转义了反斜杠导致堆缓冲区溢出，从而允许任何本地用户（无论是否在sudoers文件中）获得root权限，无需进行身份验证，且攻击者不需要知道用户密码。

当sudo通过 -s 或 -i 命令行选项在shell模式下运行命令时，它将在命令参数中使用反斜杠转义特殊字符。但使用 -s 或 -i 标志运行 sudoedit 时，实际上并未进行转义，从而可能导致缓冲区溢出。因此只要存在sudoers文件（通常是 /etc/sudoers），攻击者就可以使用本地普通用户利用sudo获得系统root权限。

安全研究人员于1月26日公开披露了此漏洞，并表示该漏洞已经隐藏了近十年。



## 0X02影响版本

- sudo 1.8.2 - 1.8.31p2
- Sudo 1.9.0 - 1.9.5p1

不受影响版本：sudo =>1.9.5p2



## 0X03复现

验证是否存在漏洞

在普通用户权限上，输入：sudoedit -s /

如果显示sudoedit: /: not a regular file，则表示该漏洞存在

![image-20210201085837633](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210201085837633.png)

安全版本同样命令以usage开头

![image-20210201112615228](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210201112615228.png)

Poc:https:*//haxx.in/CVE-2021-3156_nss_poc_ubuntu.tar.gz*

解压：tar -xvzf CVE-2021-3156_nss_poc_ubuntu.tar.gz

![image-20210201090645091](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210201090645091.png)



编译并运行make &&./sudo-hax-me-a-sandwich 0

![image-20210201090908003](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210201090908003.png)

提示没有gcc

yum 安装gcc

再次运行，存在报错，error: 'for' loop initial declarations are only allowed in C99 mode

![image-20210201091648662](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210201091648662.png)

这是因为在gcc中直接在for循环中初始化了增量：

1. for(int i=0; i<len; i++) { 
2. } 

这语法在gcc中是错误的，必须先先定义i变量：

1. int i; 
2. for(i=0;i<len;i++){ 
3.  
4. } 


这是因为gcc基于c89标准，换成C99标准就可以在for循环内定义i变量了

![image-20210201092007604](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210201092007604.png)

全部解决后依旧无法成功复现，查看poc代码，发现只针对Ubuntu&&Debian

![image-20210201093958122](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210201093958122.png)



更换环境



虚拟机搭建**ubuntu 20.04 LTS**环境

![image-20210202152435788](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210202152435788.png)

相同的攻击步骤

![image-20210202152517691](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210202152517691.png)

提权成功



## 0X04修复

目前官方已在sudo新版本1.9.5p2中修复了该漏洞，请受影响的用户尽快升级版本进行防护，官方下载链接：

https://www.sudo.ws/download.html

### 离线升级

下载不在漏洞影响范围内的安装包，例如 sudo-1.9.5p2。下载地址：[https://www.sudo.ws/dist/](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.sudo.ws%2Fdist%2F)

进入root权限解压安装包，进入安装目录，执行以下命令即可：

```bash
./configure --prefix=/usr --libexecdir=/usr/lib --with-secure-path --with-all-insults --with-env-editor --docdir=/usr/share/doc/sudo-1.9.5p2 --with-passprompt="[sudo] password for %p: " && make && make install && ln -sfv libsudo_util.so.0.0.0 /usr/lib/sudo/libsudo_util.so.0
```

![image-20210201112526160](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210201112526160.png)

![image-20210201112610561](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210201112610561.png)

### 在线升级

yum &&apt升级sudo （centos的yum源中暂无最新版sudo）

## 0X05参考链接

https://www.cnblogs.com/potatsoSec/p/14339620.html

https://mp.weixin.qq.com/s?__biz=MzkwNzE5ODAwOQ==&mid=2247484217&idx=1&sn=73efc0db03a6f660945a7459b0fb9998&chksm=c0dda86ff7aa2179025a5498c4245cdfdea68a5e3482775570820bfc759fd41b630a35ee1953&scene=21#wechat_redirect

https://mp.weixin.qq.com/s/RN4oB0a8HB5_FYc_4XxUeQ

https://mp.weixin.qq.com/s?__biz=MzkyMjE1NzQ2MA==&mid=2247483890&idx=1&sn=c338b2edb49dc12b0c19fb994523d6ed&scene=21#wechat_redirect