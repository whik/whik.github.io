---
layout:     post
title:    Microsemi Libero使用技巧——使用第三方编辑器Notepad++
subtitle:	Libero使用技巧
date:       2019-09-30 16:22:40 +0800
author:     Wang Chao
header-img: img/LiberoLogo.jpg
catalog:    true
tag:
    - Libero
    - FPGA
---

### 前言

与Xilinx的ISE和Altera的Quartus一样，Microsemi的编辑器也支持指定第三方编辑器。 Microsemi自带的编辑器，没有自动补全功能，也不支持中文注释，非常不好用，为了提高编码效率，我们可以指定第三方文本编辑器，如Notepad++、Sublime Text3、Vim、UltraEdit等，本文以Notepad++为例，其他编辑器操作方法一样，只需要修改程序路径即可。

### 修改文本编辑器

选择`Project->Preferences`下的Text editor，去掉默认选择的Libero自带编辑器，选择第三方编辑器，并指定可执行文件的路径。操作演示：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/TextEditor.gif)

### 实际效果

需要编辑文件时，直接双击，就可以在Notepad编辑器中打开，操作演示：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/OpenHDL.gif)

### 语法检查

如果使用Libero自带的编辑器，编辑完成之后，直接点击上面的对号，就可以直接进行语法检查，但是使用第三方编辑器，如何检查语法错误呢？

选择底部的`Design Hierarchy`，在.v源文件上右键，选择`Check HDL File`，就可以进行语法检查了，如果有错误，会在底部输出错误信息。 

操作演示：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/CheckHDL.gif)

### 推荐阅读

- [Microsemi Libero使用技巧——查看芯片资源占用情况](http://www.wangchaochao.top/2019/09/30/Libero-Skil-1/)
- [Microsemi Libero系列教程（一）——Libero开发环境介绍、下载、安装与注册](http://www.wangchaochao.top/2019/05/23/Libero-1/)
- [Microsemi Libero系列教程（二）——新建点灯工程](http://www.wangchaochao.top/2019/09/29/Libero-2/)
- [东芝开发板驱动OLED模块显示LOGO图片](http://www.wangchaochao.top/2019/09/15/TT-M3HQ-3/)
- [使用系统定时器SysTick实现精确延时微秒和毫秒函数](http://www.wangchaochao.top/2019/09/08/TT-M3HQ-1/)
- [东芝半导体最新ARM开发板——TT_M3HQ开箱评测](http://www.wangchaochao.top/2019/08/25/TT-M3HQ-0/)
- [详解串行通信协议及其FPGA实现](http://www.wangchaochao.top/2019/08/23/UART-Simple/)

----

- 我的个人博客：www.wangchaochao.top
- 我的公众号：mcu149