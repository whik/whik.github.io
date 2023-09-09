---
layout:     post
title: 强大的边界扫描（3）：常用边界扫描测试软件
subtitle: 常用边界扫描测试软件
date:       2023-09-03 11:57:00 +0800
author:     Wang Chao
header-img: img/jtag.jpg
catalog:    true
tag:
    - JTAG
---


本文介绍两款常用的边界扫描测试软件：**XJTAG和TopJTAG**，前者收费、功能强大，后者免费（和谐后），功能简洁。

如果只是要进行简单的边界扫描测试，使用后者即可，本文重点介绍后者，也就是TopJTAG的下载、安装和基本使用。

### 1. 功能强大的XJTAG

XJTAG是由剑桥大学的毕业生们设计开发的一整套系统，包括**JTAG调试器硬件和上位机软件**，功能强大，价格不菲。

官方网站：`www.xjtag.com/zh-hans/`

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230814/00.jpg)

以其中一款JTAG控制器XJLink2 为例，其特性如下：

- 支持最多4个TAP接口
- TCK最高可达166MHz
- JTAG信号电压可配置，1.1-3.3v之间0.1v步进
- 所有IO管脚都内置和电压测量和频率测量功能
- 开放的DLL API接口

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230814/01.jpg)

XJTAG目前在国内授权的代理商有两家：广州风标电子和烟台长韵电子，有需要的朋友可以联系这两家代理商。

### 2. 小巧简洁的TopJTAG

常用的边界扫描软件还有TopJTAG公司的开发的一款小软件TopJTAG Probe，可以基于常用的仿真器，如J-Link、USB-Blaster等，配合Top JTAG Probe软件来实现边界扫描测试，界面简洁，使用简单，比起XJTAG等专业的边界扫描软件，对于我们平时简单测试使用是足够了。

官方网站：`http://www.topjtag.com/`

TopJTAG目前共有两款工具：

- TopJTAG Probe：边界扫描测试软件，可实现IO的读取、控制、波形的显示、脉冲的计数等。
- TopJTAG Flash Programmer：可以对芯片外置的CFI Flash进行编程和读取。

### 3. TopJTAG安装

TopJTAG软件安装包下载（包含Probe和Flash两个工具）：

```c
https://www.topjtag.com/files/TopProbe-Setup-1.7.5.exe
```

以上文件已经包含了Probe和Flash两个工具，如果你只想安装Flash工具，可以下载这个安装包，TopJTAG Flash Programmer安装包下载：

```C
http://www.topjtag.com/files/TopFlash-Setup-1.3.3.exe
```

软件的使用期限是20天，如果要长期使用需要购买授权，售价约$100，也有一些供学习使用的和谐文件：

```c
https://www.eefocus.com/blog/media/download/id-280909
```

### 4. TopJTAG基本使用

这里以Xilinx Kintex-7 XC7K325T开发板，配合JLink V9调试器为例，演示TopJTAG Probe的基本使用。

首先按照下图所示，连接FPGA和JLink调试器硬件

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230814/03.jpg)

确保JLink在设备管理器能正确识别

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230814/02.jpg)

打开TopJTAG Probe软件，新建一个连接，选择调试器为JLink，TCK时钟选择最高12MHz，可以看到还是支持很多JTAG调试器的。

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230814/04.jpg)

如果JLink和FPGA连接正确，会弹出当前连接的芯片厂商和IDCODE。

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230814/05.jpg)

指定BSDL文件的路径，并进行验证。

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230814/06.jpg)

关于BSDL文件的获取方法，可以查看[上一篇文章](https://blog.csdn.net/whik1194/article/details/125984685)：

- [强大的JTAG边界扫描（2）：BSDL文件介绍](https://blog.csdn.net/whik1194/article/details/125984685)。

如果验证通过，会弹出如下芯片视图，可以看到每个管脚的状态。

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230814/07.jpg)

点击RUN，启动边界扫描，默认工作在SAMPLE模式，蓝色表示管脚当前为低电平，红色表示管脚当前为高电平，黑色表示电源管脚（VCC/GND）。

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230814/08.jpg)

至此，关于TopJTAG的安装和基本使用就介绍完了，下面的几篇文章我会以MCU STM32和FPGA XC7K325T为例，演示TopJTAG的详细使用，直观的认识边界扫描是如何运行的，边界扫描的几个应用场景。