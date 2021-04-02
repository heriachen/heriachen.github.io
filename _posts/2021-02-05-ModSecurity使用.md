---
layout:     post
title:      Modsecurity使用
subtitle:   WAF
date:       2021-02-05
author:     heria
header-img: img/post-bg-006.jpeg
catalog: true
tags:
---

## 0X01Centos+modsecurity+nginx

安装相关依赖工具

```bash
yum install -y git wget epel-release
yum install -y gcc-c++ flex bison yajl yajl-devel curl-devel curl GeoIP-devel doxygen zlib-devel pcre-devel lmdb-devel libxml2-devel ssdeep-devel lua-devel libtool autoconf automake
```

![image-20210205134124515](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210205134124515.png)



安装modsecurity

```bash
cd /usr/local
git clone https://github.com/SpiderLabs/ModSecurity
cd ModSecurity
git checkout -b v3/master origin/v3/master
git submodule init
git submodule update
sh build.sh
./configure
make
make install
```

安装nginx与ModSecurity-nginx

```bash
cd /usr/local
git clone https://github.com/SpiderLabs/ModSecurity-nginx
wget http://nginx.org/download/nginx-1.16.1.tar.gz
tar -xvzf nginx-1.16.1.tar.gz
cd /usr/local/nginx-1.16.1
./configure --add-module=/usr/local/ModSecurity-nginx
make
make install
```

启动nginx 此时无防护效果

```bash
/usr/local/nginx/sbin/nginx
```

![image-20210205141407585](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210205141407585.png)

**创建用于存放配置文件的文件夹**

```bash
mkdir /usr/local/nginx/conf/modsecurity
```



将/usr/local/Modsecurity/modsecurity.conf-recommended复制到/usr/local/nginx/conf/modsecurity，并重命名为modsecurity.conf；

将/usr/local/Modsecurity/unicode.mapping复制到/usr/local/nginx/conf/modsecurity；

下载规则包

http://www.modsecurity.cn/download/corerule/owasp-modsecurity-crs-3.3-dev.zip

解压后复制crs-setup.conf.example到/usr/local/nginx/conf/modsecurity/下并重命名为crs-setup.conf；

复制rules文件夹到/usr/local/nginx/conf/modsecurity/下，同时修改REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf.example与RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf.example两个文件的文件名，将".example"删除，可将自己写的规则放置于此两个文件中；



编辑nginx.conf

在http或server节点中添加以下内容（在http节点添加表示全局配置，在server节点添加表示为指定网站配置）：

Bash

```bash
modsecurity on;
modsecurity_rules_file /usr/local/nginx/conf/modsecurity/modsecurity.conf;
```

编辑modsecurity.conf

SecRuleEngine DetectionOnly改为SecRuleEngine On

同时添加以下内容：

Bash

```bash
Include /usr/local/nginx/conf/modsecurity/crs-setup.conf
Include /usr/local/nginx/conf/modsecurity/rules/*.conf
```



重新加载Nginx测试效果

Bash

```bash
/usr/local/nginx/sbin/nginx -s reload
```

测试XSS语句

```
 ?param="><script>alert(1);</script>
```

![image-20210205141606964](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210205141606964.png)





![image-20210205143659312](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210205143659312.png)

参考链接http://www.modsecurity.cn/practice/post/11.html



## 0X02日志审计

查看日志

nginx错误日志或modsecurity日志

触发规则

![image-20210205144528015](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210205144528015.png)

![image-20210205144604740](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210205144604740.png)





## 0X03误拦截处理（白名单）

未添加白名单

![image-20210208131754084](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210208131754084.png)

![image-20210208131736420](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210208131736420.png)



修改REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf

重启生效

![image-20210208131857203](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210208131857203.png)

![image-20210208131938588](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210208131938588.png)

![image-20210208132016791](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210208132016791.png)









## 0X04 自定义规则

modsecurity/rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf 下自己添加规则

http://www.modsecurity.cn/chm/index.html

![image-20210208113921496](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210208113921496.png)





![image-20210208114028228](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210208114028228.png)



![image-20210208114042886](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210208114042886.png)





