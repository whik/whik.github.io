---
layout:     post
title:    基于uFUN开发板和扩展板的联网校准时钟
subtitle:	 uFUN开发板评测
date:       2019-04-08 21:30:40 +0800
author:     Wang Chao
header-img: img/uFunExtend.jpg
catalog:    true
tag:
    - STM32
    - uFUN开发板评测
---

### 项目概述

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/uFUN_Extend/%E6%97%B6%E9%92%9F%E6%98%BE%E7%A4%BA.jpg)

上周在uFUN试用群里看到管理员说试用活动快结束了，要抓紧完成评测总结，看大家的评测总结也都写了，我也不能落后啊！正好最近做的扩展板到手了，于是赶紧进行调试，做了一个不用校准的时钟，时钟这种小设计应该说是烂大街了吧！我一开始学习51的时候做了个可按键校准、带闹钟功能的时钟，学习STM32的时候做了个可以手机蓝牙APP校准的时钟，现在又用uFUN开发板做了个时钟，不过这个时钟是联网校准的。由于之前做过**桌面天气预报时钟**，如下图：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/WeatherClock/%E5%A4%A9%E6%B0%94%E9%A2%84%E6%8A%A5_%E7%95%8C%E9%9D%A22.jpg)

所以这个联网校准时钟的小项目实现起来还是很顺利的，底板是使用的uFUN开发板，扩展板是自己设计的，使用PCIe的接口和uFUN开发板进行通讯。使用ESP8266 WiFi模块获取北京标准时间，然后对STM32内部的RTC进行校准，再通过OLED显示出来。我知道，这个小项目无论是PCB的设计，还是控制程序的编写，对于论坛里一些有着多年经验的大佬来说，这个小项目简直不值一提，我只不过是班门弄斧罢了。有不对的地方，欢迎各位大佬拍砖！

### 扩展板资源简介

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/uFUN_Extend/%E6%89%A9%E5%B1%95%E6%9D%BF%E6%AD%A3%E9%9D%A2%E5%9B%BE.jpg)

- 0.96寸OLED屏，IIC接口，128*64像素，通过外部电路的修改，可支持8080并口，SPI接口。
- W25Q128，SPI接口，可以用来存储字库，配合uFUN开发板的SD卡功能，可以实现字库的更新
- AT24C02，EEPROM，IIC接口，掉电不丢失，可以用来保存一些用户数据
- SHT20，IIC接口温湿度传感器。
- 5个LED，1个电源指示，4个用户LED，这四个LED都连接到了定时器的通道，可通过PWM占空比控制亮度
- 1个光敏电阻，把光照强度转换为AD电压值，可以实现根据外界亮度控制OLED屏的亮度。
- 1个OLED模块接口，可外插IIC接口的OLED模块
- 蓝牙/WiFi模块兼容接口设计，可支持HC-05蓝牙模块，或者是原子的ESP8266模块。
- 1个DHT11温湿度传感器模块接口

### 扩展板原理图设计

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/uFUN_Extend/SCH.jpg)

- 电源电路

从uFUN的原理图可以看出，PCIe扩展口中有5v电源，数字地，模拟3.3v，模拟地，而没有引出数字3.3v电源，为了不干扰模拟3.3v，扩展板的3.3v电源来自5v经过AMS117稳压，AMS117-3.3是一款比较常用的LDO电源芯片，最大可提供1A的输出电流，对于扩展板来说是足够用了。

- OLED电路

OLED外围电路，参考中景园官方推荐的电路，使用的是IIC接口，因为之前没用过IIC总线挂载多个从机，为了避免调不通，所以OLED的IIC接口使用了独立的管教，同时上拉4.7K电阻。

- W25Q128电路

W25Q128是采用的SPI通讯协议，正好扩展接口中有SPI接口，所以就用了STM32的硬件SPI2接口，当然也可以软件模拟。

- AT24C02电路

实际上焊接的是ST公司的M24C64，目前读写还没调通，和底板的LIS3DH共用一组IIC总线。

- SHT20电路

正好上次做板子，买的SHT20还剩几片，所以在这个扩展板上，也使用了这款传感器，这款数字温湿度传感器体积非常小，只有**3\*3\*1.1mm**，和底板的LIS3DH共用一组IIC总线，后面调试正常，可以获取到温湿度数据。

