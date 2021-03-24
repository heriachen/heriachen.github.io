---
layout:     post
title:      CrackMe之AfKayAs.2
subtitle:   160个CrackMe
date:       2021-3-18
author:     heria
header-img: img/post-bg-universe.jpg
catalog: true
tags:
---

## 爆破法

尝试输入，弹框后记录关键字

![image-20210318100005984](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318100005984.png)

右键智能搜索

![image-20210318100258255](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318100258255.png)

发现关键字符串，回车或者双击跳转到对应地址

![image-20210318100416813](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318100416813.png)

在you get wrong 上下文查找，发现是从上面跳转过来的，并且该处发现输入正确后的弹窗。

![image-20210318100541192](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318100541192.png)

找到关键跳后，更改运行顺序，使跳转不执行，空格或者右键汇编改为nop

![image-20210318100918559](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318100918559.png)

再次执行，You get it

![image-20210318101001874](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318101001874.png)

右键复制到可执行文件->所有修改->保存



## 逆向分析算法

第一步同样通过智能查找，定位到弹窗位置

之后向上查找以 push ebp   mov ebp,esp开头内容

在0x4080F0出发现，添加断点F8（第一遍不进入，容易绕晕）单步执行

![image-20210318101754355](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318101754355.png)

运行到0x4081EF时发现输入的name，并给到eax寄存器

![image-20210318102124191](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318102124191.png)

在0x4081FB中发现eax值变为5

结合0x4081F5调用的函数，推测该函数作用是获取输入name的长度（函数名称vbalenBstr），给到eax

![image-20210318102229209](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318102229209.png)

再往下0x4081FB中eax赋值给edi,0X408200中 edi乘0x15B38

即5*0x15B38再将结果放到edi中，执行完0x408200后edi中值为0x6C818

![image-20210318103023455](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318103023455.png)

之后的0x408206-0x408213中，发现eax中值为0x43，dx中值“razy”,结合上图中的string='C'，考虑0x40820D中的函数作用是取第一个字母，并将其转化为ascii码

![image-20210318103155685](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318103155685.png)

查看ascii码表，确认猜测

![image-20210318103527347](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318103527347.png)



继续跟进，在0x408216处，发现将edi,edx相加，值0x6C85B存入edi

![image-20210318103636623](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318103636623.png)

到此总结一下执行的运算

len(name)*0x15B38+Ascii(首字母)

输入的样例计算后

5*0x15B38+0x43--->0x6c85b 

![image-20210318104331282](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318104331282.png)



继续执行完0x40821f后发现eax中出现444507的数据，推测是该函数中进行了运算，记录地址，打下断点，后续跟进

![image-20210318104434687](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318104434687.png)

0x4082E3再次出现444507，给到edx压栈

![image-20210318105043170](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318105043170.png)

0x4082EF执行fld指令，数据为浮点数类型10

![image-20210318105147581](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318105147581.png)

*FLD类似于  PUSH指令*
*FSTP类似于 POP指令*
*FADD类似于 ADD指令*



0x4082FE处，执行fdiv指令，浮点数类型10/5，结果存放在ST0

![image-20210318105522650](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318105522650.png)

![image-20210318105617797](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318105617797.png)

*FSTSW　Mem16 将控制寄存器的内容传送到寄存器AX中*



0x40831E 执行faddp st(1),st进行浮点数运算444507+2=444509

![image-20210318110032408](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318110032408.png)



执行到0x408333后444509再次恢复成整型，猜测0x40832D函数作用是将浮点型数据转换为整型

![image-20210318110316871](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318110316871.png)



执行到0x4083FB时发现执行fmul 浮点数相乘 444509*3=1333527

![image-20210318110553216](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318110553216.png)

![image-20210318110827799](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318110827799.png)

0x408404处1333527-2=1333525

![image-20210318110928501](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318110928501.png)

0x40841D处再次将1333525转换为整型，并赋值给edx

![image-20210318111026463](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318111026463.png)

