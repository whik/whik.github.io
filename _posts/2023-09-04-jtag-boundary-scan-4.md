---
layout:     post
title: 强大的JTAG边界扫描（4）：STM32边界扫描应用
subtitle: STM32边界扫描应用
date:       2023-09-04 11:57:00 +0800
author:     Wang Chao
header-img: img/jtag.jpg
catalog:    true
tag:
    - JTAG
---


试想这样一个场景，我们新设计了一款集成了很多芯片的板卡，包括BGA封装的微控制器，如FPGA/MCU，还有LED、按键、串口、传感器、ADC等基本外设。

我们需要测试一下硬件电路工作是否正常、焊接是否良好，通常我们会写个测试代码，比如控制LED闪烁，读取按键的输入，串口收发一些数据，然后把程序烧录进去，看看现象是否和我们设计的一致。

当现象和设计不一致时，是代码设计的问题、还是硬件原理的问题、又或者是焊接的问题呢？应该如何一一排除呢？

这里就可以使用JTAG边界扫描的测试方法，来验证到底是哪里出的问题，因为JTAG边界扫描不需要写任何代码，只需要一个BSDL文件，就可以控制和读取芯片的任意管脚。

下面我们以意法半导体 MCU STM32F103为例，演示JTAG边界扫描的应用。

### 1. 获取芯片的BSDL文件

获取意法半导体MCU的BSDL文件，可以到官方网站搜索BSDL，就会弹出对应系列的BSDL文件包。

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230813/05.jpg)

STM32F1系列BSDL文件的下载地址：

```c
STM32F1:
https://www.st.com/content/ccc/resource/technical/ecad_models_and_symbols/bsdl_model/75/4a/50/d0/ad/aa/49/92/stm32f1_bsdl.zip/files/stm32f1_bsdl.zip/jcr:content/translations/en.stm32f1_bsdl.zip
```

下载到本地之后解压，可以看到很多BSDL文件，我们开发板上的芯片型号是STM32F103ZET6-LQFP144，属于大容量芯片，所以BSDL文件对应的是：

```
STM32F1_High_density_LQFP144.bsd
```

关于其他芯片的BSDL文件获取方式，可以参考之前的文章：[强大的JTAG边界扫描（2）：BSDL文件介绍](https://blog.csdn.net/whik1194/article/details/125984685)

### 2. 硬件连接

按照下图所示，使用排线连接JLink和开发板的JTAG接口。

![hw](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230909/hw.jpg)

并确保设备管理器里JLink驱动被正确识别。

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230909/08.jpg)

### 3. 边界扫描测试

关于TopJTAG边界扫描测试软件的介绍和基本使用，可以参考之前的文章：[强大的JTAG边界扫描（3）：常用边界扫描测试软件](https://blog.csdn.net/whik1194/article/details/125984788)

打开TopJTAG Probe软件之后，先创建一个工程，并选择JTAG设备类型，这里我们使用的是JLink。

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230909/00.jpg)

如果硬件连接正确，驱动安装正常，软件会自动识别到连接的芯片。

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230909/01.jpg)

指定芯片所对应的BSDL文件，这里我们选择上一步下载的`STM32F1_High_density_LQFP144.bsd`文件，并进行IDCODE校验。

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230909/02.jpg)

如果IDCODE不匹配，说明选择的BSDL文件错误，之后就进入到边界扫描测试界面了。

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230909/03.jpg)

点击Instruction按钮，可以选择三种测试命令：

- BYPASS：旁路掉当前器件，在菊花链拓扑方式时，跳过当前器件
- SAMPLE：采样模式，可以对所有管脚的状态进行读取，可以统计电平翻转的次数，或者以波形方式显示实时状态
- EXTEST：可以任意的控制所有外部管脚的状态，可手动指定为高低电平，高阻态。

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230909/05.jpg)

这里我们选择SAMPLE模式，点击RUN按钮，可以看到芯片所有的管脚实时状态，

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230909/04.jpg)

在Pins窗口，可以看到所有管脚的实时状态，选中一个管脚，可以把它添加到Watch窗口，或者Waveform窗口。

切换到EXTEST模式，可以手动设置管脚的高低电平或高阻状态。

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230909/06.jpg)

Watch窗口信号的还原能力，完全取决于JTAG_TCK的频率，即管脚信号的采样时钟。

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230909/07.jpg)

### 4. 总结

通过边界扫描可以快速的判断文章开头提到的几个问题，如果使用边界扫描的方式，发现读取和控制管脚的状态不对，那么可以判定是焊接的问题，通过编程，甚至可以按照一定的时序来控制管脚的状态，从而达到控制外部器件的目的。

总之，边界扫描是一种非常实用的测试方法，在电路板生产制造、芯片设计、芯片封测等方面都有很广泛的应用。
