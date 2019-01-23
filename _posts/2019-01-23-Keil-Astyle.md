---
layout:     post
title:    一键自动格式化你的代码
subtitle:	 Keil使用技巧
date:       2019-01-23 11:30:40 +0800
author:     Wang Chao
header-img: img/Keil.jpg
catalog:    true
tag:
    - Keil
    - STM32
    - C语言
---

### AStyle简介

AStyle，即Artistic Style，是一个可用于C, C++, C++/CLI, Objective‑C, C# 和Java编程语言格式化和美化的工具。我们在使用编辑器的缩进（TAB）功能时，由于不同编辑器的差别，有的插入的是制表符，有的是2个空格，有的是4个空格。这样如果别人用另一个编辑器来阅读程序时，可能会由于缩进的不同，导致阅读效果一团糟。为了解决这个问题，使用C++开发了一个插件，它可以自动重新缩进，并手动指定空格的数量，自动格式化源文件。它是可以通过命令行使用，也可以作为插件，在其他IDE中使用。

### 基本使用

下载完成后，解压，然后在环境变量PATH，添加AStyle.exe的路径。

基本命令行格式：

	astyle [参数] [文件路径]

如在我的电脑E盘下有一个文件main.c，现在是这样的，可以看出很不规范，多个语句写在同一行，没有合理缩进，运算符两边没有空格等等。
	
	#include "sys.h"
	#include "delay.h"
	#include "usart.h"
	#include "led.h"
	
	int main(void)
	{
	delay_init();       //延时函数初始化
	LED_Init();         //初始化与LED连接的硬件接口
	while(1)
	{
	LED0=0;LED1=1;
	delay_ms(300);  //延时300ms
	LED0= 1;LED1 =0;
	delay_ms(300);  //延时300ms
	}
	}

打开CMD命令窗口，输入以下命令：

	AStyle --style=ansi E:\main.c

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/AStyle-1.jpg)

回车执行命令，然后再打开main.c查看，变成了这样：
	
	#include "sys.h"
	#include "delay.h"
	#include "usart.h"
	#include "led.h"
	
	int main(void)
	{
	    delay_init();       //延时函数初始化
	    LED_Init();         //初始化与LED连接的硬件接口
	    while(1)
	    {
	        LED0 = 0;
	        LED1 = 1;
	        delay_ms(300);  //延时300ms
	        LED0 = 1;
	        LED1 = 0;
	        delay_ms(300);  //延时300ms
	    }
	}

是不是看着很舒服，合理缩进、美观、可读性高，是规范的代码风格，当然这只是AStyle一个很基础的功能，其实它支持很多参数，还可以对整个目录及子目录下的源文件进行格式化操作。

### Keil开发环境添加AStyle插件

很多IDE都有自动格式化代码功能，而单片机开发经常使用的Keil系列软件居然没有这个功能，这怎么能忍？还好Keil有自定义插件的功能，可以添加AStyle自动格式化的工具，来格式化我们不规范的代码。

#### 1.打开Keil软件

选择Tools->Customize Tools Menu，自定义外部工具菜单。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/AStyle-2.jpg)

#### 2.新建工具

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/AStyle-3.jpg)

点击新建按钮，输入工具名称：`Astyle Current File`，Command命令选项，指定AStyle.exe的路径，Argument选项输入以下参数，注意大小写，建议复制粘贴，不会出错。

	-pnUk1s4 --style=ansi !E 

这些命令参数的含义，在下面有详细介绍，其中`!E`表示当前文件，这个参数在Keil软件的使用手册里可以查到。点击OK保存。

#### 3.试试格式化效果

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/AStyle-5.jpg)

好了，现在来试一下一键自动格式化工具吧，无论你的代码写的有多乱，只要点击Tools->Astyle Current File工具，你就会发现代码一下子变得美观了许多，就像这样。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/AStyle-4.jpg)

#### 4.定义一个快捷键

为了更方便，我们还可以自定义一个快捷键，来执行这个命令。点击工具栏最右边的配置图标，切换到Shortcut Keys选项，选择Tools:Astyle Current File，点击Create Shortcut创建新的快捷键，在弹出的窗口按下你要设置的快捷键，然后保存退出就可以了，下次需要使用的时候，只要按下相对应的快捷键，就可以一键将当前文件格式化。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/AStyle-6.jpg)

