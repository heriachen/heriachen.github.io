---
layout:     post
title:      CrackMe之Acid burn
subtitle:   160个CrackMe
date:       2021-3-22
author:     heria
header-img: img/post-bg-2015.jpg
catalog: true
tags:
---



exeinfo 发现未加壳

![image-20210322130440677](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210322130440677.png)



运行出现弹框

![image-20210322131012013](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210322131012013.png)

## task1 去除弹框

使用od打开程序

智能搜索，找到关键字对应位置

![image-20210322131120217](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210322131120217.png)

分析上下文，在0x42f797处下断点，执行

![](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210322131120217.png)

确认为0x42f797处调用的函数触发弹窗，将这串参数传递和调用nop即可。执行完毕后跳转到0x425643处。

**更详细的分析：**

bp MessageBoxA下断点，然后跑起来，断到MessageBoxA处，栈中反汇编窗口中跟随，到了调用它的下一条指令处，

![image-20210322135722353](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210322135722353.png)



![image-20210322131856915](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210322131856915.png)

但是这里是不能直接nop或者修改的，因为MessageBox是在程序中某个函数中调用的，有那么多弹窗，不可能就这一处调用，往上稍微看下函数开始位置

![image-20210322134159700](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210322134159700.png)

![image-20210322134336191](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210322134336191.png)

右键转到，挨个F2下断，然后再次跑起来

![image-20210322134447486](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210322134447486.png)

一直断到其中一个，就是之前的0x42f797，右键二进制nop填充即可

![image-20210322142334080](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210322142334080.png)





## task2 序列号

运行后查看序列号错误的关键字 try again!!

![image-20210322142821811](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210322142821811.png)

右键智能搜索找到地址，0x42F4F8，在上面看到注册成功的关键字

![image-20210322143005334](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210322143005334.png)

法一：

爆破

向上找到程序开始处0x42F470下断点

![image-20210322143206657](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210322143206657.png)



在0x42F4d5处发现跳过正确的弹窗直接跳转到failed处

![image-20210322143633456](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210322143633456.png)

因此将0x42F4d5处的jnz替换成nop，跳过跳转即可



法二：

根据上图发现关键字hello dude!

实际上hello dude!即使序列号。

在0x42F4ca处发下能获取我们输入的字符串

![image-20210322143947677](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210322143947677.png)

在0x42F4cd处获取hello dude!

![image-20210322144029435](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210322144029435.png)

我们判断下面0x42F4d0处的函数应该就是判断是否相等的函数

至于hello和dude是哪来的，是在0x42F4AC和0x42f4a4处压栈的

![image-20210322144246938](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210322144246938.png)





## task3 名称和序列号

执行一遍，查看失败时的弹窗

![image-20210322150233383](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210322150233383.png)



在0x42fb21处发现相同信息，再向上找，找到正确信息，往上找到开始处下断点

![image-20210322150924484](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210322150924484.png)



发现在0x42fa57存在判断，将eax中值和4做比较，如果判断不通过则直接try again

![image-20210322152024312](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210322152024312.png)

发现上一栏长度不超过4直接弹框sorry,更换输入，当输入长度为5，此时通过判断，可执行跳转

![image-20210322161733032](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210322161733032.png)

在0x42FAFB处发现字符串CW-7954-CRECKED即为注册码，下面的call函数推测为判断函数，当通过判断则弹出成功的弹窗。

![image-20210322161910644](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210322161910644.png)



也可通过爆破法，直接将0x42fb03处的jnz给nop掉，就可绕过跳转

![image-20210322162301375](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210322162301375.png)





