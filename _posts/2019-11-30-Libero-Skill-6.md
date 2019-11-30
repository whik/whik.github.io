---
layout:     post
title:    Microsemi Libero使用技巧——FPGA全局网络的设置
subtitle:	Libero使用技巧
date:       2019-11-30 16:22:40 +0800
author:     Wang Chao
header-img: img/LiberoLogo.jpg
catalog:    true
tag:
    - Libero
    - FPGA
---

### 前言

刚开始做Microsemi FPGA+SoC开发时，会用到几个ARM专用的IP Core，功能一复杂起来，就会遇到某些信号如rst_n不能分配到指定的引脚上的情况，IO类型为CLKBUF，并不是普通的INBUF，而且，这些引脚既不是MSS_FIO，也不是属于Cortex-M3专用的GPIO，怎么会就不能分配呢？曾经一度怀疑是软件的BUG问题。最近在一个FPGA工程中也遇到了这个问题，搜索了一些资料，算是彻底明白了，记录一下。

### 问题描述

最近在一个FPGA工程中分配rst_n引脚时，发现rst_n引脚类型为CLKBUF，而不是常用的INBUF，在分配完引脚commit检查报错，提示需要连接到全局网络引脚上。

```

Running Global Checker...
Error:PLC002:No legal assignment exists for global net rst.n_c.
Info:Uhlocking the driver or removing the region constraint for net rst nc may help to satisfy 
Error:PLC005:Automat ic global net placement failed.

```

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/commit%E6%8A%A5%E9%94%99.jpg)

尝试忽略这个错误，直接进行编译，在布局布线时又报错。

```
Error: PLC002: No legal assignment exists for global net rst_n_c.
Error: PLC005: Automatic global net placement failed.
Error: Failure when executing Tcl script. [ Line 18 ]
```

尝试取消引脚锁定LOCK，再次commit检查成功，编译下载正常，但是功能不对，再次打开引脚分配界面，发现是rst_n对应的引脚并不是我设置的那个，看来是CLKBUF的原因。

### 问题分析

网络上搜索一些资料后，发现是在一些工程中会出现这个问题，如果rst_n信号连接了许多IP核，和很多自己写的模块，这样rst_n就需要很强的驱动能力，即扇出能力(Fan Out)，而且布线会很长，所以在分配管脚时，IDE自动添加了CLKBUF，来提供更大的驱动能力和更小的延时。那么什么是FPGA的全局时钟网络资源呢？

### FPGA全局布线资源简介

我们知道FPGA的资源主要由以下几部分组成：

- 可编程输入输出单元（IOB）
- 基本可编程逻辑单元（CLB）
- 数字时钟管理模块（DCM）
- 嵌入块式RAM（BRAM）
- 丰富的布线资源
- 内嵌专用硬件 模块。 

我们重点介绍布线资源，FPGA中布线的长度和工艺决定着信号在的驱动能力和传输速度。FPGA的布线资源可大概分为4类：

- 全局布线资源：芯片内部全局时钟和全局复位/置位的布线
- 长线资源：完成芯片Bank间的高速信号和第二全局时钟信号的布线
- 短线资源：完成基本逻辑单元之间的逻辑互连和布线
- 分布式布线资源：用于专有时钟、复位等控制信号线。 

一般设计中，我们不需要直接参与布线资源的分配，IDE中的布局布线器（Place and Route）可以根据输入逻辑网表的拓扑结构，和用户设定的约束条件来自动的选择布线资源。

其中全局布线资源具有**最强的驱动能力和最小的延时**，但是只能限制在全局管脚上，厂商会特殊设计这部分资源，如Xilinx FPGA中的全局时钟资源一般使用全铜层工艺实现，并设计了专门时钟缓冲和驱动结构，从而使全局时钟到达芯片内部的所有可配置逻辑单元（CLB）、I/O单元（IOB）和选择性块RAM（Block Select ROM）的时延和抖动都为最小。

一般全局布线资源都是针对输入信号来说的，如果IDE自动把rst_n引脚优化为了全局网络，而硬件电路设计上却把rst_n分配到了普通管脚上，那么就很麻烦了，要么牺牲全局网络的优势，手动将全局网络改为普通网络，要么为了利用全局网络的优势，修改电路，重新分配硬件引脚。所以如果一些关键的信号确定了，如时钟、复位等，产品迭代修改电路时，不要轻易调整这些关键引脚。

### Microsemi FPGA的全局布线资源

