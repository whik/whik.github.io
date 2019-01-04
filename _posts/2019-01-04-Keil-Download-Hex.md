---
layout:     post
title:     使用Keil下载单独的Hex文件到单片机内
subtitle:	 下载Hex文件
date:       2019-01-04 21:30:40 +0800
author:     Wang Chao
header-img: img/JLink.jpg
catalog:    true
tag:
    - Keil
    - STM32
---

### 前言

初学STM32时，是通过串口1把Hex文件下载进STM32的，需要一个串口模块，而且还要设置BOOT0和BOOT1电平，然后通过FlyMcu软件进行下载，这也是一种不错的方法，这里我要介绍的是使用JLink调试器和Keil MDK-ARM来下载Hex文件，无需源代码，只需要一个调试器。


### 所需要的工具和软件

- Hex文件，如Demo_STM32.hex
- Keil软件，v4或v5版本，如Keil v5.16a
- ARM调试器，Jlink或ST-Link，如Jlink v9
- STM32开发板，如STM32F103RET6

### 1.准备一个完整的工程

准备一个完整的工程，注意，这个工程的芯片型号、开发板的芯片型号、Hex文件对应的芯片型号，这三者的芯片型号要保持一致，否则会出现不能正确运行的问题。如都是STM32F103RET6。

### 2.确定Jlink已经检测到芯片

如图，先选择调试器类型，然后点击 Setting，如果连接上芯片，会在右侧显示芯片的ID号。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/Keil-Download-Hex7.jpg)

### 3.确定这个工程的Hex文件的输出路径

打开工程配置界面中的，Output选项，可以看出我这个工程输出文件存放的路径是在OBJ目录下，名称是NiceDay

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/Keil-Download-hex1.jpg)

打开OBJ目录可以看到这个工程生成的hex文件名称为NiceDay.hex

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/Keil-Download-Hex2.jpg)

### 4.把要下载的Hex文件放到OBJ目录下

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/Keil-Download-Hex3.jpg)

### 5.把Output界面的NiceDay改为Demo_STM32.hex

注意末尾的扩展名.hex不要少。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/Keil-Download-Hex4.jpg)

### 6.不要编译工程，直接点击下载按钮。

在输出窗口可以看到下载完成

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/Keil-Download-Hex5.jpg)

如果程序没有运行，可以在下载界面查看是否勾选了下载完成后复位运行。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/Keil-Download-Hex6.jpg)

---

### 历史精选文章：

- [Jlink使用技巧之J-Scope虚拟示波器功能](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247483680&idx=1&sn=882e829f182219eb9293d9e010567748&chksm=fadfa74ecda82e58c1455db594d23d3cc121dfe019099cff3f7f297d4cb2459493d940e4b45c#rd)

- [BIN、HEX、AXF、ELF文件格式有什么区别](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247483671&idx=2&sn=e59ee5d6ea3098937bed342cd1c773e0&chksm=fadfa779cda82e6f72b5fbc52d7e6aeda25abf061763bb38655e13611301cde2a5f75dd72dbd#rd)

- [如何在Keil-MDK开发环境生成Bin格式文件](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247483671&idx=1&sn=20422bf86fd8b58b58be47f2bae8819a&chksm=fadfa779cda82e6f9747c00d2f2ac763eb503f8d46b768c89a5c53a8bda6eb255deded727823&token=855879741&lang=zh_CN#rd)

- [JSON简介](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247483763&idx=1&sn=b12aed2424a355a9aacc56c5f7ba9917&chksm=fadfa71dcda82e0b7f83b31b748630a578118f34529e60e13408691f0f249b1036e50d4a2aa2&token=1722697206&lang=zh_CN#rd)

---

欢迎大家关注[我的个人博客](http://www.wangchaochao.top/)

或微信扫码关注我的公众号

不定期更新电子嵌入式方面的个人学习笔记和技术总结，欢迎大家互相学习交流！











