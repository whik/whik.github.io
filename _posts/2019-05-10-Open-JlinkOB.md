---
layout:     post
title:    手把手教你制作Jlink-OB调试器（含原理图、PCB、外壳、固件）
subtitle:	 开源项目
date:       2019-05-10 18:30:40 +0800
author:     Wang Chao
header-img: img/jlink_ob.jpg
catalog:    true
tag:
    - Jlink
    - ARM
    - STM32
    - 开源项目
---

### 前言

> 好久没更新博客和公众号了，感谢大家还没取关哈，好吧，我承认是我太懒了，**今天分享一个福利**！

趁着前段时间[嘉立创](https://www.sz-jlc.com/#)和[捷配](https://www.jiepei.com/)打价格战，一天之内，多次降价，看着真是热闹。捷配降到最低3元一款，而嘉立创降到最低5元一款，都是顺丰包邮，不过嘉立创免颜色费，而捷配不免，本着吃瓜群众的态度，赶紧薅了一把羊毛，做毕业设计时买的元器件还剩一些，就把之前练手画的一块JlinkOB小板投出去了，之前都是用的嘉立创，这次尝试一下捷配，关键是便宜！现在价格战已经结束了，刚才又去两家的官网看了一下，捷配又恢复了30元一款，而嘉立创还是保持5元。用的是网上开源的JlinkOB方案，主控STM32F103C8T6，下载Segger官方的JlinkOB固件，用了一段时间了，还算比较稳定。现在分享给大家，包含Altium版本的PCB文件、原理图文件、固件等，下载链接在文章末尾。

### 硬件电路

#### 原理图

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/Jlink_OB_DIY/Img/%E5%8E%9F%E7%90%86%E5%9B%BE.jpg)

原理图还是比较简单的，STM32最小系统+电阻电容，具体的原理，我还没看明白，USB接口连接到了PA11和PA12，STM32的这两个引脚可以用来模拟USB设备。另外，当时设计的时候，没有考虑到一些保护电路，如自恢复保险丝，所以实际使用时，要注意**不要接反了**！

#### PCB

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/Jlink_OB_DIY/Img/PCB.jpg)

从PCB布局布线来看，一般般，当时也是刚学习AltiumDesigner，没画过几块板，不过实际用起来完全没问题，速度轻松上50MHz，现在用了有一段时间了，还挺稳定。

### 焊接调试

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/Jlink_OB_DIY/Img/%E5%9B%BE%E7%89%873.jpg)

捷配的出货速度还算可以，可能是板子面积比较小，24小时就发出来了，下单的是5片，收到的时候居然有6片，这也可以理解，是为了方便拼版。

焊接了两块小板，焊接没什么难度，电阻电容大部分是0603封装，还比较好焊接。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/Jlink_OB_DIY/Img/%E5%9B%BE%E7%89%871.jpg)

如果某个阻值的电阻没有，如上下拉电阻和限流电阻，可以用相近的阻值来替换，不过个别电阻最好使用对应的值，如R5、R12，如果不一样，可以会导致USB识别失败。

确保电源没问题后，就可以下载固件了，使用另一个调试器，配合JFlash或者ST-LINK Utility烧录软件，SWD模式，把hex固件烧录进去，重新上电，就可以看到设备管理器里多了一个Jlink driver，打开Keil选择Jlink调试器，试一下看能不能用，第一次使用会提示升级固件，可以放心点击升级，这样就会把当前JlinkOB的固件升级到最新版本。SWD方式连接好ARM芯片，如STM32，可以看到成功检测到芯片，而且速度最大支持50MHz。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/Jlink_OB_DIY/Img/Driver.jpg)

这个板子的结构是按照淘宝卖的一个塑料外壳设计的，不过不用外壳也一样用。组装效果如图。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/Jlink_OB_DIY/Img/%E5%9B%BE%E7%89%872.jpg)

为了避免**广告嫌疑**，如果有需要塑料外壳的朋友，可以在后台回复，我会把淘宝链接发送给你。

### 待优化和改进的地方

- 优化布局和布线。
- 添加自恢复保险丝，防止短路。

另外网上还有一种开源的ST-Link和JlinkOB合并为一个的调试器项目，通过下载不同的固件可以作为JlinkOB或者ST-Link来用，而且还支持虚拟串口功能，有时间再做一个玩玩。

### 资料下载

- 工程打包下载：[Jlink_OB_DIY.rar](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/Jlink_OB_DIY.rar)
- 码云开源地址：`https://gitee.com/whik/Jlink_OB_DIY`

欢迎 `Fork`  & `Star`

### 历史精选

- [国产处理器的逆袭机会——RISC-V](http://www.wangchaochao.top/2019/04/27/ESBF/)
- [基于uFUN开发板和扩展板的联网校准时钟](http://www.wangchaochao.top/2019/04/08/uFun-Extend/)
- [Jlink使用技巧系列教程索引](http://www.wangchaochao.top/2019/01/17/Jlink-series/)
- [Qt实现软件自动更新的一种简单方法](http://www.wangchaochao.top/2019/03/31/Qt-Update/)
- [Verilog实现产生任意占空比的PWM波](http://www.wangchaochao.top/2019/04/17/FPGA-1/)

--------

欢迎关注我的[个人博客](http://www.wangchaochao.top)：`www.wangchaochao.top`

或微信扫码关注我的公众号
