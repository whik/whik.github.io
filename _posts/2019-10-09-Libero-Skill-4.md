---
layout:     post
title:    Microsemi Libero使用技巧——使用命令行模式下载程序
subtitle:	Libero使用技巧
date:       2019-10-09 21:22:40 +0800
author:     Wang Chao
header-img: img/LiberoLogo.jpg
catalog:    true
tag:
    - Libero
    - FPGA
---

### 前言

在工程代码编译完成之后，如果需要给某个芯片下载程序时，或者是工厂量产烧录程序时，我们不需要把整个工程文件给别人，而只需要把生成的下载文件给别人，然后使用FlashPro就可以单独下载程序文件了。上一篇文章介绍了如何使用图形化界面——FlashPro软件，来进行pdb文件的下载，本文介绍如何通过命令行脚本来调用FlashPro软件进行程序的下载。

### 关于FlashPro

关于FlashPro下载器及FlashPro软件的介绍，可以查看上一篇文章：[Microsemi Libero使用技巧——使用FlashPro单独下载程序](http://www.wangchaochao.top/2019/10/01/Libero-Skil-3/)。

### 关于FlashPro执行TCL脚本文件

在[FlashPro 用户手册](http://coredocs.s3.amazonaws.com/Libero/11_8_0/Tool/flashpro_ug.pdf)中有一个章节专门介绍了Batch Mode（批处理模式）来使用FlashPro，详细介绍了各种批处理命令，如`new project`、`set_programming_file`、`set_programming_action`、`run_selected_actions`、`close_project`等命令，使用这些命令等同于使用图形化界面来执行打开工程、加载程序文件、选择编程选项、运行等步骤，即通过命令行执行这些操作。

例如使用FlashPro执行一个TCL脚本文件的命令行指令：

`<location of Microsemi software>/bin/flashpro.exe script:batch.tc`

即：`[flashpro.exe的路径] script:name.tcl`

而这个TCL脚本文件的功能就可以根据需要来指定了，如只编程FPGA部分，只编程ARM部分，只擦除程序等等。所有的命令使用方法都在[FlashPro 用户手册](http://coredocs.s3.amazonaws.com/Libero/11_8_0/Tool/flashpro_ug.pdf)中有详细的介绍。而如果只想进行程序下载，应该如何来编写一个TCL文件呢？这个很简单，不需要我们编写，Microsemi Libero软件在生成下载文件时，已经为我们生成好了，在我们执行`Program Device`，对FPGA进行编程时，Libero软件就是在执行这个脚本文件。这个脚本文件位于：`\LED_Blink\designer\impl1\led_driver_fp\led_driver.tcl`，脚本文件已经生成好了，就是将我们工程编译生成的pdb文见烧写到FPGA内。具体内容如下：

    open_project -project {D:\FPGA_Study\Microsemi\Blog\LED_Blink\designer\impl1\led_driver_fp\led_driver.pro}
    set_programming_file -no_file
    set_device_type -type {A2F200M3F}
    set_device_package -package {208 PQFP}
    update_programming_file \
        -feature {prog_fpga:on} \
        -fdb_source {fdb} \
        -fdb_file {D:\FPGA_Study\Microsemi\Blog\LED_Blink\designer\impl1\led_driver.fdb} \
        -feature {prog_from:off} \
        -feature {prog_nvm:off} \
        -pdb_file {D:\FPGA_Study\Microsemi\Blog\LED_Blink\designer\impl1\led_driver_fp\led_driver.pdb}
    set_programming_action -action {PROGRAM}
    run_selected_actions
    save_project
    close_project

每个命令和参数的意义可以对照[FlashPro 用户手册](http://coredocs.s3.amazonaws.com/Libero/11_8_0/Tool/flashpro_ug.pdf)进行查看，不过从字面意思我们也可以大概了解整个TCL脚本执行的过程：

- 打开工程，指定模式为单芯片模式
- 设置芯片型号和封装
- 指定fdb文件和pdb文件绝对路径
- 设置编程选项为：全编程，包括FPGA和ARM
- 运行编程
- 保存关闭工程

### 使用命令行来烧写程序

#### 1.添加FlashPro.exe文件路径到系统环境变量

使用Everything搜索软件全局搜索`FlashPro.exe`文件，可以看到FlashPro.exe文件位于安装目录下的`F:\Microsemi\Designer\bin`文件夹下：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/fp_location.jpg)

添加到`系统变量->PATH->F:\Microsemi\Designer\bin`

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/path_fp_location.jpg)

#### 2.运行TCL脚本文件

在`\LED_Blink\designer\impl1\led_driver_fp`目录运行Git Bash工具，运行命令：

    flashpro script:led_driver.tcl

输出编程状态信息：

    Qt: Untested Windows version 6.2 detected!
    FlashPro
    Version: 11.8.2.4
    Release: Libero SoC v11.8 SP2
    
    Software Version: 11.8.2.4
    PDB file 'D:\FPGA_Study\Microsemi\Blog\LED_Blink\designer\impl1\led_driver_fp\led_driver.pdb' has been loaded successfully.
    DESIGN : led_driver;  CHECKSUM : 23A8;  PDB_VERSION : 1.9
    Driver : 3.0.0 build 1
    programmer '08152' : FlashPro4
    Opened 'D:\FPGA_Study\Microsemi\Blog\LED_Blink\designer\impl1\led_driver_fp\led_driver.pro'
    The 'open_project' command succeeded.
    The 'set_programming_file' command succeeded.
    The 'set_device_type' command succeeded.
    The 'set_device_package' command succeeded.
    Info:  Adding FPGA Array data from file D:\FPGA_Study\Microsemi\Blog\LED_Blink\designer\impl1\led_driver.fdb.
    PDB file 'D:\FPGA_Study\Microsemi\Blog\LED_Blink\designer\impl1\led_driver_fp\led_driver.pdb' has been loaded successfully.
    DESIGN : led_driver;  CHECKSUM : 23A8;  PDB_VERSION : 1.9
    The 'update_programming_file' command succeeded.
    The 'set_programming_action' command succeeded.
    programmer '08152' : Scan Chain...
    programmer '08152' : Scan Chain PASSED.
    programmer '08152' : Executing action PROGRAM
    programmer '08152' : EXPORT FSN[48] = 01538a207858
    programmer '08152' : Erase ...
    programmer '08152' : Completed erase
    programmer '08152' : EXPORT CHECKSUM[16] = 23a8
    programmer '08152' : Programming FPGA Array
    programmer '08152' : Verifying FPGA Array
    programmer '08152' :         Verifying FPGA Array -- pass
    programmer '08152' : Finished: Wed Oct 09 19:21:27 2019 (Elapsed time 00:00:34)
    programmer '08152' : Executing action PROGRAM PASSED.
    
                            o - o - o - o - o - o
    
    The 'run_selected_actions' command succeeded.
    Project saved.
    The 'save_project' command succeeded.
    Project closed.
    The 'close_project' command succeeded.
    The Execute Script command succeeded.

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/git_program_succeeded.jpg)

可以看到和在Libero中运行`Program Device`时，输出同样的编程信息：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/libero_program_succeeded.jpg)

