---
layout:     post
title:     Jlink使用技巧之读取STM32内部的程序
subtitle:	 Jlink系列教程
date:       2019-01-06 19:30:40 +0800
author:     Wang Chao
header-img: img/JLink.jpg
catalog:    true
tag:
    - Keil
    - STM32
    - Jlink
---

### 前言

上一篇Jlink系列文章介绍了如何使用J-Flash来下载Hex或Bin文件到单片机，具体可参考[Jlink使用技巧之单独下载HEX文件到单片机](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247483804&idx=1&sn=46ed9785a48d421325fa2f4b03a6c11c&chksm=fadfa7f2cda82ee4205020653b469ef9959d29a3a3a0c3922c8870bd17b877f8e80ac0918961&token=703787322&lang=zh_CN#rd)，本篇文章介绍，如何使用JFlash来读取单片机的程序，学习单片机程序文件的读取，不是为了破解别人的程序，而是学习破解的原理，从而更好保护自己的程序不被破解，希望大家也能尊重他人的劳动成果。

### JFlash的下载和安装

首先，安装JFlash软件，安装完成后，会默认安装JLink驱动程序，主要包含以下几个工具：

- JFlash，主要用于程序下载和读取。
- JFlashLite，JFlash的Mini版
- JFlashSPI，用于给SPI存储器下载程序，如W25Q128。
- JLinkGDBServer，用于第三方软件的调试器，如使用Eclipse搭建STM32开发环境时，就要使用GDB Server来进行调试。
- JLink Command，命令操作窗口，输入指令执行连接，擦除、下载、运行等操作。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/Jlink02-01.jpg)

### 软件准备

- Jlink软件，J-Flash
- Jlink调试器，如Jlink V9
- 单片机开发板，如STM32F103RET6

### 1.打开JFlash

![打开JFlash](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/Jlink02-02.jpg)

### 2.创建新工程

点击 File->NewProject

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/Jlink02-03.jpg)

### 3.选择芯片的型号

这里支持很多ARM Cortex内核的芯片，选择要读取单片机对应的芯片型号，我这里选择的是STM32F103RE系列。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/Jlink02-04.jpg)

### 4.连接芯片

如果选择的是SWD模式，就要连接SWDIO、SWCLK、GND这三根线，连接好之后，点击Target->Connect，如果连接成功，在下面的LOG窗口会显示连接成功。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/Jlink02-05.jpg)

### 5.读取单片机内的程序
重点来了！选择Target->Manual Programming ->Read Back，一共有三个选项，用于读取不同的Flash地址范围。

- Selected sectors

被选择的扇区，可以在工程配置选项Project settings->Flash，查看哪些扇区被选择了。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/ReadBack-02.jpg)

- Entire chip

整个Flash区域，一般选择这个选项，读取整个Flash区域的程序

- Range

手动指定读取的Flash地址范围。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/ReadBack-03.jpg)

这里我们选择Entire chip就可以了，读取整个Flash区域，地址范围：0x8000000~0x807FFFF

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/ReadBack-01.jpg)

等几秒钟，就可以看到底部窗口显示读取成功的信息。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/ReadBack-04.jpg)

### 6.保存读取到的程序

选项File-> Save data file或者是Save data file as，保存类型根据需要选择，建议选择Hex格式，已经包含了地址信息。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/ReadBack-05.jpg)

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/ReadBack-06.jpg)

### 7.程序的验证。

怎么验证读取到的程序是正确的呢？很简单，重新烧写进去，看运行现象和原来的是不是一样就行了。

