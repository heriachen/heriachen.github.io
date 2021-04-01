---
layout:     post
title:      CrackMe之aLoNg3x.1
subtitle:   160个CrackMe
date:       2021-04-01
author:     heria
header-img: img/post-bg-gr.jpeg
catalog: true
tags:
---

## **查壳**

![image-20210401130718128](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401130718128.png)

由delphi编写 无壳

## 隐藏Cancella按钮控件

运行程序，发现OK按钮被禁用，查看about help,才知道我们需要让OK和cancella隐形，看到Rinzer0的logo

![image-20210401130807995](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401130807995.png)

![image-20210401130851637](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401130851637.png)



通过dede看到两个按钮后面还有个隐藏的图片

![image-20210401131226089](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401131226089.png)



我们需要将这两个按钮隐藏，od打开开始分析，从CancellaClick这个按钮开始，定位到0x442EA8下断点

![image-20210401131721566](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401131721566.png)

输入测试数据，点击cannella按钮

![image-20210401131821821](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401131821821.png)

0x442ECC处获取输入codice数据

![image-20210401132138721](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401132138721.png)

经过0x442ECF函数后看到eax值为7B,而十进制数据123的16进制就是7B,推测该函数功能就是将10进制数据转为16进制放入eax中

![image-20210401132347244](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401132347244.png)



0x442EE3获取Nome输入的数据

![image-20210401132759993](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401132759993.png)

再往下0x442EE7有个关键函数，之后做判断决定是否跳转，分析一下跳过的这些指令，其中0x442EFD处的指令看起来很像是一直标志开关，而紧跟着的0x44EFF中可以看到[ebx+0x2CC]这个2CC看着很眼熟，其实就是OK控件的ID值，我们可以推测，当这段指令不跳过时就可将ok控件隐藏

![image-20210401133216124](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401133216124.png)

![image-20210401133438672](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401133438672.png)

分析一下0x442EE7处的关键函数，看看如何才能不跳过判断

跟进该函数，在0x0442B18处看到eax值变为输入内容的长度，之后再0x442B20处和5比较，小于等于就跳转，并且返回0

![image-20210401134124388](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401134124388.png)

重新输入长度超过5的数据，分析一下跳过的指令

这边进行了一系列的运算，获取第5位字符的ascii码16进制数，之后和7做除法运算，再把余数给到eax,再+2

![image-20210401140400899](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401140400899.png)

我们的测试数据第5位是‘e’,16进制65，除7等于E余3 ，+2后为5

![image-20210401140909740](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401140909740.png)

执行0x442B3F后发现是eax值变为78，跟进该函数可以知道进行了阶乘运算5！=120=0x78

![image-20210401144011090](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401144011090.png)

再往下，看到0x442B57处取到第一位字符16进制ascii码，后面和ESI中的阶乘结果做乘法运算，之后取第二位并开始循环，结果相加存放到ebx

![image-20210401144211825](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401144211825.png)

之后ebx减去codice栏中数据，判断是否为0x7A69，即十进制31337，如果等于那么返回1，不等返回0，

![image-20210401144422432](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401144422432.png)



总结一下这边的运算过程，如何让ok框解禁：

- Nome输入数据长度大于等于5
- 取Nome第5个字符字符ascii码（16进制）
- 除7取余，
- 将余数+2
- 做阶乘
- sum(每一个字符的ascii码×上一步的结果)
- 结果-Codice栏的输入
- 判断结果是否等于31337（0x7A69）



## Cancella注册机

```
import math
def reg():
    while(1):
        x=input("请输入Nome(>5位)：")
        sum=0
        if len(x)<=5:
            print("输入Nome值需>5")
        else:
            five=x[4]    #定位第5个字符
            asc=ord(five)  # 转换为ascii
            res=asc%7+2    #取模并+2
            value=math.factorial(res)   
            for i in range(len(x)):
                sum=sum+ord(x[i])*value
            print("Codice:",sum-31337)
            return 0
reg()
```

![image-20210401165512483](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401165512483.png)

输入后发现原先右边的按钮控件隐藏，同时左边的OK解禁，之后分析左边的OK控件

![image-20210401153043936](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401153043936.png)

## 隐藏OK按钮控件

断点下在0x442D64上，开始分析

![image-20210401153404695](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401153404695.png)

与上面类似，0x442DA6处获取codice内容，0x442DBD获取Nome内容，之后0x442DC1这边是个关键函数，下面进行判断和跳转，跟之前一样，跟进这个关键函数

![image-20210401153645710](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401153645710.png)

0x442BDD处获取codice长度，之后会和5比较，小于等于则跳转，为了不让他跳转，更换长度为6的数据来测试

![image-20210401154232121](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401154232121.png)

0x442C09获取codice数据最后1位ascii码

![image-20210401160233814](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401160233814.png)

之后做平方运算

![image-20210401160723861](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401160723861.png)

![image-20210401160756022](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401160756022.png)

取ax寄存器值，再乘esi数据，即当前数据的数据长度（注意在后面循环时，esi会自减）

![image-20210401161002111](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401161002111.png)



后面是个取模运算，除以0x19

![image-20210401161109528](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401161109528.png)

3F60除0x19等于288余18,商存放在eax,余数存放在edx

![image-20210401161237079](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401161237079.png)

0x442C1D，dx内余数+0x41=59，对应Nome最后一个字符

![image-20210401161717267](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401161717267.png)

之后esi-1,开始循环，对应倒数第二个Nome的字符……重复上述过程

![image-20210401162232437](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401162232437.png)

0x442C28和0x442C2B将之前计算结果和输入的Nome比对，不同则跳转。推测这就是“EIDQFY”就是之前输入的codice对应的验证码

![image-20210401162313204](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401162313204.png)

输入后OK按钮成功隐藏，破解成功

![image-20210401162647983](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401162647983.png)

下面总结OK按钮隐藏的计算过程

- Codice输入长度大于5
- 获取最后一位的ascii码
- 平方
- 乘当前输入当前的codice长度
- 结果除0x19取余数
- 余数+41
- 将对应数值的ascii转换为字符
- 取Codice前一位重复上述步骤，直到全部计算结束



## OK注册机

```
def reg():
    while(1):
        res=""
        x=input("请输入codice(>5位)：")
        if len(x)<=5:
            print("输入codice值需>5")
        else:
            for i in range(len(x)):
                a=ord(x[i]) #获取ascii
                b=a*a*(i+1)   #平方，之后乘当前长度
                c=b%25+65    #取模19后+41
                res=res+chr(c)
            print(res)
            return 0
reg()
```



![image-20210401170823343](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210401170823343.png)