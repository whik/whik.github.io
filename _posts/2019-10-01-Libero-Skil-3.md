---
layout:     post
title:    Microsemi Libero使用技巧——使用FlashPro单独下载程序
subtitle:	Libero使用技巧
date:       2019-10-01 16:22:40 +0800
author:     Wang Chao
header-img: img/LiberoLogo.jpg
catalog:    true
tag:
    - Libero
    - FPGA
---

### 前言

在工程代码编译完成之后，如果需要给某个芯片下载程序时，或者是工厂量产烧录程序时，我们不需要把整个工程文件给别人，而只需要把生成的下载文件给别人，然后使用FlashPro就可以单独下载程序文件了。本文介绍如何从工程目录中提取下载文件，并使用FlashPro软件来单独下载程序。

### 关于FlashPro

Microsemi FlashPro编程系统是Microsemi的FlashPro软件和硬件编程器的组合。它们可以为[PolarFire](https://www.microsemi.com/product-directory/fpgas/3854-polarfire-fpgas)，[IGLOO2](https://www.microsemi.com/product-directory/fpgas/1688-igloo2)，[SmartFusion2](https://www.microsemi.com/product-directory/soc-fpgas/1692-smartfusion2)，[RTG4](https://www.microsemi.com/product-directory/rad-tolerant-fpgas/3576-rtg4)，[IGLOO®系列](https://www.microsemi.com/product-directory/fpgas/1689-igloo)和[ProASIC3系列](https://www.microsemi.com/product-directory/fpgas/1690-proasic3)（包括RT ProASIC3）以及[SmartFusion](https://www.microsemi.com/product-directory/soc-fpgas/1693-smartfusion)，[Fusion](https://www.microsemi.com/product-directory/fpgas/1691-fusion)，[ProASIC PLUS](https://www.microsemi.com/product-directory/fpgas/4876-proasicplus)和  [旧版和停产闪存FPGA中的](https://www.microsemi.com/product-directory/fpga-soc/1638-fpgas#discontinued)所有FPGA提供系统内编程（ISP）。

这里说的FlashPro有两种含义：1.下载器FlashPro 3/4/5，2.下载软件FlashPro。

- FlashPro5下载器

![](https://www.microsemi.com/images/soc/products/nav/hw/flashpro/flashpro5_opt.jpg)

- FlashPro 4下载器

![](https://www.microsemi.com/images/soc/products/hardware/flashpro4.jpg)

- FlashPro 软件

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/FlashPro_Windows.jpg)

### 程序文件的路径

工程目录下，默认会生成一个`designer`文件夹，打开`designer\impl1\led_driver_fp`，这个文件夹就是下载程序所需要的FlashPro文件，其中包含pdb程序文件和FlashPro的工程。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/prj_fp_folder.jpg)

也就是说，当需要下载程序时，我们只需要把这个文件夹拷贝给别人就行了，而不用拷贝整个庞大的工程文件夹。

### 使用FlashPro下载程序文件

### 1.打开FlashPro软件

在Microsemi软件列表中找到FlashPro并打开，或者在安装目录找到FlashPro程序：`F:\Microsemi\Designer\bin\flashpro.exe`。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/flashpro_exe.jpg)

### 2.打开FlashPro工程

选择`Open Project`，打开工程目录下的FlashPro工程文件：`D:\FPGA_Study\Microsemi\Blog\LED_Blink\designer\impl1\led_driver_fp\led_driver.pro`

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/open_prj.jpg)

如果已经连接了下载器，会显示下载器的信息，如果没显示，要查看下载器是否连接正确。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/FlashPro_Main.jpg)

### 3.加载程序文件

默认是没有加载pdb程序文件的，我们需要手动指定。点击`Configuration->Load Programming File`，加载pdb程序文件。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/load.jpg)

选择工程目录下的`D:\FPGA_Study\Microsemi\Blog\LED_Blink\designer\impl1\led_driver_fp\led_driver.pdb`文件

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/load_pdb.jpg)

如果加载成功，会在底部显示加载成功的状态信息，并且在右下角会显示当前加载的pdb文件的路径。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/load_ok.jpg)

### 4.下载程序

连接FlashPro下载器和开发，点击`PROGRAM`按钮，就可以下载程序了，会显示下载进度，最后下载成功，会显示`RUN PASSED`

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/RUN_PASS.jpg)

如果显示红色`SCAN CHAIN FAILED`或者`RUN FAILED`，说明下载器和芯片的连接不正常，此时需要检查下载器是否正常，下载器和目标芯片的连接排线是否正常，目标芯片是否供电，USB连接等等。

### 编程方式配置

FlashPro默认是配置为`PROGRAM`编程方式，这也是最常用的一种编程方式，主要包括：验证芯片ID，擦除，编程，复位M3等操作，根据pdb文件大小，耗时几十秒，如果我们只想执行其中一项或者几项应该怎么操作呢？

选择`Configure Device`，进入编程配置选项，可以通过Action下拉来选择进行哪种编程方式。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/Configure_Device.jpg)

- DEVICE_INFO：获取芯片的信息
- ERASE：擦除FPGA程序和ARM程序
- READ_IDCODE：读取芯片的ID号
- RESET_CORTEXM3：复位ARM Cortex-M3硬核

这里有一个经常使用的配置模式，是`PROGRAM eNVM`选项，这个选项是只给内部的ARM Flash区进行编程，如果我们的工程中用到了M3硬核，这里会显示这个选项。使用这种编程方式的好处就是，如果FPGA配置没有更改，而只修改很小部分的ARM程序，那么就没有必要使用`PROGRAM`选项，编程时间会很长，如果只使用`PROGRAM eNVM`编程配置，则速度会非常快。

