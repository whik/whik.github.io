---
layout:     post
title:    兆易创新首款RISC-V开发板——GD32VF103-EVAL开箱评测
subtitle:	兆易开发板使用
date:       2019-12-29 12:22:40 +0800
author:     Wang Chao
header-img: img/RISC-V.png
catalog:    true
tag:
    - GD32VF103
    - RISC-V
---

### 开箱Vlog

<iframe src="//player.bilibili.com/player.html?aid=81164593&cid=138907086&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>
Hello，各位朋友大家好！今天我们来开箱兆易半导体的一款**RISC-V开发板——GD32VF103V-EVAL**。今年可以说是RISC-V比较火的一年，关注RISC-V的朋友可能都知道，2019年8月份的时候，兆易创新发布了国内第一款基于RISC-V内核的32位通用MCU——**GD32VF103系列**，而我今天拿到的这块板子就是基于GD32VF103的一块EVAL板，也就是评估板，相比于另一块Mini板，这块板子的外设资源比较丰富，我们一起来看看吧！

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/GD32VF103/DSC_2402-01.jpeg)

### GD32VF103V-EVAL评估板简介

GD32VF103V-EVAL全功能评估板使用 GD32VF103VBT6 作为主控制器。该评估板为采用 RISC-V内核的 GD32VF103 芯片提供了一个完整的开发平台，支持全方位的外围设备。评估板可以使用mini-USB 接口或5v DC适配器供电。提供包括扩展引脚在内的以及 SWD、Reset、Boot、User button key、LED、CAN、I2C、I2S、USART、RTC、SPI、ADC、DAC、EXMC、USBFS 等外设资源。提供从芯片手册，固件库到开发工具等全方位的文档和软件支持。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/GD32VF103/DSC_2439-01.jpeg)

### 板载资源简介

- 主控芯片，GD32VF103VBT6，RISC-V 内核，基于Nuclei Bumblebee处理器，主频108MHz。
- 板载GD_Link调试器，基于GD32F103CBT6，CMSIS-DAP固件
- 3.2寸 TFT液晶屏，240x320分辨率，电阻式触摸屏
- 1个五向按键，1个复位按键，4个用户LED
- 2路232串口，DB9接口，预留DAC/ADC接口
- 2路CAN接口，1路I2S音频接口
- 预留标准20P_JTAG接口，支持JLink调试器，调试速度要更快一些。
- 1片AT24C02 EPROM，1片GD25Q16 SPI Flash。
- RTC备用电池座，可能是因为快递原因，并没有安装纽扣电池。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/GD32VF103/DSC_2445-01.jpeg)

### 主控芯片

2019年8月22日，兆易创新GigaDevice宣布在行业内率先将开源指令集架构RISC-V引入通用微控制器领域，正式推出全球首个基于RISC-V内核的GD32V系列32位通用MCU产品，提供从谈芯片到程序代码库、开发套件、设计方案等完整工具链支持并持续打造RISC-V开发生态。

GD32VF103VBT6

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/GD32VF103/DSC_2431-01.jpeg) 

GD32VF103系列器件是基于Nuclei Bumblebee处理器的32位通用微控制器，其中Bumblebee
处理器是基于RSIC-V架构指令集开发而来。

- 最高128KB Flash，32KB SRAM，主频最高可达108MHz
- 2.6-3.3v供电范围，IO口可承受5v电平
- 2个12位 ADC/DAC
- 4个通用定时器，
- 3路SPI，2路IIC，2个IIS，2个CAN
- 3个USART，2个UART
- 提供QFN36，LQFP48/64/100封装。

系统架构框图

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/GD32VF103/GD32VF103_block.jpg)

### 生态环境与文档支持

 RISC-V的发展离不开完整的生态环境，IP核、芯片、IDE、开发板、文档资料、社区论坛等等。GD32VF103作为国内第一款RISC-V通用MCU产品，配套资料还是非常详细的。 以下是部分资料的下载地址，大家按需下载。

技术交流：

- GD32官方论坛： http://gd32mcu.21ic.com/ 
- RISC-V  MCU中文社区： https://www.riscv-mcu.com

