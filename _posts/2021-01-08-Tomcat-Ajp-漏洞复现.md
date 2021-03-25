---
layout:     post
title:      CVE-2020-1938漏洞复现
subtitle:   Tomcat Ajp漏洞
date:       2021-01-08
author:     heria
header-img: img/post-bg-swift2.jpg
catalog: true
tags:
---



## 0X01漏洞简介

Tomcat 服务器是一个免费的开放源代码的Web 应用服务器，属于轻量级应用服务器，在中小型系统和并发访问用户不是很多的场合下被普遍使用，是开发和调试JSP 程序的首选。由于Tomcat默认开启的AJP服务（8009端口）存在一处文件包含缺陷，攻击者可构造恶意的请求包进行文件包含操作，进而读取受影响Tomcat服务器上的Web目录文件。



编号：CVE-2020-1938/CNVD-2020-10487

细节：Tomcat服务器存在文件包含漏洞，攻击者可利用该漏洞读取或包含Tomcat上所有webapp目录下的任意文件，如：webapp配置文件或源代码等

## **0X02影响范围**

Apache Tomcat 6

Apache Tomcat 7 < 7.0.100

Apache Tomcat 8 < 8.5.51

Apache Tomcat 9 < 9.0.31

## 0X03环境搭建

通过docker搭建

```text
docker search tomcat-8.5.32
docker pull duonghuuphuc/tomcat-8.5.32
docker run -d -p 8080:8080 -p 8009:8009 --name ghostcat duonghuuphuc/tomcat-8.5.32
```

![image-20210108105444235](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210108105444235.png)

运行完毕后查看

![image-20210108105631859](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210108105631859.png)



创建测试文件用于检测漏洞

![image-20210108110119805](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210108110119805.png)



https://www.onebug.org/sectools/76114.html

攻击机使用exp

![image-20210108161132024](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210108161132024.png)

成功读取漏洞主机上的文件