- WiFi蓝牙兼容接口

偶然发现HC-05和原子的ESP8266模块接口几乎一样，所以这部分设计成了兼容两种模块的接口，如下图：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/uFUN_Extend/%E8%93%9D%E7%89%99WiFi%E5%85%BC%E5%AE%B9%E6%8E%A5%E5%8F%A3.jpg)

- 其他电路

4路LED都连接到了STM32的定时器输出通道，可以用来控制亮度，另外也充分利用了扩展口留出的模拟电源和模拟地，通过简单的分压原理，可以把光照强度转换为AD电压值，还增加了OLED模块接口和DHT11温湿度传感器接口。

### 扩展板PCB设计

- 结构的设计

SHT20官方推荐的放置方式是在板子边缘挖个岛出来，然后把传感器放进去，这样能最大程度的减少板子热量的传递，从而提高精度，但是，由于扩展板面积较小，再挖个岛出来，看着太难看了，就没有挖。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/uFUN_Extend/SHT-20.jpg)

板子的整体结构图来自于前一段时间，[@张进东](https://www.mianbaoban.cn/blog/uid-me-1595057.html) 张工在uFUN试用群里分享的AD版本的PCB图，那个图是这样的：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/uFUN_Extend/%E7%BB%93%E6%9E%84%E5%9B%BEAD.jpg)

我进行了稍微的修改，把安装孔右边多余的部分去掉了，而且为了和底板一致，我把扩展板改成了圆角，但是接口和安装孔的相对位置没动，板子的TOP面：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/uFUN_Extend/PCB-TOP.jpg)

- 封装的选择

电阻电容封装的选择上，一开始觉得板子空间比较小，怕放不下所有的元器件，几乎所有的电阻和电容都选择了0402封装，同时也出现了一个比较尴尬的事，就是有1种电容和1种电阻，嘉立创SMT的基础库里没有，扩展库里才有，而如果使用扩展库里的元件，需要加每种20元的换料费，所以最终打板多花了40元，这一点也是我没有考虑到的。等投板之后，我又尝试着把所有的电阻电容封装都换成0603的，发现也能放得下。

- 元件的布局布线

元件布局方面，为了防止元器件放在背面和底板的部分元件冲突，同时也为了满足嘉立创只能单面贴片的工艺要求，我把所有的元件都放在了TOP面，BOTTOM面只放了OLED裸屏的接口，为了方便连接外部的模块，把接口都放在了板子的边缘，LED也放在了板子边缘，由于SHT20是DFN封装，为了方便后面使用风枪进行焊接，周围留出了一点空间。

板子的BOTTOM面：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/uFUN_Extend/PCB-BOTTOM.jpg)

3D效果显示

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/uFUN_Extend/PCB-TOP-3D.jpg)

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/uFUN_Extend/PCB-BOTTOM-3D.jpg)

我用的是AD9版本，3D效果渲染的还是挺不错的。

### PCB的打样和调试

不知道今年是不是板材降价了，各大PCB板厂都在降价，去年做毕业设计的板子时还是50元5片10\*10cm以内，今年就是30元5片10\*10以内，而且连彩色油墨费都不收了。PCB打样和SMT贴片还是选择了之前用过的嘉立创，5片全贴，一共花了**191大洋**！生产工艺方面时，由于嘉立创只支持绿油进行SMT，所以只能用看着比较LOW的绿色油墨，其实我更喜欢蓝色油墨，而且和底板很搭！其他的工艺如过孔盖油、有铅喷锡，由于只是样板，就没有选择镀金工艺了，所以金手指成了锡手指，外加倒斜边，倒斜边是金手指板卡常用的一种工艺，是为了能够方便的插到插槽里面。还要注意非常非常重要的一点：PCB板的厚度，其实这个我一开始设计的时候就考虑过，使用之前做的1.6mm的板子试着往里面**插了一下**，发现根本插不进，网上也没搜到PCIe的板子应该做多厚，所以就选择了比1.6稍微薄一点的1.2mm厚度。详细的生产工艺：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/uFUN_Extend/%E7%94%9F%E4%BA%A7%E5%B7%A5%E8%89%BA-1.jpg)

