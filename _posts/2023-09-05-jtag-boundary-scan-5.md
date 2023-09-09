---
layout:     post
title: 强大的JTAG边界扫描（5）：FPGA边界扫描应用
subtitle: FPGA边界扫描应用
date:       2023-09-01 11:57:00 +0800
author:     Wang Chao
header-img: img/jtag.jpg
catalog:    true
tag:
    - JTAG
---


[上一篇文章](https://blog.csdn.net/whik1194/article/details/125984596)，介绍了基于STM32F103的JTAG边界扫描应用，演示了TopJTAG Probe软件的应用，以及边界扫描的基本功能。本文介绍基于Xilinx FPGA的边界扫描应用，两者几乎是一样。

### 1. 获取芯片的BSDL文件

FPGA的BSDL文件获取方式，可以参考之前的文章：[BSDL文件获取](https://blog.csdn.net/whik1194/article/details/125984685)。

以Xilinx Kintex-7系列FPGA XC7K325T为例，可以在BSDL Library网站（www.bsdl.info ）获取，或者在ISE、Vivado的安装目录获取，

```c
D:\Program\Xilinx\14.7\ISE_DS\ISE\kintex7\data
D:\Program\Xilinx\Vivado\Vivado\2018.3\ids_lite\ISE\kintex7\data
```

### 2. 硬件连接

首先需要准备好以下硬件：

- JTAG调试器，如JLink V9标准版
- 一块FPGA板子，如Xilinx XC7K325T

Xilinx的JTAG接口和Jlink的JTAG接口线序不一致，需要使用单独的杜邦线分别连接TCK、TMS、TDI、TDO和VREF、GND信号。

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230908/11.jpg)

### 3. 边界扫描测试

打开TopJTAG新建工程，选择JTAG设备为JLink
![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230908/00.jpg)

如果连接正常，会显示当前连接芯片的IDCODE

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230908/01.jpg)

指定BSDL文件路径，并进行IDCODE校验。

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230908/02.jpg)

初始状态为stop状态，

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230908/03.jpg)

初始默认为Sample状态，点击RUN按钮，就可以看到所有管脚的实时状态，黑色的是电源管脚，黑色的是高电平，蓝色的是低电平。闪烁的说明当前为高低电平翻转状态。

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230908/04.jpg)

在左侧Pins窗口或右侧芯片视图，选择一个芯片管脚，右键，可以选择添加到Watch窗口或Waveform窗口

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230908/05.jpg)

Watch窗口可以看到管脚实时状态，并且可以统计电平翻转的次数，Waveform窗口可以显示实时的波形。

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230908/06.jpg)

Waveform支持放大、缩小、暂停等基本操作。

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230908/07.jpg)

Pins窗口，选择一个管脚右键之后，可以进行命名，输出高、低电平或高阻状态。

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230908/08.jpg)

支持多选之后，批量控制电平状态

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230908/09.jpg)

支持多选之后，批量添加到Waveform窗口

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230908/10.jpg)

### 4. 总结

和单片机不同，大多数FPGA芯片都是BGA封装的，管脚个数从200至1000不等，这也就意味着需要多层PCB来进行硬件设计，密集的引脚和PCB的内层走线，会导致故障的排查越来越困难，通过边界扫描，可以方便、快捷的判断出故障点，在产品研发、生产、测试阶段可以大大提高效率。
