---
layout:     post
title:     Jlink使用技巧之合并烧写文件
subtitle:	 Jlink系列教程
date:       2019-01-17 12:30:40 +0800
author:     Wang Chao
header-img: img/JLink.jpg
catalog:    true
tag:
    - Keil
    - STM32
    - Jlink
---

### 前言

IAP(In-application-programming)，即在应用中编程。当产品发布之后，可以通过网络方便的升级固件程序，而不需要拆机下载程序。IAP系统的固件一般由两部分组成，即BootLoader Code和Application Code，并存储在不同起始地址的空间里：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/JLINK-MERGE-1.png)

系统运行时，先运行Bootloader程序，检测状态，判断是执行应用程序还是升级固件。在实际开发过程中，这两段程序一般是单独编写，然后生成两个Bin文件，为了方便下载程序，可以把两个文件合并为一个文件，这样会节省很多时间。本文将介绍如何使用JFlash来合并两个Bin文件或者两个Hex文件，

### 准备

- 要合并的文件1：bootloader.hex，起始地址：0x8000000
- 要合并的文件2：app.hex，起始地址：0x20001000，如果是Bin文件要先确定起始地址。
- JFlash软件

### 创建工程

和之前下载程序一样，首先要新建一个工程。

### 1.打开JFlash

![打开JFlash](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/Jlink02-02.jpg)

### 2.创建新工程

点击 File->NewProject

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/Jlink02-03.jpg)

### 3.选择芯片的型号

这里支持很多ARM Cortex内核的芯片，选择对应的芯片，我这里选择的是STM32F103RE系列。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/Jlink02-04.jpg)

### 4.打开要合并的程序文件1：bootloader.hex

点击File -> Open data file，打开bootloader程序。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/JLINK-MERGE-3.jpg)

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/JLINK-MERGE-4.jpg)

### 5.打开要合并的程序文件2：app.hex

点击File -> Merge data file，打开app程序。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/JLINK-MERGE-6.jpg)

要保证，bootloader程序起始地址+bootloader代码大小不超过app程序的起始地址，如下图示意：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/JLINK-MERGE-2.png)

### 6.保存合并后的文件

点击File->Save data file as，将合并后的文件另存，可根据需要选择要保存的文件类型。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/JLINK-MERGE-8.jpg)

### 注意

如果要合并的文件为bin文件，自身不带地址信息，所以会让你指定地址，注意不要互相重叠地址。所以最好各种文件生成的时候就保存为带地址信息的格式，比如hex。关于Hex文件和Bin文件的区别，可以参考文章：[BIN、HEX、AXF、ELF文件格式有什么区别](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247483671&idx=2&sn=e59ee5d6ea3098937bed342cd1c773e0&chksm=fadfa779cda82e6f72b5fbc52d7e6aeda25abf061763bb38655e13611301cde2a5f75dd72dbd#rd)

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/JLINK-MERGE-9.jpg)

### JLink软件的下载

JLink_Windows_V614b软件下载链接：[JLink_Windows_V614b.exe](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/JLink_Windows_V614b.exe)

---

Jlink使用技巧系列文章：

- [Jlink使用技巧之合并烧写文件](http://www.wangchaochao.top/2019/01/17/Jlink-merge/)

- [Jlink使用技巧之烧写SPI Flash存储](http://www.wangchaochao.top/2019/01/12/Jlink-SPI-Flash/)

- [Jlink使用技巧之虚拟串口功能](http://www.wangchaochao.top/2019/01/09/Jlink-UART/)

- [Jlink使用技巧之读取STM32内部的程序](http://www.wangchaochao.top/2019/01/06/Jlink-ReadBack-Hex/)

- [Jlink使用技巧之J-Scope虚拟示波器功能](http://www.wangchaochao.top/2018/10/17/JScope/)

- [Jlink使用技巧之单独下载HEX文件到单片机](http://www.wangchaochao.top/2019/01/05/Jlink-Download-Hex/)

----

欢迎大家关注[我的个人博客](http://www.wangchaochao.top/)

或微信扫码关注我的公众号
