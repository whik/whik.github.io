---
layout:     post
title:     Jlink使用技巧之烧写SPI Flash存储芯片
subtitle:	 Jlink系列教程
date:       2019-01-12 19:30:40 +0800
author:     Wang Chao
header-img: img/JLink.jpg
catalog:    true
tag:
    - Keil
    - STM32
    - Jlink
---

### 前言

大多数玩单片机的人都知道Jlink可以烧写Hex文件，作为ARM仿真调试器，但是知道能烧写SPI Flash的人应该不多，本篇文章将介绍如何使用JLink来烧写或者读取SPI Flash存储器，JLink软件包含的工具中，有一个是JFlashSPI工具，这就是一个烧写和读取SPI存储器的工具了。

### 准备

- 要烧写程序或读取程序的的Flash芯片：SPI协议的Flash都可以，如W25Q128。
- JFlashSPI软件工具：在Jlink系列软件的安装目录下
- JLink V9仿真器
- 要烧写的文件：如GBK字库文件，UNIGBK.BIN

### 硬件连接

Jlink内部集成了SPI协议，部分接口是作为SPI复用功能的，具体硬件连接，如下图所示：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/JLink-SPI-2.jpg)

对于20P的标准JTAG接口

|引脚编号|名称|输入输出|和SPI Flash的连接|
|----|-----|-----|
|5|DI	|输入|连接SPI Flash的MOSI引脚|
|7|nCS	|输出|连接SPI Flash的CS引脚|
|9|CLK	|输出|连接SPI Flash的CLK引脚|
|13|DO	|输出|连接SPI Flash的MISO引脚|

对于10P的JTAG接口

|引脚编号|名称|输入输出|和SPI Flash的连接|
|----|-----|-----|
|2|nCS	|输出|连接SPI Flash的CS引脚|
|4|CLK	|输出|连接SPI Flash的CLK引脚|
|6|DO	|输出|连接SPI Flash的MISO引脚|
|8|DI	|输入|连接SPI Flash的MOSI引脚|

这里要注意的一点，正版的Jlink仿真器1脚是输入引脚，是外部提供参考电平的，但由于现在大部分的JLink仿真器都是学习(dao)版的，1脚不是输入，而是3.3v的输出，所以可以直接用这个管教来给SPI Flash供电。

### 1.打开

有两个工具，一个是JFlashSPI.exe是图形化工具，一个JFlashSPI_CL.exe是命令行操作，这里重点介绍图形化工具JFlashSPI，打开Jlink软件的安装目录，双击打开JFlashSPI，界面和之前介绍的JFlash差不太多。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/JLink-SPI-1.jpg)

### 2.连接SPI Flash芯片

点击Target->Connect，如果连接成功的话，会在底部输出连接信息，会显示Flash芯片的型号，生产厂家，Flash ID等等信息。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/JLink-SPI-3.jpg)

就像我这个Flash芯片，丝印明明写的华邦Winbond W25Q128，这里读取的却是飞索Spansion S25FL128K，难道是盗版芯片？

### 3.打开程序文件

点击File->Open data file，打开要烧写的字库文件，支持多种格式的文件，由于是选择的Bin文件，没有起始地址，所以手动输入烧写的起始地址，这里填写0就可以了。关于烧写文件的格式说明，可以查看之前的一篇文章:[BIN、HEX、AXF、ELF文件格式有什么区别](http://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247483671&idx=2&sn=e59ee5d6ea3098937bed342cd1c773e0&chksm=fadfa779cda82e6f72b5fbc52d7e6aeda25abf061763bb38655e13611301cde2a5f75dd72dbd#rd)

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/JLink-SPI-4.jpg)

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/JLink-SPI-5.jpg)

### 4.下载

点击Target->Auto下载程序到Flash芯片内。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/JLink-SPI-6.jpg)

下载完成后，会在底部窗口显示下载成功的信息，可以看出烧写速度还是比较快的，170KB的字库文件，用时不到1秒钟。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/JLink-SPI-7.jpg)

### 5.程序文件的读取

和读写单片机程序一样，也是支持读取SPI Flash芯片程序的，为了尊重他人的劳动成果，这里的介绍仅供学习使用，不可用于商业破解目的。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/JLink-SPI-8.jpg)

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/JLink-SPI-9.jpg)

