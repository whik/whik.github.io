---
layout:     post
title: 强大的JTAG边界扫描（1）：基本原理
subtitle: 基本原理
date:       2023-09-01 11:57:00 +0800
author:     Wang Chao
header-img: img/jtag.jpg
catalog:    true
tag:
    - JTAG
---


**我是怎么了解到边界扫描的呢？**

这就要从我淘到一块FPGA板卡的事情说起了。

前段时间我在某二手平台上淘了一块FPGA板子，它长这样：

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230827/DSC_3901.JPG)

板子的整体尺寸很小巧，和手掌差不多大，外设也很简单：

- 12v供电，带一个散热器
- FPGA芯片是Xilinx XC7K325T，FFG676封装，芯片等级2I，生产日期是2017年21周
- 4路LED
- 3路轻触按键，其中一路是Config
- 1路CAN接口（没有焊接CAN收发器和电平转换芯片）
- 1路USB串口，CP2102转换芯片
- 1颗Spansion 128Mb QSPI Flash S25F128
- 1颗有源差分时钟200MHz
- 标准2.54mm 14P下载接口

听卖家介绍说，这是之前挖矿盛行时，定制矿机中的一块HASH算力卡，主要功能是通过串口接收数据，FPGA计算出HASH值，再通过串口输出，由于工作频率较高，还外加了散热器，后来由于矿难，就把矿机中的板卡都处理掉了，遗憾的是没有留下任何软硬件资料。

好在价格比较便宜，只要150块，要知道仅一颗FPGA芯片的价格都不止150块。

板子买来之后，接上12v电源，板子正常点亮，JTAG口也是正常的，FPGA芯片也没有加密，可以下载调试，虽然没有DDR等大容量缓存，无法做一些复杂的运算，即使跑MicroBlaze也无法运行太大的程序，但是对于入门学习FPGA基本知识，比如LED按键驱动，串口，CAN总线，SPI接口，MicroBlaze SDK入门学习等等足够用了。

遗憾的是不知道芯片的管脚定义，最简单粗暴的方式是，使用热风枪先把FPGA芯片拆下来，然后通过万用表蜂鸣档来确定LED、串口等外设的管脚，这种方式风险极高，一旦拆下再装上，板子有很高的报废风险。

那么，有没有一种方式，在不破坏板子的情况下可以确定管脚定义呢？

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230827/DSC_3923.JPG)

经过一番搜索和问询，还真发现了一种方式，那就是**JTAG边界扫描**。

简单的理解，只要通过JTAG口就可以随意的**读取或改变**芯片的任意一个管脚状态。

比如要获取按键对应的管脚，只要用手按住和松开按键，然后通过边界扫描，查看FPGA哪个管脚的状态有变化即可确定；对于LED，虽然是输出方向，同样我们也可以把它当成输入，人为的通过跳线给定高或低电平，通过这种方式，串口管脚、CAN管脚，时钟管脚都可以一一确定。

下面，我将分几个部分，带领大家大致了解**JTAG边界扫描**，从JTAG边界扫描介绍、到上位机软硬件，再到基于MCU和FPGA的边界扫描实际应用。

### 1. 什么是边界扫描？

提到边界扫描，就不得不提JTAG，因为边界扫描是JTAG接口的功能之一。

**JTAG，是Joint Test Action Group的简称，即联合测试行为小组。**

JTAG，对于电子行业的工程师们来说再熟悉不过了，无论是搞单片机、ARM开发，还是FPGA、DSP开发，都离不开这个接口，它不仅可以进行程序下载，还能在线调试Debug，简简单单几根线就完成了如此强大的功能，大大的提高了开发效率。

但是，你知道吗？JTAG协议的设计初衷，**并不是用来下载程序的**。

JTAG中的'T'，是Test的缩写，没错！JTAG接口被设计之初，就是用来测试的！

上世纪90年代，集成电路、芯片设计产业开始迅速发展，同时，也面临着诸多问题：芯片管脚和晶圆之间的连接如何确定是正常的？芯片管脚之间是没有短路的？芯片被焊接到PCB板上之后，如何保证焊接是良好的，没有短路、开路？芯片外围的电路和与之互联的芯片是正常的呢？尤其是一些BGA封装的芯片，无法使用探针等方式来直接测量芯片的管脚。

面对这些问题，Philips、TI等半导体厂商在1985年成立了联合测试行动小组 ，即JTAG，用来解决这些问题。

尽管人们认为 IEEE 1149.1 标准实际上就是JTAG，不过该标准的官方称谓是“**标准测试访问端口与边界扫描架构 (Standard Test Access Port and Boundary-Scan Architecture)**”。它定义了利用边界扫描检测 PC 电路板的检测访问端口 (TAP) 等。

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230827/DSC_3943.JPG)

JTAG协议发展到现在，目前主要有三个典型应用：

- 程序下载。即目前最常用的一个功能，它可以把用户程序下载到芯片内部的Flash中。
- 程序调试。即实时监控程序的运动状态，并且可以通过加入断点的方式来实时调试程序。

- 边界扫描。即Boundary-scan，也就是JTAG设计的初衷，主要用于芯片本身和PCB电路板的硬件测试。

### 2. JTAG硬件接口

JTAG协议工作的基本逻辑全依赖内部的TAP控制器（Test Access Port），其实就是一个状态机，通过TMS信号来切换不同的状态。

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230827/03.jpg)

标准的JTAG接口最少需要4个引脚，即：TCK、TDI、TDO和TMS，在IEEE1149.1标准中，TRST信号是可选的。

下面是每个信号的定义和功能：