PCB生产进度和SMT进度：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/uFUN_Extend/%E7%94%9F%E4%BA%A7%E8%BF%9B%E5%BA%A6.jpg)

由于板子比较简单，从下单到发货大概用了3天的时间吧，实际的样板图片：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/uFUN_Extend/PCB.jpg)

板子拿回来一插，发现还是稍微厚了点，不能完全插进PCIe座里面，而且底板的铜柱还不能对准扩展板的安装孔。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/uFUN_Extend/%E5%B0%BA%E5%AF%B8%E8%AF%B4%E6%98%8E.jpg)

后来请教了 [@张进东](https://www.mianbaoban.cn/blog/uid-me-1595057.html) 张工，原来只需要1.0mm就够了，斜着插进去往下一压，再用铜柱固定就行。不过我用暴力手段解决了一下，直接用剪刀剪掉了一段金手指，还好金手指上面的铜箔没有掉，这样才勉强能用一个螺丝固定好，安装效果如文章第一张图：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/uFUN_Extend/%E5%AF%B9%E6%AF%94%E5%9B%BE.jpg)

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/uFUN_Extend/%E8%BF%99%E4%B9%88%E7%A5%9E%E5%A5%87%E5%90%97.jpg)

就这么不完美的解决了，不过还是有几个管脚接触不良，如DHT11接口的那个引脚，稍微松动一点就不能用，不过已经板载了温湿度传感器，倒也影响不大。然后就试着点亮了OLED屏，先显示个uFUN的LOGO吧：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/uFUN_Extend/%E9%BB%84%E8%93%9D%E5%8F%8C%E8%89%B2%E5%B1%8F%E5%B9%9516_9.jpg)

到这里觉得应该没啥问题了吧，开始调试WiFi接口，使用的是引出的串口3来和WiFi模块进行通讯，由于之前的程序是用的串口2，所以又稍微改了一下，调了半天，还是不通，就把同样的程序下载到其他板子上，没问题啊！难道是接触不好，最后一查，是uFUN的原理图中PB10和PB11的网络标号也反了，导致扩展板的TX和RX反了，这可怎么办呢？

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/uFUN_Extend/UART3.jpg)

又舍不得浪费板子啊！所以把原来TX和RX连接的两根线割断，重新飞了两根线，完美解决！这线飞的可没有uFUN开发板上的那根飞线漂亮！

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/uFUN_Extend/%E9%A3%9E%E7%BA%BF.jpg)

另外，扩展板IIC总线上的24C02读写一直没调通，不知道是物料的问题还是读写时序的问题，而同是IIC总线上的SHT20温湿度传感器就没问题。另外由于本项目中没有用到过多的中文，所以SPI Flash也没用到。

### 控制程序的设计

可能是由于字库文件没有进行精简，所以整个程序占用了非常多的空间，总是报内存不足的错误，为了能跑起来，我把单片机型号改成了STM32F103RE系列，由于RE系列比RC系列的存储空间要大的多，并没有报错误，而且程序可以正常运行，虽然都是大容量产品，程序可以兼容，但这是一种**非常不推荐**的方式。

#### 实现思路

实现原理就是：STM32驱动ESP8266 GET北京时间的接口，得到一串JSON数据，然后STM32调用cJSON库进行解析，获取到时间信息，然后把这个时间信息写入到RTC寄存器，STM32内部RTC从当前时间开始运行，然后显示到OLED屏上，程序中设置的是每隔10分钟，获取一次时间。北京标准时间的来源使用了K780数据的API接口，地址：`http://api.k780.com:88/?app=life.time&appkey=10003&sign=b59bc3ef6191eb9f747dd4e83c99f2a4&format=json`，是JSON格式的，数据的解析使用了开源的JSON解析库cJSON，只有两个文件，使用起来还是比较简单的。

#### 主要功能：

- STM32 RTC的使用。
- OLED的驱动，支持6*8 ASCII码显示，8*16 ASCII码显示，16*32数字0-9和冒号:的显示，汉字“周一二三四五六日”的显示。
- ESP8266的驱动，和北京标准时间服务器接口建立TCP连接。
- cJSON库解析接收的JSON数据，获取北京时间，年月日，时分秒。

关于JSON格式的说明和cJSON库的使用，可以参考我之前写的两篇文章：

