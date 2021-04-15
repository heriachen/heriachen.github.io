---
layout:     post
title:      CrackMe之aLoNg3x.2
subtitle:   160个CrackMe
date:       2021-04-15
author:     heria
header-img: img/post-bg-007.jpeg
catalog: true
tags:
---

## 查壳

![image-20210402171957962](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210402171957962.png)

依旧是delphi编写的无壳程序

## 分析流程

dede分析，看到有个隐藏按钮

![image-20210402172205320](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210402172205320.png)



首先分析register,通过dede直接可以定位到0x442F28

![image-20210402172259892](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210402172259892.png)

运行程序开始测试

![image-20210402172626403](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210402172626403.png)



Register控件ID为2CC，通过控件ID常量搜索

![image-20210415141830306](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210415141830306.png)

发现两处

![image-20210415142104856](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210415142104856.png)

跟进第一处0x442FC4，根据aLoNg3x经验，推测上一个红框中设dl=0，即隐藏Register按钮，第二个框中dl=1和0x2E8（Again按钮的ID） ，推测显示Again

![image-20210415142348935](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210415142348935.png)

所以如果程序执行到这边就可以将register隐藏，将again显示

从函数入口0x442F28处向下分析



0x442F4E处函数获取codice数据，0x442F5E处转为16进制

![image-20210402172814483](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210402172814483.png)

0x442F60处[local.1]与0比较，相等则跳转，目前该值为0，触发跳转，分析一下

![image-20210402173248464](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210402173248464.png)

锁定0x18F5DC堆栈，测试输入，发现当输入为纯数字时，在经过0x442F59时值变为0,触发跳转，跳过弹窗和对0x445830的赋值操作

![image-20210402173731467](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210402173731467.png)

不为纯数字时，该值变成不为0的一个值，执行弹窗，并对0x445830赋值

![image-20210402173832697](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210402173832697.png)





![image-20210415150801697](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210415150801697.png)

codice如果是纯数字，则执行如下的流程

0x442FAF-0x442FB4分别获取nome、codice(hex)、和0x445830的值，作为参数，传入0x442FB4的函数，之后根据结果做判断决定运行流程，如果合规则隐藏Register，显示Again.因此需要跟进分析算法

![image-20210415150953664](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210415150953664.png)

![image-20210415151003359](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210415151003359.png)

![image-20210415151035232](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210415151035232.png)



## 分析算法

首先分析0x445830这个变量

该变量初值为0

![image-20210415155034168](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210415155034168.png)

测试输入，直到输入长度为6时，该变量不为0，并且测试数据“AAAAAA”,会被固定赋值0x1686

![image-20210415155311995](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210415155311995.png)

由于该变量输入输出皆可控，可不纠结算法，直接固定输入值即可，将0x445830值固定为0x1686



F7跟进0x442FB9分析算法

0x4429D7处要求用户名长度大于4

![image-20210415160400594](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210415160400594.png)

该处是个双重循环

0x442A06取用户名第i位ascii(当前i为1,即第一位)

![image-20210415161022588](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210415161022588.png)

0x442A0E取用户名最后一位

之后做乘法运算，值放在edx

0x61*0x68=0x2768

![image-20210415161341131](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210415161341131.png)

0x442A19结果累加存放ebx

0x442A1B用户名长度自减，开始循环

![image-20210415161726598](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210415161726598.png)

每次将用户名第i位与最后一位相乘，之后再乘edi,结果累加存放到ebx.而edi值就是之前固定的0x1686，因此如果输入纯数字，无论怎么计算都是0，不可能通过合规判断

![image-20210415163747386](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210415163747386.png)

内循环结束后，0x442A20处esi自增，当前i=2(上面第一次循环时i=1),即获取用户名第2位

![image-20210415162219653](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210415162219653.png)

0x442A0E依旧从获取最后一位开始，和之前一样的循环

![image-20210415162240515](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210415162240515.png)

循环结束后eax保存结果

之后cdq扩展寄存器，进行取模运算，而0xA2C2A即是十进制的666666，eax=eax%0xA2C2A    结果记为1

![image-20210415164027306](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210415164027306.png)



0x442A37取注册码，取模0x59,商存放eax,余数存在edx,再将商赋给ecx

![image-20210415164427446](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210415164427446.png)

之后重新取注册码，取模0x50，商存放eax,余数存放edx

![image-20210415165751858](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210415165751858.png)

ecx=ecx+edx+1,即ecx=注册码/0x59+注册码%0x50+1，结果记为2

![image-20210415170032926](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210415170032926.png)



判断结果1是否等于结果2，若相等则消除Register

## 总结

总结一下算法流程：

codice总共需要输入两次第一次用于给0x445830变量赋值，此处可将输入固定为“AAAAAA”,将变量值固定为0x1686。

第二次codice的输入为纯数字（记为num），需要根据nome进行如下运算，计算出codice

- 用户名长度大于4（为避免溢出，长度固定为5）
- 双重循环，计算的是用户名的第一位和最后一位的乘积，然后再乘以刚刚用用户名计算出来的被固定的值（0x1686）。内层循环变换用户名最后一位，每次往前移动一位。外层循环变换用户名第一位，每次往后移动一位。结果累加记为tmp。
- result1=tmp%0xA2C2A即十进制666666
- result2=num/0x59+num%0x50+1
- 判断两个result是否相同，相同则按钮消失

## 注册机

代码:

```
def reg():
    while(1):
        name=input("请输入长度为5的nome:")
        if(len(name)!=5):
            print("nome长度需为5")
        else:
            leng=len(name)
            result1=0
            for x in range(leng):
                for y in range(leng):
                    temp=ord(name[x])*ord(name[-y]) 
                    tmp=temp*5766 #乘0x1686
                    result1=result1+tmp
            result1=result1%666666
            codice=(80-((result1-1)*89%80))+(result1-1)*89
            print(codice)
        break
reg()

```

![image-20210415180118672](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210415180118672.png)

![image-20210415175845587](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210415175845587.png)

成功将register消除，弹出again按钮



again按钮流程与register完全一样

首先输入“AAAAAA”固定输出，关闭弹窗，再将之前生成的codice输入即可消除按钮

![image-20210415180519865](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210415180519865.png)

