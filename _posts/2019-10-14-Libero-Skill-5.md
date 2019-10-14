---
layout:     post
title:    Microsemi Libero使用技巧——使用FlashPro生成stp程序文件
subtitle:	Libero使用技巧
date:       2019-10-14 15:22:40 +0800
author:     Wang Chao
header-img: img/LiberoLogo.jpg
catalog:    true
tag:
    - Libero
    - FPGA
---

### 前言

在工程代码编译完成之后，如果需要给某个芯片下载程序时，或者是工厂量产烧录程序时，我们不需要把整个工程文件给别人，而只需要把生成的下载文件给别人，然后使用FlashPro就可以单独下载程序文件了。Microsemi FlashPro编程器支持stp/pdb两种文件格式，本文介绍如何通过FlashPro软件来将生成的pdb文件转换为stp文件。

### pdb文件的结构

pdb文件主要包含以下几部分内容：

- 安全配置，设置PASS密钥和AES密钥
- FPGA 文件，生成的FPGA阵列数据，为fpb格式
- FlashROM文件，为ufc格式
- eNVM文件，为efc格式

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/pdb_file_struct.jpg)

其中pdb文件内部包括FPGA编程阵列、ARM程序、安全配置等内容，并且可以通过FlashPro软件中的FlashPoint工具来再次修改这些内容，而stp文件是把pdb再次打包，而且不能再对其中的内容进行修改。

### 关于FlashPro

关于FlashPro下载器及FlashPro软件的介绍，可以查看上一篇文章：[Microsemi Libero使用技巧——使用FlashPro单独下载程序](http://www.wangchaochao.top/2019/10/01/Libero-Skil-3/)。

### 导出stp格式程序文件

打开工程的FlashPro工程。在`Program Device`右键，选择`Open Interactively`，打开FlashPro工程。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/program_right.jpg)

在打开的FlashPro中，选择`File->Export->Export Single Programming File`导出单程序文件。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/Export_Menu.jpg)

在弹出的窗口，勾选生成stp文件，输入文件名称，点击`Export`导出文件

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/sel_stp_file.jpg)

之后会在`\LED_Blink\designer\impl1\led_driver_fp`生成stp文件。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/gen_stp_file.jpg)

### stp文件的使用

stp文件和pdb文件一样，都是Microsemi FlashPro下载器支持的程序文件类型，在FlashPro软件界面点击`Configuration->Load Programming File`，加载pdb或stp程序文件。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/load.jpg)

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

### 资料下载

- [FlashPro官方介绍](https://www.microsemi.com/product-directory/programming/4977-flashpro#overview)
- [FlashPro 用户手册](http://coredocs.s3.amazonaws.com/Libero/11_8_0/Tool/flashpro_ug.pdf)
- [FlashPro软件和硬件安装指南](https://www.microsemi.com/document-portal/doc_download/130807-flashpro-software-and-hardware-installation-guide)
- [FlashPro v11.9安装包](http://download-soc.microsemi.com/FPGA/v11.9/Program_Debug_v11.9_win.exe)

### 推荐阅读

- [Microsemi Libero使用技巧——使用命令行模式下载程序](http://www.wangchaochao.top/2019/10/09/Libero-Skill-4/)
- [Microsemi Libero使用技巧——使用FlashPro单独下载程序](http://www.wangchaochao.top/2019/10/01/Libero-Skil-3/)
- [Microsemi Libero使用技巧——使用第三方编辑器Notepad++](http://www.wangchaochao.top/2019/09/30/Libero-Skil-2/)
- [Microsemi Libero使用技巧——查看芯片资源占用情况](http://www.wangchaochao.top/2019/09/30/Libero-Skil-1/)
- [Microsemi Libero系列教程（一）——Libero开发环境介绍、下载、安装与注册](http://www.wangchaochao.top/2019/05/23/Libero-1/)
- [Microsemi Libero系列教程（二）——新建点灯工程](http://www.wangchaochao.top/2019/09/29/Libero-2/)
- [详解串行通信协议及其FPGA实现](http://www.wangchaochao.top/2019/08/23/UART-Simple/)

----

- 我的个人博客：www.wangchaochao.top
- 我的公众号：mcu149

