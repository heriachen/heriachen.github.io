---
layout:     post
title:      Dependency-Check踩坑指南
subtitle:   Dependency-Check
date:       2020-12-18
author:     heria
header-img: img/post_bg_black.jpg
catalog: true
tags:
---

## 0X01简介

Dependency-Check是OWASP（Open Web Application Security Project）的一个实用开源程序，用于识别项目依赖项并检查是否存在任何已知的，公开披露的漏洞。目前，已支持Java、.NET、Ruby、Node.js、Python等语言编写的程序，并为C/C++构建系统（autoconf和cmake）提供了有限的支持。而且该工具还是OWASP Top 10的解决方案的一部分。
Dependency-Check支持面广（支持多种语言）、可集成性强，作为一款开源工具，在多年来的发展中已经支持和许多主流的软件进行集成，比如：命令行、Ant、Maven、Gradle、Jenkins、Sonar等；具备使用方便，落地简单等优势。



## 0X02搭建

### 命令行

https://owasp.org/www-project-dependency-check/

官网选择command line

![image-20201218102559010](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20201218102559010.png)





或者

https://github.com/jeremylong/DependencyCheck

下载最新版本

无需安装即可使用



常用命令：

![image-20210204142152241](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210204142152241.png)



### DCWeb

https://github.com/he1m4n6a/dcweb

安装方法大致如下

![A](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20201127133608884.png)

但是由于bintray站点失效，docker安装会遇到不可避免且无法解决的错误。因此采用普通安装，记录安装过程中遇到的问题及解决方法。



首先从github下载dcweb

之后上传至需要部署的机器之后解压



之后从https://github.com/jeremylong/DependencyCheck

下载dependencycheck 将文件夹整个mv到dcweb目录下

![image-20201127135109500](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20201127135109500.png)



yum下载安装jdk  python

![image-20201127135245689](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20201127135245689.png)



执行python manage.py runserver 0.0.0.0:8080 （端口可自定义）

![image-20201127135430367](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20201127135430367.png)

报错 Couldn't import Django. Are you sure it's installed and available on your PYTHONPATH environment variable? Did you forget to activate a virtual environment?

执行pip install Django

![image-20201127135617150](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20201127135617150.png)

再次报错

![image-20201127140504325](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20201127140504325.png)

执行pip install --upgrade pip后再次下载django

![image-20201127140709378](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20201127140709378.png)

下载完成后

执行python manage.py runserver 0.0.0.0:8080

![image-20201127141536738](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20201127141536738.png)

此时可访问8080web页面，但是存在问题

![image-20201127141630387](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20201127141630387.png)

后台报错

![image-20201127141644069](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20201127141644069.png)



Invalid HTTP_HOST header: '124.70.149.226:8080'. You may need to add u'124.70.149.226' to ALLOWED_HOSTS.  

此时找到项目中setting.py文件（可用find命令），修改以下字段

![image-20201127141911841](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20201127141911841.png)

改为ALLOWED_HOSTS =['*']

再次访问依旧报错

![image-20201127142036838](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20201127142036838.png)

看到后台如下报错

You have 13 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.



根据提示 python manage.py migrate

![image-20201127142220211](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20201127142220211.png)

再次执行python manage.py runserver 0.0.0.0:8080

![image-20201127143246957](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20201127143246957.png)



上传文件,后台开始执行检测

![image-20201127143413395](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20201127143413395.png)



直接访问报告地址会报错，因此更改下源码

![image-20201127143540592](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20201127143540592.png)

找到app文件夹中的views.py文件

将如下字段更改为自己主机ip及端口

![image-20201127143758080](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20201127143758080.png)



发现原始代码只能生成html文件格式报告

更改位于app下的views.py源码 生成csv格式，但是在查看时报错（文件存在，但是访问时报not found）

![image-20201127152141946](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20201127152141946.png)

使用python 在report文件夹起SimpleHTTPServer服务，之后能够将.csv文件报告文件下载下来

![image-20201127164113655](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20201127164113655.png)

点击连接后自动下载

![image-20201127164137277](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20201127164137277.png)



更改后的代码  