### 独立的FlashPro软件安装

FlashPro软件默认是和Libero一起安装，如果想单独安装FlashPro软件，而不想安装体积庞大的Libero也是可以的，不需要注册就可以使用。

官方下载地址：[FlashPro v11.9安装包](http://download-soc.microsemi.com/FPGA/v11.9/Program_Debug_v11.9_win.exe)

官网信息：

> Starting with Libero SoC v12.0, The FlashPro programming software will no longer be included in the Libero design software nor will it be available in stand-alone mode. Microsemi will be supporting the FlashPro Express v12.0 programming software, which replaces the FlashPro programming software. The last versions of Libero that supports FlashPro are Libero SoC v11.9 and Libero SoC PolarFire v2.3.

中文翻译：

> 从Libero SoC v12.0开始，FlashPro编程软件将不再包含在Libero设计软件中，也不会以独立模式提供。Microsemi将支持FlashPro Express v12.0编程软件，该软件将替代FlashPro编程软件。支持FlashPro的Libero的最新版本是Libero SoC v11.9和Libero SoC PolarFire v2.3。

从以上官网的说明可以了解到，12.0版本之后的版本将不再包含FlashPro，而只包含Flash Pro Express，这个软件的功能和Flash Pro软件的功能几乎一样。

### 菊花链拓扑实现多芯片同时编程

通过JTAG协议的菊花链拓扑可以实现多芯片的同时编程，对于同一块PCB上有多块FPGA时，可以节省多个JTAG口所占用的PCB空间。对于嵌入式封闭环境中，有时需要对系统中的FPGA程序进行在线或远程升级，必须将JTAG口引到机箱外，显然这种单JTAG口的菊花链结构是最佳选择。

传统的JTAG下载程序

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/jtag_normal.png)

JTAG菊花链拓扑实现多芯片编程

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/jtag_chain.png)

这种方式虽然好，但是对硬件设计也提出了要求：

- TMS和TCK的驱动能力

TMS和TCK由于连接了多个器件，所以要求驱动能力要足够，可以采用缓冲器的方式增强驱动能力，当器件较多，通过降低频率，即使不增加缓冲器，也可以提高信号的完整性。

- TDI到TDO的贯通延时

JTAG协议规定TCK下降沿输出TDI数据有效，并在TCK上升沿采集TDO数据，因此，在整个JTAG链中必须保证TDI至TDO的贯通延时（Propagation Delay）TCPD必须小于TCK的1/2周期TCLK/2，即△T=TCLK/2 –TCPD>0。也就是说，在增加缓冲驱动的情况下，JTAG链路中的芯片总数与每个芯片的TDO延时TDOV（FPGA为TTCKTDO）和TCK频率有关。在芯片总数确定以后，为保证△T>0，可以降低TCK的频率。

将处理器中的JTAG仿真接口连接成菊花链的方式，使用一个JTAG仿真控制器便能访问菊花链中任何一个处理器。如此，只需通过一个JTAG接口便能访问JTAG菊花链中的任何一个器件．但是由于菊花链的串联特性，如果任何一个设备从链路中移走，则链路便断裂开。

Microsemi FlashPro也支持菊花链拓扑方式来进行多芯片编程，但由于手上没有这样连接的硬件，所以没办法进行操作演示，具体的使用方法可以参考FlashPro使用手册：[FlashPro 用户手册](http://coredocs.s3.amazonaws.com/Libero/11_8_0/Tool/flashpro_ug.pdf)

### FlashPro下载器的其他功能

- 配合Synopsys Identity Debug实现在线调试
- 支持多种编程选项：擦除、编程、验证、复位M3、编程M3等
- 配合Microsemi SoftConsole实现ARM程序的调试和下载
- 支持导出或运行TCL脚本文件，或通过命令行下载程序
- 支持菊花链拓扑同时编程多个目标芯片
- 只更新ARM Cortex-M3的Hex程序
- 设置编程密钥和AES密钥，增强安全性
- 导出stp格式单程序文件

以上都是很实用的功能，具体的使用方法可以参考：[FlashPro 用户手册](http://coredocs.s3.amazonaws.com/Libero/11_8_0/Tool/flashpro_ug.pdf)，里面介绍的很详细。

本来只想写FlashPro下载程序的部分，没想到这一写就停不下来了。有错误的地方，欢迎指正。

### 资料下载

- [FlashPro官方介绍](https://www.microsemi.com/product-directory/programming/4977-flashpro#overview)
- [FlashPro 用户手册](http://coredocs.s3.amazonaws.com/Libero/11_8_0/Tool/flashpro_ug.pdf)
- [FlashPro软件和硬件安装指南](https://www.microsemi.com/document-portal/doc_download/130807-flashpro-software-and-hardware-installation-guide)
- [FlashPro v11.9安装包](http://download-soc.microsemi.com/FPGA/v11.9/Program_Debug_v11.9_win.exe)

### 推荐阅读

- [Microsemi Libero使用技巧——使用第三方编辑器Notepad++](http://www.wangchaochao.top/2019/09/30/Libero-Skil-2/)
- [Microsemi Libero使用技巧——查看芯片资源占用情况](http://www.wangchaochao.top/2019/09/30/Libero-Skil-1/)
- [Microsemi Libero系列教程（一）——Libero开发环境介绍、下载、安装与注册](http://www.wangchaochao.top/2019/05/23/Libero-1/)
- [Microsemi Libero系列教程（二）——新建点灯工程](http://www.wangchaochao.top/2019/09/29/Libero-2/)
- [详解串行通信协议及其FPGA实现](http://www.wangchaochao.top/2019/08/23/UART-Simple/)

----

- 我的个人博客：www.wangchaochao.top
- 我的公众号：mcu149

