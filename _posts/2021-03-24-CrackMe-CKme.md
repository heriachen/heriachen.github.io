---
layout:     post
title:      CrackMe之CKme
subtitle:   160个CrackMe
date:       2021-3-24
author:     heria
header-img: img/post_bg_maple.jpg
catalog: false
tags:
---



查壳

发现是delphi编写的，无壳

![image-20210324144823577](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324144823577.png)





运行程序，发现没有注册按钮

![image-20210324144936754](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324144936754.png)



借助Delphi反编译工具Darkde4来帮助我们寻找突破口

forms下

发现图片位置其实是个按钮panel1，并可点击

![image-20210324145059720](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324145059720.png)



procedures下

![image-20210324145338583](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324145338583.png)

这里是整个程序的所有的响应事件 以及对应的RVA

第一个FormCreate是窗体的创建事件 这个不必关心，一般创建事件都是显示图形界面相关的操作
第二个事件名是chkcode，全称应该是checkcode，校验代码，至于校验的是什么代码？不知道
第三个KeyUp是响应的键盘的弹起
第四个DbClick是按钮的双击事件
第五个Click是按钮的单击事件

那么思路和突破口也就有了，对应单击和双击事件的RVA 直接去OD分析两个响应事件的具体实现部分。



**单击事件**

在单击事件的RVA 0x00457B8处下断点，随便输入一组用户名，从上往下分析所有的执行过程

执行到0x457FEA处，发现获取输入的用户名及长度，推测0x457FEA处call的函数用于获取用户名和长度

![image-20210324150154495](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324150154495.png)



0x457FF2处 eax(用户名长度)+0x1E

![image-20210324150352778](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324150352778.png)



将计算结果转换为字符串5+1E=0x23=35

![image-20210324150705378](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324150705378.png)



0x458009处再次调用获取用户名函数

![image-20210324151038387](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324151038387.png)



0x0045804B处将0转为字符串

![image-20210324151140912](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324151140912.png)

在0x458026处call该函数进行某种运算之后开始循环

![image-20210324152048238](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324152048238.png)



在第四次循环的时候跟进0x458026处的函数

![image-20210324151859140](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324151859140.png)

跟进后发现将之前的计算结果35和输入名称和循环次数拼接，之后return会0x45802B处

共循环18次

![image-20210324170412815](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324170412815.png)



判断ds:[esi+0x30C]是否是0x85,不等则跳转，跳过注册成功

![image-20210324170650333](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324170650333.png)





双击事件

分析

找到该处下断点

![image-20210324171245237](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324171245237.png)

分析同上，发现在0x457EF5处cmp比较ds:[esi+0x30c]和0x3E的值，如不相同则执行跳转，相同则执行0x00457EFE处的mov指令，将0x85值赋给ds:[esi+0x30c],

![image-20210324171230355](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324171230355.png)



因此，ds:[esi+0x30c]值必须为0x3E



根据上面的分析，我们能猜测，肯定有一个地方是把0x3E赋值给了[esi+0x30C]。

直接在OD中，右键->查找所有常量，输入3E，看看能不能找到mov [esi+0x30C],0x3E这样一条指令。如果能，那么这个就是真正校验的地方。



![image-20210324172010238](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324172010238.png)

找到该地址，往上拉到函数最上，下断点分析

![image-20210324172220557](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324172220557.png)



在刚输入注册码时就断下来了。应该响应的是编辑框变换的响应事件，向下执行

0x457c66时获取用户名长度

0x457c6c 长度加5

![image-20210324172802687](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324172802687.png)

0x457c6f “黑头Sun Bird”

![image-20210324172847066](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324172847066.png)



0x457c7a将长度（5+5=0A）转为字符串

![image-20210324173126248](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324173126248.png)

0x457C82字符串“dseloffc-012-OK”

![image-20210324173344344](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324173344344.png)



0x457c86拼接 方式“黑头Sun Bird（用户名长度+5）dseloffc-012-OK（用户名）”

![image-20210324173555098](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324173555098.png)



输入用户名和上面分析得到的注册码，首先双击之后单击就能注册成功

![image-20210324174000522](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324174000522.png)



python 编写注册机

def registry(a):

  name=a

  one="黑头Sun Bird"

  two=len(name)+5

  thr="dseloffc-012-OK"

  key=one+str(two)+thr+name

  return key

  \#print(key)



b=input("请输入用户名:")

print("注册码：",registry(b))

![image-20210324175251428](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324175251428.png)