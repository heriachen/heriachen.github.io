---
layout:     post
title:      CrackMe之AfKayAs.2(续)
subtitle:   160个CrackMe
date:       2021-3-24
author:     heria
header-img: img/post-bg-os-metro.jpg
catalog: true
tags:
---



# 20210324 AfKayAs.2续

先前算法及注册机已经分析编写完毕，但是neg还为去除

task

去除neg

![image-20210324115533326](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324115533326.png)







查壳 

![image-20210324133703292](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324133703292.png)

程序使用VB5.0写的无壳



法1：

## **Timer搜索法**

将内存定位到程序开始处0x401000，右键查找 二进制字串 区分大小写 输入Timer



![image-20210324134154858](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324134154858.png)



找到该位置，右键数据窗口中跟随

![image-20210324134329079](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324134329079.png)

在上面的位置可以看到一个0x1B58，这个是计时器的秒数。也就是十进制的7000,7000毫秒就是7秒。所以第一种去Neg的方法就是将0x1B58改为0x0001

但是此种方法也有一定的局限性，如果程序的作者将计时器的默认名称改掉之后 根本无法在内存中搜索到Timer关键字 也就无法下手。下面介绍一种通用的解决方法

![image-20210324134627270](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324134627270.png)



法2：

## 4C法

对于Visual Basic5/6编译生成的程序，不管自然编译还是伪编译，其程序入口点处的结构都是一样的。来到OEP处

VB程序有个特点-入口点处都是一个PUSH指令,然后一个CALL指令，看JMP 后面跟的MSVBVM50，是VB5.0编写的。(如果不是这种情况的话,那么该程序可能被加过壳)， PUSH将要压入堆栈的是004067D4

转到立即数

![image-20210324135412403](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324135412403.png)



指针0x4067D4指向的结构就是Virtual Basic程序的VBHeader结构



**Virtual Basic程序框架结构**



![image-20210324135543191](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324135543191.png)



**结构定义**

```
Signature		        array[1..4] of char;		//00H签名，必须为VB5！
Flagl			    	WORD；					   //04H.未知标志
LanguageDll	        	array[1..14] of char;		//06H语言链接库名，通常为”*"或者vb6chsdll
BackLanguageDll     	array[1..14] of char;   	//14H.后备语言链接库名
RuntimeDLLVersion 		WORD；					   //22H运行库版本
Languageld			    DWORD：				       //24H语言标识
BackupLanguageID 		DWORD；				       //28H，未知标志
aSubMain		        DWORD；				       //2CH，不为00000000则指向Sub MainO指针
aProjectlnfo		    DWORD					   //30H.指针，指向tProjeclnfo结构  
Plag2					WORD；					  //34H未知标志
Flag3					WORD；					  //36H.未知标志
Flag4					DWORD；					  //38H未知标志
ThreadSpace 			DWORD；					  //3CH.线程空间
Flag5					DWORD；					  //14H未知标志
ResCount				WORD；					  //44H.数量，表示form与cls文件个数
ImportCount				WORD；					  //46H.数量，引用的ocx、dll文件个数
Flag6					BYTE：					 //48H未知，可能代表运行时程序所占内存大小
Flag7				    BYTE；					 //49H未知，可能与程序启动时花费时间有关
Flag8					WORD；					 //4AH.未知
AGUTTable 				DWORD;					 //4CH.指针，指向Form GuI描述表
AExdCompTrble		    DWORD;				      //50H指针，指向“引入的ocx、dll文件描述表
Aproicclpescrinion 		DWORD;					 //54H，指针；指向tProjectDescriptionTable的指针
OProjectExename			DWORD					 //58H.偏移，Offset ProjectExename
OProjectTitle 		    DWORD；				    //5CH，偏移，Offset Projectitle
OHelpFile 				DWORD；					//60H.偏移，Ofset Helpile
OProjectName 			DWORD.					 //64H.偏移，Offset ProjectName
```



**AGUTTable(4C的位置)结构定义**

```
Signature				DWORD					//00H.必须是50000000
FomID					TGUID					//04，可能是以GUID方式命名的formID
Index					BYTE   					//24H 窗体的序号
Flag1					BYTE					//28H 第一个窗体的启动标志，可能是90 也可能是10
AGUIDescriptionTable    DWORD					//48H指针指向以“FFCC…“开始的FormGUI表
Flag3					Dword					//4CH.意义不明
```



在该程序中，偏移4C处为指针，指向406868

![image-20210324140454464](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324140454464.png)

或直接ctrl+G 输入4067D4+4c

![image-20210324140655587](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324140655587.png)

数据窗口继续跟踪：00406820处的DWORD值，00406868，内容如下

![image-20210324140918811](https://raw.githubusercontent.com/heriachen/cloudimg/main/img/image-20210324140918811.png)

看到一块有规律的数据区域

每块50(十六进制)个字节的长度,每块数据的第24(十六进制)个字节处都有一个标志（第一个是00，第二个是01）。其中00和01是窗体的序号 而10是窗体的启动标志。

所以 我们只要把第一个窗体数据块的00改成10 然后把第二个窗体的数据块的10改成00，或者将窗体的序号调换。即可去除neg。