Microsemi FPGA的全局时钟管脚编号，我们可以通过官方Datasheet来找到，在手册中关于全局IO的命名规则上，有如下介绍：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/%E6%89%8B%E5%86%8C%E5%85%B3%E4%BA%8E%E5%85%A8%E5%B1%80%E6%97%B6%E9%92%9F%E5%BC%95%E8%84%9A%E7%9A%84%E5%91%BD%E5%90%8D.jpg)

即只有管脚名称为GFA0/1/2，GFB0/1/2，GFC0/1/2，GCA0/1/2，GCB0/1/2，GCC0/1/2（共18个）才支持全局网络分配，而且，如果使用了GFA0引脚作为全局输入引脚，那么GFA1和GFA2都不能再作为全局网络了，其他GFC等同理，这一点在设计电路时要特别注意。

对于Microsemi SmartFusion系列FPGA芯片A2F200M3F-PQ208来说，只有7个，分别是：GFA0-15、GFA1-14、GFA2-13、GCA0-145、GCA1-146、GCC2-151、GCA2-153，引脚分配如下图所示：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/A2F200%E5%85%A8%E5%B1%80%E5%BC%95%E8%84%9A.jpg)

所以在设计A2F200M3F-PQ208硬件电路时，时钟和复位信号尽量分配在这些管脚上，以获得硬件性能的最大效率。

这些全局引脚的延时时间都是非常小的，具体的时间参数可以从数据手册上获得。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/A2F200%E5%85%A8%E5%B1%80%E6%97%B6%E9%92%9F%E5%BB%B6%E6%97%B6.jpg)

### 全局网络改为普通输入

像文章开头介绍的情况，IDE自动把rst_n设置为全局网络，而实际硬件却不是全局引脚，应该怎么修改为普通输入呢？即CLKBUF改为普通的INBUF？网络上zlg的教程中使用的是版本较低的Libero IDE 8.0，新版的Libero SoC改动非常大，文中介绍的修改sdc文件的方法已经不能使用了，这里提供新的修改方法——调用INBUF IP Core的方式。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/commit%E6%8A%A5%E9%94%99.jpg)

这里官方已经考虑到了，在官方提供的INBUF IP Core可以把CLKBUF改为INBUF。

在Catalog搜索框中输入：INBUF，可以看到这里也提供了LVDS信号专用的IP Core。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/INBUF_CORE.jpg)

拖动到SmartDesign中进行连接

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/rst_inbuf.jpg)

或者在源文件中直接例化的方式调用INBUF Core：

```
INBUF INBUF_0(
        // Inputs
        .PAD ( rst_n ),	
        // Outputs
        .Y   ( rst_n_Y ) 
        );
```

这两种方法都是一样的。添加完成之后，再进行管脚分配，可以看到rst_n已经是普通的INBUF类型了，可以进行普通管脚的分配，而且commit检查也是没有错误的。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/INBUF.jpg)

### 普通输入上全局网络

如果布局布线器没有把我们要的信号上全局网络，如本工程的CLK信号，IDE自动生成的是INBUF类型，我们想让他变成CLKBUF，即全局网络，来获取最大的驱动能力和最小的延时。那么应该怎么办呢？

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/CLKA_INBUF.jpg)

这里同样要使用到一个IP Core，和INBUF类似，这个IP Core的名称是CLKBUF，同样是在Catalog目录中搜索：CLKBUF，可以看到有CLKBUF开头的很多Core，这里同样也提供了LVDS信号专用的IP Core。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/CLKBUF_Core.jpg)

可以直接拖动Core到SmartDesign图形编辑窗口：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/CLKA_CLKBUF.jpg)

或者是在源文件中以直接例化的方式调用：

```
CLKBUF CLKBUF_0(
        // Inputs
        .PAD ( CLKA ),
        // Outputs
        .Y   ( CLKA_Y ) 
        );
```

这两种方式都是一样的，添加完成之后，再进行管脚分配，可以看到CLKA已经是全局网络了，只能分配在全局管脚上。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/Libero_Skil/CLKA_BUF.jpg)

### 总结

对于不同厂家的FPGA，让某个信号上全局网络的方法都不尽相同，如Xilinx的FPGA是通过BUFG Core来让信号上全局网络，而且还有带使能端的全局缓冲 BUFGCE ， BUFGMUX 的应用更为灵活，有2个输入，可通过选择端选择输出哪一个。所以，信号的全局缓冲设置要根据不同厂商Core的不同来使用。

### 推荐阅读

- [Microsemi Libero使用技巧——使用FlashPro生成stp程序文件](http://www.wangchaochao.top/2019/10/14/Libero-Skill-5/)
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

