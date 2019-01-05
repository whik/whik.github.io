---
layout:     post
title:     Jlink使用技巧之单独下载HEX文件到单片机
subtitle:	 Jlink系列教程
date:       2019-01-05 10:30:40 +0800
author:     Wang Chao
header-img: img/JLink.jpg
catalog:    true
tag:
    - Keil
    - STM32
    - Jlink
---

### 前言

上一篇文章介绍了[使用Keil下载单独的Hex文件到单片机内](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247483795&idx=1&sn=591064b0bafc0ab4ba1097d1d2c4f3f3&chksm=fadfa7fdcda82eeb1f4b55e9dbb9f5704634492b2e1344664bf7a713aa85e144abfc2048b511&token=1936031160&lang=zh_CN#rd")，本篇文章介绍，如何使用SEGGER官方软件JFlash来进行程序的下载，支持Hex和Bin文件。

### JFlash的下载和安装

首先，安装JFlash软件，安装完成后，会默认安装JLink驱动程序，主要包含以下几个工具：

- JFlash，主要用于程序下载和读取。
- JFlashLite，JFlash的Mini版
- JFlashSPI，用于给SPI存储器下载程序，如W25Q128。
- JLinkGDBServer，用于第三方软件的调试器，如使用Eclipse搭建STM32开发环境时，就要使用GDB Server来进行调试。
- JLink Command，命令操作窗口，输入指令执行连接，擦除、下载、运行等操作。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/Jlink02-01.jpg)

### 软件准备

- Jlink软件
- Hex文件或者Bin文件
- Jlink调试器，如Jlink V9

### 1.打开JFlash

![打开JFlash](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/Jlink02-02.jpg)

### 2.创建新工程

点击 File->NewProject

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/Jlink02-03.jpg)

### 3.选择芯片的型号

这里支持很多ARM Cortex内核的芯片，选择对应的芯片，我这里选择的是STM32F103RE系列。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/Jlink02-04.jpg)

### 4.连接芯片

如果选择的是SWD模式，就要连接SWDIO、SWCLK、GND这三根线，连接好之后，点击Target->Connect，如果连接成功，在下面的LOG窗口会显示连接成功。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/Jlink02-05.jpg)

### 5.打开烧写文件

JLink支持Hex、Bin等多种文件类型，

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/Jlink02-06.jpg)

这里如果选择的是Bin文件，还需要指定烧写的起始地址，因为Bin文件是不包含烧写地址的，而Hex文件是包含的，具体的区别可以查看之前发的一篇文章：[BIN、HEX、AXF、ELF文件格式有什么区别](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247483671&idx=2&sn=e59ee5d6ea3098937bed342cd1c773e0&chksm=fadfa779cda82e6f72b5fbc52d7e6aeda25abf061763bb38655e13611301cde2a5f75dd72dbd#rd")

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/Jlink02-09.jpg)

### 6.开始烧写

打开Hex文件之后，点击Target->Producion Programming，或者使用快捷键F7，等待几秒之后，程序就下载进去了，下载成功后，会在底部窗口显示烧写成功。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/Jlink02-07.jpg)

### 7.开始运行

烧写成功之后，此时程序还没有运行，点击Target->Manual Programming->Start Application，或者按快捷键F9，程序才开始运行，或者按复位键也可以让程序运行。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/Jlink02-08.jpg)


### 9.工程配置为自动运行

如果想让每次下载完成后，程序自动运行，而不用复位。可以使用工程配置下的自动运行选项。打开Option->Project Setting，切换到Production选项，勾选Start Application，就可以让程序自动运行。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/Jlink02-10.jpg)

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/Jlink02-11.jpg)

可以把当前工程的配置存为一个文件，如STM32F103RE.jflash，下次需要下载时，直接打开这个工程就可以了。

### JLink软件的下载

JLink_Windows_V614b软件下载链接：[JLink_Windows_V614b.exe](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/JLink_Windows_V614b.exe)

---

历史精选文章：

- [Jlink使用技巧之J-Scope虚拟示波器功能](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247483680&idx=1&sn=882e829f182219eb9293d9e010567748&chksm=fadfa74ecda82e58c1455db594d23d3cc121dfe019099cff3f7f297d4cb2459493d940e4b45c#rd)

- [百度智能手环方案开源(含源码，原理图，APP，通信协议等)](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247483787&idx=1&sn=a4d478dd46dfcf94a0c8f1f369062df8&chksm=fadfa7e5cda82ef3c9320aeba5d3e6e6d60dc32d80c570f5412198ec289bfec9e50a4814c8cf&token=1936031160&lang=zh_CN#rd)

- [如何在Keil-MDK开发环境生成Bin格式文件](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247483671&idx=1&sn=20422bf86fd8b58b58be47f2bae8819a&chksm=fadfa779cda82e6f9747c00d2f2ac763eb503f8d46b768c89a5c53a8bda6eb255deded727823&token=855879741&lang=zh_CN#rd)

- [elf格式转换为hex格式文件的两种方法](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247483759&idx=1&sn=eb7ee69807d7c0091f95bc4a98f1ce71&chksm=fadfa701cda82e1748e5005f3726df66027170f82ea8cdfcae9e5f3207ced11d6ab02735a536#rd)

- [两个HC-05蓝牙模块互相绑定构成无线串口模块](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247483709&idx=1&sn=2d5ab85d2cd48ee139d1af056a7019b6&chksm=fadfa753cda82e455883f0958515a139fcf7eb14b8e3da02bb30bf04d260ad6728ad3300c039#rd)

----

欢迎大家关注[我的个人博客](http://www.wangchaochao.top/)

或微信扫码关注我的公众号