稍等几十秒，就会看到程序执行成功的信息。

### 使用bat批处理文件简化命令行操作

在工程目录`\LED_Blink\designer\impl1\led_driver_fp`下新建`batch_mode.bat`文本文件，文件方式编辑，输入以下内容：

	flashpro script:led_driver.tcl
	echo 烧写完成
	pause

如果想要下载程序，只需要直接双击这个批处理文件，稍等几十秒，然后看到"烧写完成"，就说明烧写成功了。

由于windows自带的cmd命令行没有后台运行和交互的功能，所以这种方式不会显示下载的状态信息，我手动添加了"烧写完成"的提示：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/bat_mode_program.jpg)

关于bat批处理文件的更多实用技巧可以参考：[BAT批处理基本命令总结](http://www.wangchaochao.top/2018/11/07/BatCmd/)

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

### 总结

本篇文章主要介绍一个命令`[flashpro.exe的路径] script:name.tcl`，通过使用此命令，可以实现不用打开FlashPro软件，而直接进行程序烧写，其实调用的还是FlashPro软件，只不过没有了图形化界面。使用这种方法可以简化操作的过程，但是不能显示下载的进度，如擦除、编程、完成等状态，如果熟练掌握了FlashPro支持的TCL命令，可以根据需要写出很实用的脚本文件。

### 资料下载

- [FlashPro官方介绍](https://www.microsemi.com/product-directory/programming/4977-flashpro#overview)
- [FlashPro 用户手册](http://coredocs.s3.amazonaws.com/Libero/11_8_0/Tool/flashpro_ug.pdf)
- [FlashPro软件和硬件安装指南](https://www.microsemi.com/document-portal/doc_download/130807-flashpro-software-and-hardware-installation-guide)
- [FlashPro v11.9安装包](http://download-soc.microsemi.com/FPGA/v11.9/Program_Debug_v11.9_win.exe)

### 推荐阅读

- [Microsemi Libero使用技巧——使用FlashPro单独下载程序](http://www.wangchaochao.top/2019/10/01/Libero-Skil-3/)
- [Microsemi Libero使用技巧——使用第三方编辑器Notepad++](http://www.wangchaochao.top/2019/09/30/Libero-Skil-2/)
- [Microsemi Libero使用技巧——查看芯片资源占用情况](http://www.wangchaochao.top/2019/09/30/Libero-Skil-1/)
- [Microsemi Libero系列教程（一）——Libero开发环境介绍、下载、安装与注册](http://www.wangchaochao.top/2019/05/23/Libero-1/)
- [Microsemi Libero系列教程（二）——新建点灯工程](http://www.wangchaochao.top/2019/09/29/Libero-2/)
- [详解串行通信协议及其FPGA实现](http://www.wangchaochao.top/2019/08/23/UART-Simple/)

----

- 我的个人博客：www.wangchaochao.top
- 我的公众号：mcu149

