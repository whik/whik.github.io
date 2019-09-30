---
layout:     post
title:    Microsemi Libero使用技巧——查看芯片资源占用情况
subtitle:	Libero使用技巧
date:       2019-09-30 15:22:40 +0800
author:     Wang Chao
header-img: img/LiberoLogo.jpg
catalog:    true
tag:
    - Libero
    - FPGA
---

### 前言

与MCU不同，FPGA的资源主要包括：逻辑资源，IO资源，Flash大小，PLL资源，SoC硬核处理器资源等，其中逻辑资源和IO资源是我们主要关心的，本篇文章将介绍，如何通过Microsemi Libero IDE来查看工程的详细资源占用情况。

### A2F200M3F的资源

以Microsemi SmartFusion系列A2F200M3F-PQ208为例：

- 系统门数：200K
- D触发器数量：4608个
- RAM Block：8 * 4608 Bit
- 用户IO：66
- 差分IO：31
- PLL：1个，集成在MSS中
- SoC：ARM Cortex-M3硬核，256KB Flash，64KB SRAM，DMA、IIC、UART、TIMER、PLL
- 可编程模拟资源：2路ADC，2路DAC

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/A2F_DS_1.jpg)

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/A2F_DS_2.jpg)

更详细的资源配置，可以查看Datasheet手册：[A2F200M3F_Datasheet.pdf](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/A2F200.pdf)

### Libero中查看资源占用

以点灯工程为例：[LED_Blink](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero-2/LED_Blink.rar)，打开工程之后，点击左侧的Compile选项，等待编译完成，会在右侧的窗口输出编译报告，如led_driver_compile_log.rpt文件，里面有详细的资源占用情况：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/compile_rpt.jpg)

编译报告（部分）：

    Compile report:
    ===============
    
        Microcontroller Subsystem  Used:      0  Total:      1   (0.00%)
        Fabric                     Used:     87  Total:   4608   (1.89%)
        Fabric IO (W/ clocks)      Used:      3  Total:     66   (4.55%)
        Fabric Differential IO     Used:      0  Total:     31   (0.00%)
        Dedicated Analog IO        Used:      0  Total:     31   (0.00%)
        Dedicated MSS IO           Used:      0  Total:     23   (0.00%)
        GLOBAL (Chip+Quadrant)     Used:      1  Total:     15   (6.67%)
        MSS GLOBAL                 Used:      0  Total:      3   (0.00%)
        On-chip RC oscillator      Used:      0  Total:      1   (0.00%)
        Main Crystal oscillator    Used:      0  Total:      1   (0.00%)
        32 KHz Crystal oscillator  Used:      0  Total:      1   (0.00%)
        RAM/FIFO                   Used:      0  Total:      8   (0.00%)
        User JTAG                  Used:      0  Total:      1   (0.00%)
    
    I/O Function:
    
        Type                                  | w/o register  | w/ register  | w/ DDR register
        --------------------------------------|---------------|--------------|----------------
        Input I/O                             | 2             | 0            | 0
        Output I/O                            | 1             | 0            | 0
        Bidirectional I/O                     | 0             | 0            | 0
        Differential Input I/O Pairs          | 0             | 0            | 0
        Differential Output I/O Pairs         | 0             | 0            | 0


- ARM SoC硬核，共1个，使用0个
- D触发器，共4608个，使用87个，占用1.89%
- 用户IO，共66个，使用3个，占用4.55%
- 输入引脚2个，输出引脚1个

资源占用主要看D触发器的占用情况，只要不超过4608，整个工程就可以编译通过，如果超过最大值，工程会报错。

如果是已经编译完成的工程，编译报告文件存放在工程目录下：`\LED_Blink\designer\impl1\led_driver_compile_log.rpt`

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/rpt_folder.jpg)

### 推荐阅读

- [Microsemi Libero系列教程（一）——Libero开发环境介绍、下载、安装与注册](http://www.wangchaochao.top/2019/05/23/Libero-1/)
- [Microsemi Libero系列教程（二）——新建点灯工程](http://www.wangchaochao.top/2019/09/29/Libero-2/)
- [东芝开发板驱动OLED模块显示LOGO图片](http://www.wangchaochao.top/2019/09/15/TT-M3HQ-3/)
- [使用系统定时器SysTick实现精确延时微秒和毫秒函数](http://www.wangchaochao.top/2019/09/08/TT-M3HQ-1/)
- [东芝半导体最新ARM开发板——TT_M3HQ开箱评测](http://www.wangchaochao.top/2019/08/25/TT-M3HQ-0/)
- [详解串行通信协议及其FPGA实现](http://www.wangchaochao.top/2019/08/23/UART-Simple/)

----

- 我的个人博客：www.wangchaochao.top
- 我的公众号：mcu149