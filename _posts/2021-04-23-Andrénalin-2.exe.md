---
layout:     post
title:      CrackMe之Andrénalin.2.exe
subtitle:   160个CrackMe
date:       2021-04-23
author:     heria
header-img: img/post-bg-010.jpg
catalog: true
tags:
---

# Andrénalin.2.exe

## 查壳



![image-20210422174509809](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210422174509809.png)

VB程序，无壳

想要分析出这个Crackme，必须具备两个VB的前置知识。一开始并不知道，看了半天没看懂。

- 第一个就是VB的变量特征，VB是采用寄存器接收变量地址，地址是指向一个结构体，首地址存放的是类型编号。如果是字符串则首地址+0x8的位置是字符串的地址
- 第二个VB反汇编中很多时候会在[ebp-0x34]这个堆栈地址中看到最终的计算的结果。

## 分析

输入测试数据开始分析

![image-20210423132600318](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210423132600318.png)

点击ok按钮，断点下在0x401FF0，向下F8

在0x402106处看到call vbaLenVar，获取用户名长度。这里首先看堆栈找到参数的地址，数据窗口跟随，在首地址+0x8处 发现0x554D2C的地址，之后数据窗口跟随，查看到Name变量参数的数据。这就是VB变量的存放规则

![image-20210423133119011](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210423133119011.png)

![image-20210423133154039](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210423133154039.png)



eax返回地址+0x8就是输入Name的数据长度

![image-20210423133254135](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210423133254135.png)

以输入长度做为循环次数

获取用户每一位，后面转换为ascii码

![image-20210423133541726](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210423133541726.png)

0x68位首字母h的ascii

![image-20210423133627869](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210423133627869.png)

0x402195处执行vbavaradd函数，结果可在eax+0x8查看到，实际上在后续循环过程中可以发现，实际是将用户名的ASCII值和前一位的ASCII值相加得出结果，再用vbaVarMove这个函数将结果保存到[ebp-0x34]这个地址

![image-20210423133840506](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210423133840506.png)

获取第二位e

![image-20210423134745318](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210423134745318.png)

执行vbavaradd，结果为CD（0x68+0x65）

![image-20210423135016584](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210423135016584.png)



当循环结束计算值为0x209

![image-20210423135637222](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210423135637222.png)



0x4021E5处 将0x499602D2赋值bep-0xA4,其实就是十进制1234567890

![image-20210423140709811](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210423140709811.png)

![image-20210423141357128](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210423141357128.png)



0x4021F9处vbavarmul，将之前的计算结果和0x499602D2相乘

![image-20210423142308077](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210423142308077.png)

找到ss:[ebp-0x34]，但是并没有看到计算结果，再看看这个vbaVarMul()函数，乘了这么大的数1234567890，应该是存不下去的，猜测转成浮点数了，右键，64bit double

![image-20210423142248783](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210423142248783.png)

![image-20210423142152597](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210423142152597.png)

之后用-替换第4位

![image-20210423142914431](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210423142914431.png)

用-替换第9位

![image-20210423143012295](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210423143012295.png)

得到正确key值643-0987-690

最后就是获取key,在下面的比较函数中进行合规校验。

![image-20210423143145051](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210423143145051.png)



## 算法总结

1.获取Name各个字符的ascii之和

2.将1的结果乘1234567890（0x499602D2）

3.将结果第4和第9位用-替换

4.key值与3结果比较，相等则验证通过



## 注册机

```
def reg():
    while(1):
        name=input("请输入name值:")
        length=len(name)
        sum=0
        for i in range(length):
            sum=sum+ord(name[i])
        tmp=int(sum)*int(1234567890)
        res=list(str(tmp))
        res[3]='-'
        res[8]='-'
        key=''.join(res)
        print(key)
        break
reg()
```

![image-20210423145014231](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210423145014231.png)

验证

![image-20210423145157320](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210423145157320.png)