可以看出，和下载相比，读写速度明显要慢得多，因为是读取的整个16M的存储区，所以时间会相对长一些。

### 6.程序文件的保存

程序文件读取完成后，可选择将文件保存到本地目录，保存格式可根据需要选择。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/JLink-SPI-10.jpg)

### 7.命令行工具的使用

JFlashSPI_CL.exe是JFlashSPI的命令行工具，通过输入命令实现读写Flash，这里简单介绍一下烧写功能。

在终端运行：`./JFlashSPI_CL.exe`

可看到一些帮助信息，主要是指令的说明：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/JLink-SPI-11.jpg)

可以看到-connect连接，-open打开烧写文件，-auto烧写，如果是烧写，这3个命令就够了，首先把要烧写的文件复制到JFlashSPI_CL.exe同级目录，输入指令：

	./JFlashSPI_CL.exe -open UNIGBK.BIN 0 -connect -auto

可以看到，烧写成功

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/JLink-SPI-12.jpg)

为了方便快捷，我们可以将以上命令写成一个批处理命令，直接双击运行即可，

新建download.bat文件，并以记事本方式打开，输入以下内容

	JFlashSPI_CL.exe -open UNIGBK.BIN 0 -connect -auto
	echo 程序烧写完成！
	pause

然后将这个bat文件和要烧写的字库文件放到一个文件夹下。双击直接运行就可以直接烧写，是不是方便了许多呢？

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/JLink-SPI-13.jpg)

### 支持的芯片列表

支持的Flash芯片多达百种，如Atmel的AT25系列，国产兆易的GD25Q系列等等，基本上常见的SPI协议Flash芯片都支持，具体的芯片列表可查看SEGGER官方网址：[List of supported SPI flashes](https://www.segger.com/products/debug-probes/j-link/technology/cpus-and-devices/supported-spi-flashes/)

### 速度说明

对于不同型号的Flash芯片，Jlink烧写器最大的写入速度也不同，具体可参考：

|Flash device	|Programming speed1	|Flash device |	Programming speed1|
|----|----|----|-----|
|ISSI IS25LP128	|500 KB/s	|Micron N25Q128A|	270 KB/s|
|ISSI IS25LD040|	100 KB/s	|Micron M25P10	|160 KB/s|
|ISSI IS25LQ080|	340 KB/s |Micron M25PX16	|230 KB/s|
|ISSI IS25CD010|	100 KB/s	|Micron M45PE10	|230 KB/s|
|ISSI IS25CQ032|	190 KB/s	|Micron M25PE4	 |  215 KB/s|
|Macronix MX25L3235E|	285 KB/s|	Spansion S25FL128	|410 KB/s|
|Macronix MX66L1G45G	|430 KB/s	|Spansion S25FL116K|	265 KB/s|
|Macronix MX66L51235F	|315 KB/s	|Winbond W25Q128FV	|340 KB/s|

### 参考资料：

[J-Flash SPI](https://www.segger.com/products/debug-probes/j-link/tools/j-flash-spi/)

### JLink软件的下载

JLink_Windows_V614b软件下载链接：[JLink_Windows_V614b.exe](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/JLink_Windows_V614b.exe)

---

Jlink使用技巧系列文章：

- [Jlink使用技巧之烧写SPI Flash存储](http://www.wangchaochao.top/2019/01/05/Jlink-SPI-Flash/)

- [Jlink使用技巧之虚拟串口功能](http://www.wangchaochao.top/2019/01/09/Jlink-UART/)

- [Jlink使用技巧之读取STM32内部的程序](http://www.wangchaochao.top/2019/01/06/Jlink-ReadBack-Hex/)

- [Jlink使用技巧之J-Scope虚拟示波器功能](http://www.wangchaochao.top/2018/10/17/JScope/)

- [Jlink使用技巧之单独下载HEX文件到单片机](http://www.wangchaochao.top/2019/01/05/Jlink-Download-Hex/)

----

欢迎大家关注[我的个人博客](http://www.wangchaochao.top/)

或微信扫码关注我的公众号