具体操作方法查看上一篇Jlink系列文章：[Jlink使用技巧之单独下载HEX文件到单片机](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247483804&idx=1&sn=46ed9785a48d421325fa2f4b03a6c11c&chksm=fadfa7f2cda82ee4205020653b469ef9959d29a3a3a0c3922c8870bd17b877f8e80ac0918961&token=703787322&lang=zh_CN#rd)

---

### 总结

既然能这么简单的读取到单片机的程序，那么我们自己的程序应该如何保护起来呢？很显然，我们可以对Flash设置读保护功能，即大家说的“加密”功能，可以防止对Flash的非法访问，这里的加密是针对整个Flash区域的，如果设置了读保护功能，那么程序只能正常的从RAM中加载运行，而不能通过调试器读出来，那么别人就不能破解了。哈哈！

具体怎么实现呢？

这里先介绍几个关于Flash保护操作的几个库函数：

	FLASH_Unlock();   //Flash解锁
	FLASH_ReadOutProtection(DISABLE);  //Flash读保护禁止  
	FLASH_ReadOutProtection(ENABLE);   //Flash读保护允许

这个函数在固件库stm32f10x_flash.h中，使用这个功能要先添加这个库文件。

设置读保护：
	
	void Set_Protect(void)
	{
		if(FLASH_GetReadOutProtectionStatus() != SET)
		{
			FLASH_Unlock();
			FLASH_ReadOutProtection(ENABLE);
			FLASH_Lock();
		}
	}

注意：

- 启动读保护后，就不能读写程序了，如使用JLink读取程序，或者是重新下载程序。
- 所以，在下载程序之前，需要通过程序内部调用关闭读保护，关闭读保护之后，会自动清空Flash
- 另外，当第一次调用Set_Protect()函数启动读保护之后，不能再次调用Off_Protect()函数关闭读保护,需要重新断电才能关闭读保护

关闭读保护，在串口接收某个有效数据或按下某个按键时，调用关闭读保护：

	void Off_Protect(void)
	{
		if(FLASH_GetReadOutProtectionStatus() != RESET)
		{
			FLASH_Unlock();
			FLASH_ReadOutProtection(DISABLE);
			FLASH_Lock();
		}
	}

程序可以这样实现：
	
	int main(void)
	{
		/*用户代码*/
		if(KEY == 0)		//按键按下
		{
			Off_Protect();
		}
		else 
		{
			Set_Protect();
		}
		/*用户代码*/
		while(1)
		{
		/*用户代码*/
		}
	
	}

---

JLink_Windows_V614b软件下载链接：[JLink_Windows_V614b.exe](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/JLink_Windows_V614b.exe)

---

历史精选文章：

- [Jlink使用技巧之J-Scope虚拟示波器功能](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247483680&idx=1&sn=882e829f182219eb9293d9e010567748&chksm=fadfa74ecda82e58c1455db594d23d3cc121dfe019099cff3f7f297d4cb2459493d940e4b45c#rd)

- [Jlink使用技巧之单独下载HEX文件到单片机](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247483804&idx=1&sn=46ed9785a48d421325fa2f4b03a6c11c&chksm=fadfa7f2cda82ee4205020653b469ef9959d29a3a3a0c3922c8870bd17b877f8e80ac0918961&token=703787322&lang=zh_CN#rd)

- [百度智能手环方案开源(含源码，原理图，APP，通信协议等)](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247483787&idx=1&sn=a4d478dd46dfcf94a0c8f1f369062df8&chksm=fadfa7e5cda82ef3c9320aeba5d3e6e6d60dc32d80c570f5412198ec289bfec9e50a4814c8cf&token=1936031160&lang=zh_CN#rd)

- [如何在Keil-MDK开发环境生成Bin格式文件](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247483671&idx=1&sn=20422bf86fd8b58b58be47f2bae8819a&chksm=fadfa779cda82e6f9747c00d2f2ac763eb503f8d46b768c89a5c53a8bda6eb255deded727823&token=855879741&lang=zh_CN#rd)

- [elf格式转换为hex格式文件的两种方法](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247483759&idx=1&sn=eb7ee69807d7c0091f95bc4a98f1ce71&chksm=fadfa701cda82e1748e5005f3726df66027170f82ea8cdfcae9e5f3207ced11d6ab02735a536#rd)

- [两个HC-05蓝牙模块互相绑定构成无线串口模块](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247483709&idx=1&sn=2d5ab85d2cd48ee139d1af056a7019b6&chksm=fadfa753cda82e455883f0958515a139fcf7eb14b8e3da02bb30bf04d260ad6728ad3300c039#rd)

----

欢迎大家关注[我的个人博客](http://www.wangchaochao.top/)

或微信扫码关注我的公众号