执行到0x4084E5时，发现执行fsub，浮点数1333525-（-15）=1333540

![image-20210318111210547](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318111210547.png)

再往下，执行到0x4085ce时，发现获取了输入的序列号‘12345’，推测0x4085c8处的函数为获取序列号的函数

![image-20210318111434509](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318111434509.png)

继续往下，0x4085f1处执行fdiv，将先前的1333540和12345做除法运算

![image-20210318111631763](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318111631763.png)



再往下回到关键跳，由此猜测刚才的除法运算就是用来判断序列号是否合法的

![image-20210318111904142](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318111904142.png)

再次梳理：

上半部分已总结：

len(name)*0x15B38+Ascii(首字母)

=5*0x15B38+0x43--->0x6c85b 

进行某种运算出现444507的结果

下半部分

（（444507+2）*3-2-（-15））/12345

（（运算结果+2）*3-2-（-15））/序列号



接下来弄清楚444507是通过什么运算得到的

之前在0x40821f处下的断点处跟进，跳转到0x740DBECF

执行到0x740dbef1时已经出现444507的数据，推测是0x740dbee3处的函数中进行了运算，在该地址下断点重新执行并跟进

![image-20210318112952143](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318112952143.png)



跟进到0x7530E94F的地址，跟之前一样，执行到0x7530E984s时已经出现444507，推测时0x7530E97E处的函数进行了运算，下断点跟进

![image-20210318113235566](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318113235566.png)



跳转到0x7530E92A处，执行到0x7530E946,发现堆栈中已经存在444507,推测时0x7530E941处的函数进行了运算，下断点跟进

![image-20210318113634918](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318113634918.png)



跟进到0x7530E896,向下执行，执行到0x7530E8BA处，发现6c85B

![image-20210318114000466](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318114000466.png)

之后pop ebx，发现将0000000A出栈存入ebx

![image-20210318114314046](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318114314046.png)

![image-20210318114344495](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318114344495.png)

0x7530E8c2继续执行div ebx,用6c85b除0A,商存放在eax,余数存放在edx

![image-20210318114525840](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318114525840.png)

6c85b/0A=ADA2--------7

之后add edx 0x30   及余数+30   =37（后续循环中发现这个数值不参加运算）

通过test设置标志位，之后ja条件跳转回上面开始循环

![image-20210318114830548](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318114830548.png)

第二次运算ADA2/0A=115D----------0

![image-20210318115033112](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318115033112.png)

此时eax 中为115D   edx为0（该处30为经过add 0x30运算后的数值）

![image-20210318115244625](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318115244625.png)

再次循环 115D/0A =1BC ----------5

![image-20210318115416049](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318115416049.png)

拼接余数5 0 7后续几次循环略过..由此得出444507的数据

总结一下444507的计算过程

首先由之前的计算结果

6c85b/0a=商---------余数

商/0a--------余数

循环，直到eax为0，test eax，eax 后ja不跳转

至此所有的运算过程全部破解

算法总结：

length(Crazy)=5

5*0x15B38=6c818

6c818+0x43（首字母C的ascii码）=0x6c85b 

（这一串除以0x0a取余运算实际就是将16进制数转换为10进制数）

6c85b/0a=ada2----------7

ada2/0a=115d-----------0

115d/0a=1bc------------5

1bc/0a=2c----------------4

2c/0a=4-------------------4

4/0a=0--------------------4

444507+2=444509

444509*3=1333527

1333527-2=1333525

1333525-(-15)=1333540

1333540/12345 （比较）



在使用计算器的时候也可以发现6c85b其实就是444507的16进制数据

![image-20210318140524729](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210318140524729.png)



## python编写注册机

def cal(name):

  length=len(name)  #获取长度

  firstchar=name[0]  #获取首字母

  asc=ord(firstchar) #转换成ascii码

  a=length*0x15B38+asc #长度*常量+ascii码

  key=(a+2)*3-2+15

  print(key)