资料下载：

- Bumblebee内核文档下载：https://github.com/nucleisys/Bumblebee_Core_Doc
- GD32VF103集成开发环境：[NucleiStudio_IDE_201909.rar](https://www.nucleisys.com/upload/file/2019/10/Nucleistudio/NucleiStudio_IDE_201909.rar)

包括Eclipse开发环境，JAVA运行环境JDK，RISC-V工具链，Openocd调试工具，串口调试助手，调试器驱动，SVD文件。

- GD32VF103参考手册英文：[GD32VF103_User_Manual_EN_V1.2.pdf](http://gd32mcu.21ic.com/data/documents/shujushouce/GD32VF103_User_Manual_EN_V1.2.pdf)
- GD32VF103参考手册中文：[GD32VF103_User_Manual_CN_V1.2.pdf](http://gd32mcu.21ic.com/data/documents/shujushouce/GD32VF103_User_Manual_CN_V1.2.pdf)
- GD32VF103数据手册：[GD32VF103_Datasheet_EN.pdf](http://gd32mcu.21ic.com/data/documents/shujushouce/GD32VF103_Datasheet_Rev%201.1.pdf)
- GD32VF103固件库用户指南英文：[GD32VF103_Firmware_Library_User_Guide_V1.0.pdf](http://gd32mcu.21ic.com/data/documents/yingyongbiji/GD32VF103_Firmware_Library_User_Guide_V1.0.pdf)
- GD32VF103固件库用户指南中文：[GD32VF103_Firmware_Library_User_Guide_CN_V1.0.pdf](http://gd32mcu.21ic.com/data/documents/yingyongbiji/GD32VF103_gujiankuyonghuzhinan_V1.0.pdf)
- GD32VF103固件库：[GD32VF103 Firmware Library_V1.0.1.rar](http://gd32mcu.21ic.com/data/documents/yingyongruanjian/GD32VF103_Firmware_Library_V1.0.1%20(1).rar)

GD32VF103标准固件库，包括程序、数据结构和宏定义，覆盖所有集成外设的特征，并包括了全部相关驱动和示例程序。 

-  GD32VF103 Demo Suites：[GD32VF103_Demo_Suites_V1.0.2.rar](http://gd32mcu.21ic.com/data/documents/kaifaban/GD32VF103_Demo_Suites_V1.0.2.rar)

GD32VF103系列开发板套件资料，包括原理图、使用指南、示例工程等。包含GD32VF103C-START板，GD32VF103R-START板，GD32VF103T-START板和GD32VF103V-EVAL板资料。 

- 国产卡姆派乐集成开发环境：https://code.ihub.org.cn/projects/790/repository/riscv-ide

康派来 IDE没有试用期限制的版本，相比于公版IDE Eclipse，这个IDE界面更简介，用户上手更快，更实用，目前最新版本是v0.31 Beta测试版，还没有发布正式版本。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/GD32VF103/DSC_2430-01.jpeg)

### 开机Demo

上电，打开开关，可以看到开机的Demo演示，按下液晶屏上对应的LED图标，对应的LED点亮，LED图标变为红色，触摸其他区域LED全点亮。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/GD32VF103/DSC_2406-01.jpeg)

### 推荐阅读

- [织女星开发板使用RISC-V核驱动GPIO](http://www.wangchaochao.top/2019/12/22/Vega-development-board-uses-risc-v-core-to-drive-GPIO/)
- [全平台轻量开源verilog仿真工具iverilog+GTKWave使用教程](http://www.wangchaochao.top/2019/12/03/Introduction-of-open-source-simulation-tool/)
- [详解EMC测试国家标准GB/T 17626](http://www.wangchaochao.top/2019/11/24/Detailed-explanation-of-EMC-test-national-standards/)
- [电路板上的这些标志你都知道是什么含义吗？](http://www.wangchaochao.top/2019/11/17/Certification/)
- [详解串行通信协议及其FPGA实现](http://www.wangchaochao.top/2019/08/23/UART-Simple/)

---

- 个人博客：www.wangchaochao.top
- 我的公众号：mcu149

