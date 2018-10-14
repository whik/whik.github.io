---
layout:     post
title:     Keil开发环境如何生成BIN文件
subtitle:	 配置Keil开发环境生成Bin文件
date:       2018-10-14 21:30:40 +0800
author:     Wang Chao
header-img: img/Keil.jpg
catalog:    true
tag:
    - Keil
    - STM32
---

## 为什么需要BIN文件呢？

- 有些烧录器只支持BIN文件。
- 进行OTA远程升级时，只能使用BIN文件。
- 使用JLink脚本文件进行一键烧录时，只支持BIN文件。
- BIN文件要比HEX和AXF文件小的多。

但Keil默认生成的是AXF和HEX文件格式，那BIN怎么来生成呢？

## Keil配置生成BIN文件

Keil自带了一个小工具，可以通过执行指令来将AXF文件转换为BIN文件这就需要调用一个外部程序fromelf.exe来将AXF文件转换为BIN格式文件。

`fromelf.exe`文件的位置在安装目录 `Keil_v5\ARM\ARMCC\bin` 或者 ` Keil_v5\ARM\ARMCC_505u2\bin` 目录下。

![fromelf](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/20181014-KeilBin/FROM.jpg)

****在工程配置菜单中，User选项卡，编译后执行的命令，设置为 `
fromelf --bin -o "$L@L.bin" "#L"` ，当然也可以使用上面那种绝对路径的方式，需要看指定fromelf文件的路径，输出BIN文件的路径和生成的AXF文件的路径。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/20181014-KeilBin/2018-10-13_145516.jpg)


重新编译，可以看到在输出目录下已经生成了BIN文件。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/20181014-KeilBin/2018-10-13_145833.jpg)


## 其他开发环境如何将AXF文件转换为BIN文件？

当然如果你想把其他开发环境生成的AXF文件转换为BIN文件，也可以直接调用这个小工具来实现。



### 命令格式为：

	[fromelf.exe文件路径] --bin -o [BIN路径} [AXF文件路径}

如：

	E:/Keil_v5/ARM/ARMCC/bin/fromelf.exe --bin -o E:/Keil_Project/OneNET.bin E:/Keil_Project/OneNET.axf

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/20181014-KeilBin/2018-10-13_143419.jpg)