```
# -*- coding: utf-8 -*-

from __future__ import unicode_literals

from django.shortcuts import render,HttpResponseRedirect,HttpResponse

import os
import uuid
import json
import datetime as dt
import hashlib
import random
import requests
import time
import logging


# 项目名字

Project = "app"

# 修改成自己的主机地址

Host = "http://119.45.55.187:8888"
Hostt= "http://119.45.55.187:7777"

# 日志记录

LOG_FORMAT = "%(asctime)s - %(levelname)s - %(message)s"
DATE_FORMAT = "%m/%d/%Y %H:%M:%S %p"
logging.basicConfig(filename='log/scan.log', level=logging.INFO, format=LOG_FORMAT, datefmt=DATE_FORMAT)

def index(request):
    return HttpResponseRedirect("/upload")

def report(request, rid):
    return render(request, 'report/%s.csv' %rid)

def illegalFileName(filename):
    dangerChars = [';','|','"',"'",'&','^','%','$','@']
    for c in dangerChars:
        if c in filename:
            return True
    return False

def upload_files(request):
    if request.method == "POST":
        f = request.FILES.get("uploadfile")
        if f:
            # 设置白名单后缀
            allow_suffix = ['tar', '7z', 'zip', 'gz', 'war', 'so', 'rar', 'bz2', 'arj', 'cab', 'lzh','iso', 'UUE', 'jar', 'rpm']
            file_suffix = f.name.split(".")[-1].lower()
            if file_suffix not in allow_suffix:
                return render(request, 'uploadfile.html', {'warning': '文件格式错误，请重新上传！'})
            if illegalFileName(f.name):
                 return render(request, 'uploadfile.html', {'warning': '文件名非法，请重新上传！'})
            pwd = os.path.abspath('.')
            uploaddir = os.path.abspath('.') + os.sep + 'app/files'
            today = dt.datetime.today()
            dir_name = uploaddir + '/%d/%d/' % (today.year, today.month)
            filename = os.path.join(uploaddir, dir_name, f.name)
            if not os.path.exists(dir_name):
                os.makedirs(dir_name)
            # 保存文件
            try:
                fobj = open(filename, 'wb')
                for chrunk in f.chunks():
                    fobj.write(chrunk)
                fobj.close()
            except Exception as e:
                print str(e)
            firstname = f.name.split('.')[0]
            fr = open(filename, 'r').read()
            md5file = hashlib.md5(fr).hexdigest()
            output =  md5file + '.csv'
            # 执行第三方依赖库的分析
            try:
                logging.info('[+] 开始扫描%s项目, 文件md5值是%s' %(firstname, md5file))
                shell = './dependency-check/bin/dependency-check.sh --project "%s" -f CSV --scan "%s" -o %s' %(firstname, filename, output)
                print shell
                os.system('./dependency-check/bin/dependency-check.sh --project "%s" -f CSV --scan "%s" -o %s' %(firstname, filename, output))
                logging.info('[+] %s项目扫描结束, 文件md5值是%s' %(firstname, md5file))
            except Exception as e:
                print str(e)
                logging.info('[-] %s项目扫描出错, 文件md5值是%s' %(firstname, md5file))
                return render(request, 'uploadfile.html', {'error': '命令执行错误，请联系管理员！'})

            # 网页形式report
            absfilename = pwd + os.sep + output
            absreport = pwd + os.sep + Project + os.sep + 'templates' + os.sep + 'report'
            if not os.path.exists(absreport):
                os.makedirs(absreport)
            absmd5name = absreport + os.sep + md5file + '.csv'
            try:
                os.system('mv %s %s' %(absfilename, absmd5name))
            except Exception as e:
                logging.info('[-] %s项目扫描出错, 文件md5值是%s' %(firstname, md5file))
                return render(request, 'uploadfile.html', {'error': '命令执行错误，请联系管理员！'})
            report_addr = Hostt + os.sep + md5file + ".csv"
            return render(request, 'uploadfile.html', {'success':report_addr})
        else:
            return render(request, 'uploadfile.html', {'warning': '请先选择需要上传的文件！'})
    
    else:
        return render(request,'uploadfile.html')


```



## 0X03使用

dcweb的使用在搭建部分也有涉及，使用相对简单。直接在web页面上传需要检测的项目文件就可以。检测完成后下载报告即可。

但是由于检测指令是写入代码的，因此检测不够灵活。



命令行形式的dc使用方法：

解压后进入dependency-check的bin目录 dependency-check.sh即检测程序

./dependency-check.sh -h 查看命令介绍

常用指令：

./dependency-check.sh --project  项目名（可自定义，不一定是文件名） -s 扫描文件路径 -f  结果类型（html、CSV等） -o 保存路径



## 0X04常见问题

检测不出结果

原因：

1.项目本身不存在漏洞依赖

2.未检测到依赖项（根据检测的经历来看，java项目是根据jar包中的BOOT-INF下lib文件中的第三方依赖来检测，.net检测对应dll,node.js根据package.json或package-lock.json文件）



各种报错：

遇到过的

```
Failed to initialize the RetireJS repo XXXXXXXXX
```

提示jsrepository不存在或为空 ，可查看dependency-check目录data文件夹下jsrepository是否存在或为空，如果为空，去下面的站点下载并将文件替换。

https://github.com/RetireJS/retire.js/blob/master/repository/jsrepository.json



检测不了.net

```
.NET Assembly Analyzer could not be initialized and at least one 'exe' or 'dll' was scanned. The 'dotnet' executable could not be found on the path; either disable the Assembly Analyzer or add the path to dotnet core in the configuration.
[ERROR] ----------------------------------------------------
```

安装dotnet，具体步骤不展示，可自行搜索，需要注意的是有可能安装后依旧无法检测或报错，可能是dotnet的版本问题，dc v6.0.3安装dotnet 5.0.1后依旧报错并无法检测出dll漏洞，卸载dotnet并更换dotnet 3.1后可正常检测无报错.

centos7 安装

sudo rpm -Uvh https://packages.microsoft.com/config/centos/7/packages-microsoft-prod.rpm

sudo yum install dotnet-sdk-3.1



检测node.js时报错 

```
[WARN] No lock file exists - this will result in false negatives; please run `npm install --package-lock`
[WARN] Analyzing `/var/folders/dj/ckvgkdz501v2kzcfw9846znw0000gn/T/dctempd5bb51b0-8461-4d2b-a844-9716d4023d8c/check7877920696590669917tmp/21/package/package.json` - however, the node_modules directory does not exist. Please run `npm install` prior to running dependency-check
[WARN] No lock file exists - this will result in false negatives; please run `npm install --package-lock`
[WARN] Analyzing `/var/folders/dj/ckvgkdz501v2kzcfw9846znw0000gn/T/dctempd5bb51b0-8461-4d2b-a844-9716d4023d8c/check7877920696590669917tmp/5/package/package.json` - however, the node_modules directory does not exist. Please run `npm install` prior to running dependency-check
[WARN] No lock file exists - this will result in false negatives; please run `npm install --package-lock`
```

各种warn ,解决方法：在正常检测语句后添加 --nodeAuditSkipDevDependencies --disableNodeJS即可正常输出结果。