- **Test Clock Input (TCK) **
  TCK 为 TAP 的操作提供了一个独立的、基本的时钟信号，TAP 的所有操作都是通过这个时钟信号来驱动的。TCK 在 IEEE 1149.1 标准里是强制要求的。
- **Test Mode Selection Input (TMS) **
  TMS 信号用来控制 TAP 状态机的转换。通过 TMS 信号，可以控制 TAP 在不同的状态间相互转换。TMS 信号在 TCK 的上升沿有效。TMS 在 IEEE 1149.1 标准里是强制要求的。
- **Test Data Input (TDI) **
  TDI 是数据输入的接口。所有要输入到特定寄存器的数据都是通过 TDI 接口一位一位串行输入的（由 TCK 驱动）。TDI 在 IEEE 1149.1 标准里是强制要求的。
- **Test Data Output (TDO) **
  TDO 是数据输出的接口。所有要从特定的寄存器中输出的数据都是通过 TDO 接口一位一位串行输出的（由 TCK 驱动）。TDO 在 IEEE 1149.1 标准里是强制要求的。
- **Test Reset Input (TRST) **
  TRST可以用来对TAP Controller进行复位（初始化）。不过这个信号接口在IEEE 1149.1标准里是可选的，并不是强制要求的。因为通过 TMS 也可以对 TAP Controller 进行复位（初始化）。

以Jlink的JTAG接口为例，可以看到标准的4个JTAG管脚：

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230827/01.jpg)

以下是JTAG接口的使用示意：

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230827/02.jpg)

每个管脚都有一个边界扫描寄存器单元，在时钟的驱动下，每个管脚的信号在寄存器单元之间依次流动，从而实现每个管脚状态的控制和读取。

### 3. 边界扫描相关的软硬件

理论上只要支持JTAG协议的调试器、下载器，都可以用来进行边界扫描测试，不过可能需要开发相对应的上位机软件。

本文介绍常见的两款边界扫描测试方案。

- JLink + TopJTAG Probe

TopJTAG是一款非常简洁、实用的边界扫描测试软件，支持多种调试器，比如最常用的JLink、USB-Blaster等等。我会在后面的文章单独介绍这款软件配合Jlink来进行边界扫描测试。

- X-JTAG

一套非常专业的边界扫描方案，研发公司位于英国剑桥，包括调试器和上位机，功能极其强大，当然售价也不菲！广泛应用于航天、汽车、国防、医疗、通信等专业领域。

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230827/04.jpg)

### 4. 学习资料

一位国外小哥在YouTube发布的视频：EEVblog#449-什么是JTAG以及边界扫描，B站有人搬运了，地址是：

- https://www.bilibili.com/video/BV1TT4y1e7HU

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230827/00.jpg)

还有一个是《ARM JTAG调试原理》文档，很精简，只有22页，可以对JTAG协议有个初步了解。

- http://www.micetek.com.cn/technic/jtag.pdf

JTAG协议的官方文档JTAG_IEEE-Std-1149.1-2001：

- https://fiona.dmcs.pl/~rkielbik/nid/JTAG_IEEE-Std-1149.1-2001.pdf

虽然不是最新版本的，但是对于学习JTAG协议的参考来说足够了。

### 5. 总结

对了，开头介绍的那款板卡，我使用边界扫描获取到的管脚定义如下：

```vhdl
####################################################################
#  Copyright(C), 2010-2023, https://blog.csdn.net/whik1194
#  ModuleName : top.xdc 
#  Date       : 2023-03-04
#  Time       : 23:55:00
#  Author     : whik1194
#  Function   : Pin constraint
#  Version    : v1.0
#       Version | Modify
#       ----------------------------------
#        v1.0    first version
####################################################################

set_property PACKAGE_PIN AA10 [get_ports gclk_p]
set_property PACKAGE_PIN D9 [get_ports greset]
set_property PACKAGE_PIN D8 [get_ports key]
set_property PACKAGE_PIN G20 [get_ports led1]
set_property PACKAGE_PIN H19 [get_ports led2]
set_property PACKAGE_PIN E20 [get_ports led3]
set_property PACKAGE_PIN F19 [get_ports led4]

set_property PACKAGE_PIN F8 [get_ports uart_rxd]
set_property PACKAGE_PIN F9 [get_ports uart_txd]
set_property PACKAGE_PIN G14 [get_ports can_rx]
set_property PACKAGE_PIN H14 [get_ports can_tx]

set_property IOSTANDARD DIFF_SSTL12 [get_ports gclk_p]
set_property IOSTANDARD DIFF_SSTL12 [get_ports gclk_n]
set_property IOSTANDARD LVCMOS33 [get_ports greset]
set_property IOSTANDARD LVCMOS25 [get_ports led1]
set_property IOSTANDARD LVCMOS25 [get_ports led2]
set_property IOSTANDARD LVCMOS25 [get_ports led3]
set_property IOSTANDARD LVCMOS25 [get_ports led4]
set_property IOSTANDARD LVCMOS33 [get_ports key]
set_property IOSTANDARD LVCMOS33 [get_ports uart_rxd]
set_property IOSTANDARD LVCMOS33 [get_ports uart_txd]
set_property IOSTANDARD LVCMOS33 [get_ports can_rx]
set_property IOSTANDARD LVCMOS33 [get_ports can_tx]

#QSPI 

set_property BITSTREAM.GENERAL.COMPRESS TRUE [current_design]
set_property BITSTREAM.CONFIG.CONFIGRATE 50 [current_design]
set_property BITSTREAM.CONFIG.SPI_BUSWIDTH 4 [current_design]
```

如果哪位朋友也买了这款板子，欢迎互相交流学习！

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230827/DSC_3950.JPG)