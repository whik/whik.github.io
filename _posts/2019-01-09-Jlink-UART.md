---
layout:     post
title:     Jlink使用技巧之虚拟串口功能
subtitle:	 Jlink系列教程
date:       2019-01-09 21:30:40 +0800
author:     Wang Chao
header-img: img/JLink.jpg
catalog:    true
tag:
    - Keil
    - STM32
    - Jlink
---

### 前言

串口调试是单片机开发过程必不可少的一个功能，一般是使用一个UART-TTL的串口模块来实现串口的功能，其实下载调试使用的Jlink仿真器也可以实现串口调试的功能，本篇文章将介绍如何使用Jlink实现虚拟串口功能，。

### ITM简介

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/JLINK-UART1.jpg)

ITM是ARM Cortex-M系列内核芯片中的一种全新的调试功能，可以方便的通过调试器来实现printf调试功能。来自STM32中文参考手册的介绍：

> ITM ( 指令跟踪微单元 instrumentation trace macrocell)：ITM是一应用驱动的跟踪源，它支持printf类的调试手段来跟踪操作系统(OS)和应用事件，并发布判定的系统信息。ITM以包的形式发布跟踪信息，它由以下部分组成：

> - 软件跟踪：软件可以通过直接写ITM激发寄存器来发布包信息。
> - 硬件跟踪：ITM会发布由DWT产生的信息包。
> - 时间戳：时间戳被发布到相应的包上。ITM包含一个21位的计数器以产生时间戳。Cortex-M3的时钟或串行线观测器(Serial Wire Viewer)的位时钟率给计数器提供时钟。由ITM发送的信息包输出到TPIU(Trace Port Interface Unit)，TPIU再添加一些额外的包(参考TPIU)，然后输出完整的包序列给调试器。用户在设置或使用ITM之前，必需先使能异常调试和监视控制寄存器(Debug Exception and Monitor Control Register)的TRCEN位。

### 1.将ITM端口寄存器定义添加到源代码中

在程序开始处添加以下代码：

	#define ITM_Port8(n)    (*((volatile unsigned char *)(0xE0000000+4*n)))
	#define ITM_Port16(n)   (*((volatile unsigned short*)(0xE0000000+4*n)))
	#define ITM_Port32(n)   (*((volatile unsigned long *)(0xE0000000+4*n)))
	#define DEMCR           (*((volatile unsigned long *)(0xE000EDFC)))
	#define TRCENA          0x01000000

### 2.重定向printf函数

添加重定向printf函数代码：	

	struct __FILE { int handle; /* Add whatever you need here */ };
	FILE __stdout;
	FILE __stdin;
	
	int fputc(int ch, FILE *f) {
	  if (DEMCR & TRCENA) {
	    while (ITM_Port32(0) == 0);
	    ITM_Port8(0) = ch;
	  }
	  return(ch);
	}

以上两段代码可以添加在usart.c文件中，如果文件中已经有了重定向printf的代码，要屏蔽掉，只保留一个重定向。

### 3.在应用程序中添加调试信息

在程序中添加需要输出的调试信息：

	printf("电子电路开发学习:mcu149, Test: %.1f \r\n" ,test += 0.1);

### 4.设置ITM端口0以使能调试功能

在进行调试之前要先使能ITM调试功能，具体配置如下图所示：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/JLINK-UART2.jpg)

### 5.确认硬件的连接方式。

使用ITM调试机制必须使用SWD模式，而且必须要连接SWO，SWO对应JTAG接口的13脚，即至少需要连接4根线。如下图所示：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/JLINK-UART3.jpg)

### 5.打开Debug（printf）窗口

进入Debug调试模式之后，调出Debug(printf)窗口，View - Serial Windows - Debug (printf) Viewer，如下图所示：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/JLINK-UART4.jpg)

程序运行之后，就会在printf窗口看到串口输出的信息。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/JLINK-UART5.jpg)


参考资料：

[Debug (printf) Viewer](http://www.keil.com/support/man/docs/jlink/jlink_trace_itm_viewer.htm)

---

JLink_Windows_V614b软件下载链接：[JLink_Windows_V614b.exe](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/JLink_Windows_V614b.exe)

---

历史精选文章：

- [Jlink使用技巧之虚拟串口功能](http://www.wangchaochao.top/2019/01/09/Jlink-UART/)

- [Jlink使用技巧之读取STM32内部的程序](http://www.wangchaochao.top/2019/01/06/Jlink-ReadBack-Hex/)

- [Jlink使用技巧之J-Scope虚拟示波器功能](http://www.wangchaochao.top/2018/10/17/JScope/)

- [Jlink使用技巧之单独下载HEX文件到单片机](http://www.wangchaochao.top/2019/01/05/Jlink-Download-Hex/)

- [百度智能手环方案开源(含源码，原理图，APP，通信协议等)](http://www.wangchaochao.top/2018/12/27/duband/)

- [elf格式转换为hex格式文件的两种方法](http://www.wangchaochao.top/2018/11/13/elf-to-hex/)

----

欢迎大家关注[我的个人博客](http://www.wangchaochao.top/)

或微信扫码关注我的公众号



