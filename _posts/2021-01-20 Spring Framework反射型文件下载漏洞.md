---
layout:     post
title:      Spring Framework反射型文件下载漏洞复现
subtitle:   Spring Framework漏洞
date:       2021-01-20
author:     heria
header-img: img/post-bg-f.jpg
catalog: true
tags:
---

## 0X01漏洞简介

CVE-2020-5421 可通过jsessionid路径参数，绕过防御RFD攻击的保护。先前针对RFD的防护是为应对 CVE-2015-5211 添加的。

```
RFD:反射型文件下载漏洞(RFD)是一种攻击技术，通过从受信任的域虚拟下载文件，攻击者可以获得对受害者计算机的完全访问权限
```

![image-20210203105316782](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210203105316782.png)

在Spring Framework版本5.2.0-5.2.8、5.1.0-5.1.17、5.0.0-5.0.18、4.3.0-4.3.28和更旧的不受支持版本中，可能会绕过CVE-2015-5211对RFD攻击的保护，具体取决于通过使用jsessionid路径参数使用的浏览器。

## 0X02CVE-2015-5211复现

/spring/input?content=calc

![image-20210120153107508](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210120153107508.png)

/spring/input.json?content=calc

![image-20210120153124757](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210120153124757.png)

/spring/input.bat?content=111

![image-20210120152855592](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210120152855592.png)

此时通过浏览器访问，浏览器会将响应内容保存为文件，在响应头未指定Content-Disposition的情况下，根据URL保存文件名为input.bat，此时若用户点击下载的文件，将执行input.bat中的命令。RFD漏洞即通过钓鱼诱骗的方式，以可信的网站地址欺骗用户下载执行恶意脚本。
Spring针对CVE-2015-5211的修复方式是指定一个后缀的白名单，白名单外的后缀文件类型将统一添加一个响应头Content-Disposition: inline;filename=f.txt，此时下载文件后将保存为文件名f.txt，即使用户点击该文件也不会执行恶意脚本

(在/src/main/filter/springJsessionidRdfFilter中)

![image-20210120164607927](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210120164607927.png)

## 0X03CVE-2020-5421复现

CVE-2020-5421是针对CVE-2015-5211修复方式的绕过，定位到CVE-2015-5211的修复代码
org.springframework.web.servlet.mvc.method.annotation.AbstractMessageConverterMethodProcessor. addContentDispositionHeader

![image-20210120155622426](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210120155622426.png)

跟进rawUrlPathHelper.getOriginatingRequestUri方法，一路跟进定位到org.springframework.web.util.removeJsessionid方法中会将请求url中;jsessionid=字符串开始进行截断(或者下一个;前)

![image-20210120155719835](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210120155719835.png)

此时关注spring另一个特性，URL路径中添加;开头字符串的path仍可正常访问请求，如
/spring/input?content=111
/spring/;xxx/input?content=111
是可以得到同样响应的

![image-20210120155838828](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210120155838828.png)



结合上述删除;jsessionid=的代码片段，删除;jsessionid=之后CVE-2015-5211的后续防御代码即将获取不到请求的真实后缀文件名，得到CVE-2020-5421的绕过payload
/spring/;jsessionid=/input.bat?content=calc

下载f.txt 内容为calc

根据参考链接构建的环境并不能实现运行bat，仅能下载内容到f.txt文件内。查找后发现在/src/main/filter/springJsessionidRdfFilter中doFilter进行判断并添加了http头内容。

![image-20210120163551694](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210120163551694.png)

删除后运行

![image-20210120163907385](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210120163907385.png)



双击运行弹出计算器。



## 0X04参考链接

https://www.xf1433.com/4595.html

http://cn-sec.com/archives/153847.html

https://github.com/wangmengqiang001/CVE-2020-5421/tree/c_2015_5211

https://github.com/pandaMingx/CVE-2020-5421