**其实，我还是觉得鼠标操作更方便。**

### AStyle插件参数详解

AStyle插件的参数实在太多了，这里只介绍我们上面那个命令中用到的参数。

	-pnUk1s4 !E --style=ansi

命令参数详解：

|参数名|大小写|说明|
|:---------|:--------|:--------|
|p|小写|只在操作符两边加空格|
|P|大写|在操作符和括号两边都加空格|
|n|小写|不备份格式化之前的文件，后缀为.orig，默认备份|
|U|大写|移除括号两边不必要的空格|
|d|小写|只在括号外面插入空格|
|D|大写|只在括号里面插入空格|
|k1|命令|指针或引用运算符*/&/^号靠近类型名|
|k2|命令|指针或引用运算符*/&/^号在类型名和变量名中间|
|k3|命令|指针或引用运算符*/&/^号靠近变量名|
|s4|命令|TAB键替换为4个空格|
|xC80|命令|一行最大字符数，超过后会在运算符处换行|
|H|大写|在关键字'if','for', 'while'之后添加空格
|S|大写|switch 与case不同列,case缩进
|K|大写|缩进case下面的语句|
|F|大写|空行分隔无关块|
|x|小写|删除多余空行|
|--style=ansi|命令|指定程序风格，如kr/linu/gnu等等

更多、更详细的参数说明可以查看自带的帮助文档。

### BAT命令格式化目录下的源文件

下面这个bat命令可以格式化当前目录及子目录下的所有源文件。

新建bat文件，以记事本打开，输入以下命令：	

	for /R %%f in (*.c;*.h) do AStyle.exe --style=allman --indent=spaces=4 --pad-oper --pad-header --unpad-paren --suffix=none --align-pointer=name --lineend=windows --convert-tabs --verbose %%f
	pause

### 各种代码风格的比较

这里只介绍几种常见的代码风格，更多的代码风格参考帮助文档->Brace Style Options。

**allman风格**

	int Foo(bool isBar)
	{
	    if (isBar)
	    {
	        bar();
	        return 1;
	    }
	    else
	        return 0;
	}

**java风格** 
	
	int Foo(bool isBar) {
	    if (isBar) {
	        bar();
	        return 1;
	    } else
	        return 0;
	}

**kr 风格**
	
	int Foo(bool isBar)
	{
	    if (isBar) {
	        bar();
	        return 1;
	    } else
	        return 0;
	}
**gnu 风格**

	int Foo(bool isBar)
	{
	    if (isBar)
	        {
	            bar();
	            return 1;
	        }
	    else
	        return 0;
	}

**linux 风格**

	int Foo(bool isBar)
	{
	        if (isFoo) {
	                bar();
	                return 1;
	        } else
	                return 0;
	}

**google 风格**
	
	int Foo(bool isBar) {
	    if (isBar) {
	        bar();
	        return 1;
	    } else
	        return 0;
	}

### 参考资料

> 
- 官方网址：http://astyle.sourceforge.net/
- 帮助文档：http://astyle.sourceforge.net/astyle.html
- 版本介绍：http://astyle.sourceforge.net/install.html

### 插件的下载

[AStyle_3.1_windows.zip](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/AStyle_3.1_windows.zip)

----

Jlink使用技巧系列文章：

- [Jlink使用技巧之合并烧写文件](http://www.wangchaochao.top/2019/01/17/Jlink-merge/)
- [Jlink使用技巧之烧写SPI Flash存储](http://www.wangchaochao.top/2019/01/12/Jlink-SPI-Flash/)
- [Jlink使用技巧之虚拟串口功能](http://www.wangchaochao.top/2019/01/09/Jlink-UART/)
- [Jlink使用技巧之读取STM32内部的程序](http://www.wangchaochao.top/2019/01/06/Jlink-ReadBack-Hex/)
- [Jlink使用技巧之J-Scope虚拟示波器功能](http://www.wangchaochao.top/2018/10/17/JScope/)
- [Jlink使用技巧之单独下载HEX文件到单片机](http://www.wangchaochao.top/2019/01/05/Jlink-Download-Hex/)

----

欢迎大家关注我的[个人博客](http://www.wangchaochao.top)

或微信扫码关注我的公众号