- [使用cJSON库解析JSON](http://www.wangchaochao.top/2018/12/04/Parse-JSON/)
- [JSON简介](http://www.wangchaochao.top/2018/11/18/cJSON/)

很多API接口的数据格式都是JSON格式的，如我之前做的桌面天气预报时钟，使用的是心知天气的数据源，就是JSON格式，网上也有很多免费的API接口，可以做出很多好玩的东西，就看你的想象力了！


#### 不足之处

- 断电不能走时

这算是这个程序最大的一个BUG吧，虽然uFUN开发板有超级电容备用电池接着，但一直没调通断电走时的功能，这种情况带来的一个问题就是，如果不插WiFi模块的情况下重新上电，是不能从上次保存的时间继续运行的。

- 程序不够精简

为了能让程序运行起来，把单片机改成大容量型号，应该从根本上精简字库，优化程序大小。

- GET请求偶尔会失败

偶尔会出现发送GET请求，接收不到数据的情况，可能是内存分配的问题。

### 总结

这个小项目总得来说，硬件调试方面比较坎坷，又是飞线，又是割板子的。而软件调试方面，由于之前做过类似的设计，所以还是比较顺利的。由于评测活动的时间关系，本次评测的大作业我只做了这个简单的联网校准时钟。其实，有了WiFi/蓝牙模块接口和OLED屏，就可以做很多好玩的东西了，比如选择一个开放的云平台，如中移OneNET等等，然后和云台进行一些交互，如采集温湿度上传到云平台做个温湿度远程监测系统，或者是远程控制LED或者蜂鸣器，桌面天气预报等。下面这几个界面是我去年在学校时，使用中移OneNET云平台做的一些界面：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/uFUN_Extend/OneNET-2.jpg)

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/uFUN_Extend/OneNET-3.jpg)

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/uFUN_Extend/OneNET-1.jpg)

对于uFUN开发板的整体评测过程来说，虽然配套的文档存在一些瑕疵，在之前的评测文章中，我也都有提到，但是不影响新手入门STM32，况且论坛里还有那么多的入门教程，也希望我的这些评测文章能对那些刚入门STM32的小伙伴们有一些帮助，少走一些弯路。据说2.0版本的开发板已经在进行紧锣密鼓的开发中了，当然，如果能有幸得到2.0版本开发板的使用机会，我会试着做一些远程控制相关的小项目。期待ing。。。。


### 文件的下载

- STM32工程：[uFUN_Extend_STM32_Prj.rar](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/uFUN_Extend_STM32_Prj.rar)
- 扩展板AD版本的原理图和PCB文件：[uFUN_Extend_PCB_Prj.rar](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/uFUN_Extend_PCB_Prj.rar)
- 扩展板各芯片的Datasheet：[uFUN_Extend_Datasheet.rar](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/uFUN_Extend_Datasheet.rar)

### uFUN评测系列文章

- [基于uFUN开发板的RGB调色板](http://www.wangchaochao.top/2019/04/06/uFun-7/)
- [基于uFUN开发板的心率计（一）DMA方式获取传感器数据](http://www.wangchaochao.top/2019/03/23/uFun-3/)
- [基于uFUN开发板的心率计（二）动态阈值算法获取心率值](http://www.wangchaochao.top/2019/03/31/uFun-5/)
- [基于uFUN开发板的心率计（三）Qt上位机的实现](http://www.wangchaochao.top/2019/04/05/uFun-6/)
- [Qt实现软件自动更新的一种简单方法](http://www.wangchaochao.top/2019/03/31/Qt-Update/)
- [【UFUN开发板评测】小巧而不失精致，简单而不失内涵——uFun开发板开箱爆照](http://www.wangchaochao.top/2019/03/09/uFun-1/)
- [如何使用串口来给STM32下载程序](http://www.wangchaochao.top/2019/03/20/uFun-4/)
- [STM32串口打印输出乱码的解决办法](http://www.wangchaochao.top/2019/03/17/uFun-2/)
- [Keil报错：cannot open source input file "core_cmInstr.h" 解决办法](http://www.wangchaochao.top/2019/03/09/uFun-0/)


--------

欢迎大家关注我的[个人博客](http://www.wangchaochao.top)：`www.wangchaochao.top`

或微信扫码关注我的公众号
