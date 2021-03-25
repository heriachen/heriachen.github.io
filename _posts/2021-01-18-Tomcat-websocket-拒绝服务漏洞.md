---
layout:     post
title:      CVE-2020-13935漏洞复现
subtitle:   Tomcat websocket 拒绝服务漏洞
date:       2021-01-18
author:     heria
header-img: img/post-bg-coffee.jpeg
catalog: true
tags:
---



## 0X01漏洞简介

cve-2020-13935   Apache Tomcat中的WebSocket存在安全漏洞，该漏洞源于程序没有正确验证payload的长度。攻击者可利用该漏洞造成拒绝服务（无限循环）。

## 0X02漏洞影响范围

Apache Tomcat 10.0.0-M1-10.0.0-M6

Apache Tomcat 9.0.0.M1-9.0.36

Apache Tomcat 8.5.0-8.5.56

Apache Tomcat 7.0.27-7.0.104

## 0X03复现

用docker安装好的tomcat环境，本次选取的是漏洞验证环境vulhub，github地址是https://github.com/vulhub/vulhub的tomcat/CVE-2020-1938这个漏洞版本，这个版本也存在cve-2020-13935的漏洞

![image-20210325181533802](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210325181533802.png)



exp地址：

https://github.com/RedTeamPentesting/CVE-2020-13935

安装说明步骤进行编译会报错，这里需要修改proxy地址，命令：go env -w GOPROXY=https://goproxy.cn

go build

编译成功

![image-20210325181604539](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210325181604539.png)



受害机器提前打开top

![image-20210325181630250](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210325181630250.png)



./tcdos    ws://119.45.55.187:8080/examples/websocket/echoStreamAnnotation

![image-20210325181739406](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210325181739406.png)



## 0X04修复

Apache Tomcat更新版本

其他方式：禁用或限制对WebSockets的访问

  **（1）、**升级至Apache Tomcat至安全版本及其以上。

  **（2）、**若无特殊需要，关闭Tomcat Websocket功能，**删除example下的websocket示例**。

  **（3）、**相关链接：https://blog.redteam-pentesting.de/2020/websocket-vulnerability-tomcat/