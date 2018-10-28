---
layout:     post
title:     两个HC-05蓝牙模块互相绑定构成无线串口模块
subtitle:  蓝牙模块配置无线串口
date:       2018-10-28 11:30:40 +0800
author:     Wang Chao
header-img: img/bluetoothUART.jpg
catalog:    true
tag:
    - 单片机
    - 蓝牙
---


## 关于HC-05蓝牙模块

> - 蓝牙模块BT-HC05模块是一款高性能的蓝牙串口模块。
> - 可用于各种带蓝牙功能的电脑、蓝牙主机、手机、PDA、PSP等智能终端配对。
> - 宽波特率范围4800~1382400，并且模块
> - 兼容单片机系统。
> - 当主从模式两个蓝牙模块配对成功后，可以简单的，更改为无线的蓝牙，让您设备或者产品更高级，更时尚。 
> - 可以很容易的使用提供的[蓝牙手机软件](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/%E8%93%9D%E7%89%99WiFi%E8%B0%83%E8%AF%95%E6%89%8B%E6%9C%BAAPP_Android.rar)

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/%E8%93%9D%E7%89%99%E5%B0%81%E9%9D%A2.jpg)

## 工作模式

HC-05嵌入式蓝牙串口通讯模块具有两种工作模式：命令响应工作模式和自动连接工作模式，在自动连接工作模式下模块又可分为主（Master）、从（Slave）和回环（Loopback）三种工作角色。当模块处于自动连接工作模式时，将自动根据事先设定的方式连接的数据传输；当模块处于命令响应工作模式时能执行下述所有 AT 命令，用户可向模块发送各种 AT 指令，为模块设定控制参数或发布控制命令。通过控制模块外部引脚（PIO11）输入电平，可以实现模块工作状态的动态转换。
![HC_05蓝牙模块原理图](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/HC05_SCH.jpg)

## 获取蓝牙模块地址

1. HC-05蓝牙串口模块连接USB-TTL模块，RX/TX交叉连接
1. 长按蓝牙模块上的小按键
![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/%E5%B0%8F%E6%8C%89%E9%94%AE.jpg)
1. 上电，红灯慢闪，表示已经进入到AT模式，可以进行蓝牙参数的配置
1. 打开“蓝牙测试软件”，点击左上角搜索端口，搜索到串口号后，点击“获取模块信息”
1. 左侧消息窗口会显示如下信息：

		AT
		OK

		AT+VERSION?
		+VERSION:2.0-20100601
		OK

		AT+ADDR?
		+ADDR:98d3:32:7105fd
		OK

其中ADDR后面的`98d3:32:7105fd`，就是当前蓝牙模块的地址，同理可以得到另外一个模块的地址。

A模块地址：`98d3:32:10f0ea`

B模块地址：`98d3:32:7105fd`

## 两个蓝牙模块互相绑定

我们要把A模块设置为主机，B模块设置为从机，并把B的地址绑定到A模块上，上电时，A模块搜索到B模块时，发起主动连接，从而构成无线串口模块。

### 对A模块的设置：

1. 恢复默认设置`AT+ORGL`
1. 设置配对密码`AT+PSWD=1234`
1. A设置为主机模式`AT+ROLE=1`
1. A绑定B地址：`AT+BIND=98d3,32,7105fd` （要把蓝牙地址中的冒号“：”换成“，”）　　　　　　　　

### 对B模块的设置：

1. 恢复默认设置`AT+ORGL`
1. 设置配对密码`AT+PSWD=1234`
1. B设置为从机模式`AT+ROLE=0`

通过以上的设置，对两个模块重新上电，两个模块先是快闪，随后马上变慢闪，说明两个模块已经连接上了，可以通过两个串口调试助手来测试是否连接上。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/%E6%97%A0%E7%BA%BF%E4%B8%B2%E5%8F%A3%E8%BF%9E%E6%8E%A5%E7%A4%BA%E6%84%8F%E5%9B%BE.jpg)

如果需要修改进行通讯的波特率，参考"[HC05指令集](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/HC05%E6%8C%87%E4%BB%A4%E9%9B%86.pdf)"手册可以知道，需要使用命令`AT+UART=<Param>,<Param2>,<Param3>`

Param1：波特率（bits/s）
取值如下（十进制）：

	4800
	9600
	19200
	38400
	57600
	115200
	23400
	460800
	921600
	1382400

	Param2：停止位
	0——1 位
	1——2 位
	Param3：校验位
	0——None
	1——Odd
	2——Even
	默认设置：9600，0，0

模块默认波特率是`9600`，如果需要更改为`115200`，则命令为 `AT+UART=115200,0,0`


## HC蓝牙模块参考资料

> [HC-05蓝牙模块测试软件](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/HC-05AT%E6%B5%8B%E8%AF%95%E7%89%88.rar)
> 
> [HC05指令集](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/HC05%E6%8C%87%E4%BB%A4%E9%9B%86.pdf)
> 
> [HC蓝牙模块原理图](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/HC%E8%93%9D%E7%89%99%E6%A8%A1%E5%9D%97%E5%8E%9F%E7%90%86%E5%9B%BE.pdf)
> 
> [蓝牙WiFi调试手机APP_Android](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/%E8%93%9D%E7%89%99WiFi%E8%B0%83%E8%AF%95%E6%89%8B%E6%9C%BAAPP_Android.rar)

## SPP蓝牙模块参考资料

> [SPP-CA蓝牙模块AT指令集](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/SPP-CA%E8%93%9D%E7%89%99%E6%A8%A1%E5%9D%97AT%E6%8C%87%E4%BB%A4%E9%9B%86.pdf)

> [SPP-CA蓝牙模块技术手册](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/SPP-CA%E8%93%9D%E7%89%99%E6%A8%A1%E5%9D%97%E6%8A%80%E6%9C%AF%E6%89%8B%E5%86%8C.pdf)


## 欢迎互相交流学习。
