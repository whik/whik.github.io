---
layout:     post
title:    CRC校验原理及其实现
subtitle:	CRC校验
date:       2020-09-20 23:57:00 +0800
author:     Wang Chao
header-img: img/post-bg-c-interface-implementatios.jpg
catalog:    true
tag:
    - 电子
---

> 由于公众号申请的时间比较晚，所以没有留言互动功能，最近公众号上线了**读者讨论**功能，和留言功能差不多，对本篇文章有什么感想的都可以到文章末尾留言评论。

#### 目录

- 前言
- CRC算法简介
- CRC计算
- CRC校验
- CRC计算的C语言实现
- CRC计算工具
- 总结

#### 前言

最近的工作中，要实现对通信数据的CRC计算，所以花了两天的时间好好研究了一下，周末有时间整理了一下笔记。

一个完整的数据帧通常由以下部分构成：

![2020-09-20_151837](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200920/2020-09-20_151837.jpg)

校验位是为了保证数据在传输过程中的完整性，采用一种指定的算法对原始数据进行计算，得出的一个校验值。接收方接收到数据时，采用同样的校验算法对原始数据进行计算，如果计算结果和接收到的**校验值一致**，说明数据校验正确，这一帧数据可以使用，如果不一致，说明传输过程中出现了差错，这一帧数据丢弃，请求重发。

常用的校验算法有奇偶校验、校验和、CRC，还有LRC、BCC等不常用的校验算法。

以串口通讯中的奇校验为例，如果数据中1的个数为奇数，则奇校验位0，否则为1。

例如原始数据为：0001 0011，数据中1的个数（或各位相加）为3，所以奇校验位为0。这种校验方法很简单，但这种校验方法有很大的误码率。假设由于传输过程中的干扰，接收端接收到的数据是0010 0011，通过奇校验运算，得到奇校验位的值为0，虽然校验通过，但是数据已经发生了错误。

![2020-09-20_125410](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200920/2020-09-20_125410.jpg)

校验和同理也会有类似的错误：

![2020-09-20_125438](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200920/2020-09-20_125438.jpg)

一个好的校验校验方法，配合数字信号编码方式，如(差分)曼彻斯特编码，(不)归零码等对数据进行编码，可大大提高通信的健壮性和稳定性。例如以太网中使用的是CRC-32校验，曼彻斯特编码方式。本篇文章介绍CRC校验的原理和实现方法。

#### CRC算法简介

> 循环冗余校验（Cyclic Redundancy Check， CRC）是一种根据网络数据包或计算机文件等数据产生简短固定位数校验码的一种信道编码技术，主要用来检测或校验数据传输或者保存后可能出现的错误。它是利用除法及余数的原理来作错误侦测的。

CRC校验计算速度快，检错能力强，易于用编码器等硬件电路实现。从检错的正确率与速度、成本等方面，都比奇偶校验等校验方式具有优势。因而，CRC 成为计算机信息通信领域最为普遍的校验方式。常见应用有以太网/USB通信，压缩解压，视频编码，图像存储，磁盘读写等。

#### CRC参数模型

不知道你是否遇到过这种情况，同样的CRC多项式，调用不同的CRC计算函数，得到的结果却不一样，而且和手算的结果也不一样，这就涉及到CRC的参数模型了。计算一个正确的CRC值，需要知道CRC的参数模型。

一个完整的CRC参数模型应该包含以下信息：WIDTH，POLY，INIT，REFIN，REFOUT，XOROUT。

- NAME：参数模型名称。
- WIDTH：宽度，即生成的CRC数据位宽，如CRC-8，生成的CRC为8位
- POLY：十六进制多项式，省略最高位1，如 x8 + x2 + x + 1，二进制为1 0000 0111，省略最高位1，转换为十六进制为0x07。
- INIT：CRC初始值，和WIDTH位宽一致。
- REFIN：true或false，在进行计算之前，原始数据是否翻转，如原始数据：0x34 = 0011 0100，如果REFIN为true，进行翻转之后为0010 1100 = 0x2c
- REFOUT：true或false，运算完成之后，得到的CRC值是否进行翻转，如计算得到的CRC值：0x97 = 1001 0111，如果REFOUT为true，进行翻转之后为11101001 = 0xE9。
- XOROUT：计算结果与此参数进行异或运算后得到最终的CRC值，和WIDTH位宽一致。

通常如果只给了一个多项式，其他的没有说明则：INIT=0x00，REFIN=false，REFOUT=false，XOROUT=0x00。

常用的21个标准CRC参数模型：

![2020-09-20_131404](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200920/2020-09-20_131404.jpg)

CRC校验在电子通信领域非常常用，可以说有通信存在的地方，就有CRC校验：

- 美信(MAXIM)的芯片DS2401/DS18B20，都是使用的CRC-8/MAXIM模型
- SD卡或MMC使用的是CRC-7/MMC模型
- Modbus通信使用的是CRC-16/MODBUS参数模型
- USB协议中使用的CRC-5/USB和CRC-16/USB模型
- STM32自带的硬件CRC计算模块使用的是CRC-32模型

至于多项式的选择，初始值和异或值的选择，输入输出是否翻转，这就涉及到一定的编码和数学知识了。感兴趣的朋友，可以了解一下每个CRC模型各个参数的来源。至于每种参数模型的检错能力、重复率，需要专业的数学计算了，不在本文讨论的范畴内。

#### CRC计算

好了，了解了CRC参数模型知识，下面手算一个CRC值，来了解CRC计算的原理。

**问：原始数据：0x34，使用CRC-8/MAXIN参数模型，求CRC值？**

答：根据CRC参数模型表，得到CRC-8/MAXIN的参数如下：

```C
POLY = 0x31 = 0011 0001(最高位1已经省略)
INIT = 0x00
XOROUT = 0x00
REFIN = TRUE
REFOUT = TRUE
```

有了上面的参数，这样计算条件才算完整，下面来实际计算：

```
0.原始数据 = 0x34 = 0011 0100，多项式 = 0x31 = 1 0011 0001
1.INIT = 00，原始数据高8位和初始值进行异或运算保持不变。
2.REFIN为TRUE，需要先对原始数据进行翻转：0011 0100 > 0010 1100
3.原始数据左移8位，即后面补8个0：0010 1100 0000 0000
4.把处理之后的数据和多项式进行模2除法，求得余数：
原始数据：0010 1100 0000 0000 = 10 1100 0000 0000
多项式：1 0011 0001
模2除法取余数低8位：1111 1011
5.与XOROUT进行异或，1111 1011 xor 0000 0000 = 1111 1011 
6.因为REFOUT为TRUE，对结果进行翻转得到最终的CRC-8值：1101 1111 = 0xDF
7.数据+CRC：0011 0100 1101 1111 = 34DF，相当于原始数据左移8位+余数。
```

模2除法求余数：

![2020-09-20_160023](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200920/2020-09-20_160023.jpg)

验证手算结果：

![2020-09-20_154725](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200920/2020-09-20_154725.jpg)

可以看出是一致的，当你手算的结果和工具计算结果不一致时，可以看看INIT，XOROUT，REFINT，REFOUT这些参数是否一致，有1个参数不对，计算出的CRC结果都不一样。

#### CRC校验

上面通过笔算的方式，讲解了CRC计算的原理，下面来介绍一下如何进行校验。

按照上面CRC计算的结果，最终的数据帧：0011 0100 1101 1111 = 34DF，前8位0011 0100是原始数据，后8位1101 1111 是 CRC结果。

接收端的校验有两种方式，一种是和CRC计算一样，在本地把**接收到的数据和CRC分离**，然后在本地对数据进行CRC运算，得到的CRC值和接收到的CRC进行比较，如果一致，说明数据接收正确，如果不一致，说明数据有错误。

另一种方法是把整个数据帧进行CRC运算，因为是数据帧相当于把原始数据左移8位，然后加上余数，如果直接对整个数据帧进行CRC运算（除以多项式），那么余数应该为0，如果不为0说明数据出错。

![2020-09-20_161239](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200920/2020-09-20_161239.jpg)

而且，不同位出错，余数也不同，可以证明，余数与出错位数的对应关系只与CRC参数模型有关，而与原始数据无关。

例如正确的数据帧：0011 0100 1101 1111，前8位0011 0100是原始数据，后8位1101 1111 是 CRC结果。我们来计算一下当只有1位发生变化时，CRC计算得到的余数。



#### CRC计算的C语言实现

无论是用C还是其他语言，实现方法网上很多，这里我找了一个基于C语言的CRC计算库，里面包含了常用的21个CRC参数模型计算函数，可以直接使用，只有`crcLib.c`和`crcLib.h`两个文件。

GitHub地址：https://github.com/whik/crc-lib-c

使用方法非常简单：

```c
#include <stdio.h>
#include <stdlib.h>
#include "crcLib.h"

int main()
{
    uint8_t LENGTH = 10;
    uint8_t data[LENGTH];
    uint8_t crc;

    for(int i = 0; i < LENGTH; i++)
    {
        data[i] = i*5;
        printf("%02x ", data[i]);
    }
    printf("\n");

    crc = crc8_maxim(data, LENGTH);

    printf("CRC-8/MAXIM:%02x\n", crc);
    return 0;
}
```

计算结果：

![2020-09-20_185246](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200920/2020-09-20_185246.jpg)

#### CRC计算工具

下面这几款工具都可以自定义CRC算法模型，而且都有标准CRC模型可供选择。如果自己用C语言或者Verilog实现校验算法时，非常适合作为标准答案进行验证。

- 在线计算：www.ip33.com/crc.html
- 离线计算工具：CRC_Calc v0.1.exe或者GCRC.exe

格西CRC计算器：

![2020-09-20_153450](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200920/2020-09-20_153450.jpg)

公众号后台回复【CRC】，获取以上两款CRC计算工具的下载链接。

http://www.geshe.com/home/products/GToolbox/bin/GCRC.exe

#### 总结

CRC校验并不能100%的检查出数据的错误，非常低的概率会出现CRC校验正确但数据中有错误位的情况。这和CRC的位数，多项式的选择等等有很大的关系，所以在实际使用中尽量选择标准CRC参数模型，这些多项式参数都是经过理论计算得出的，可以提高CRC的检错能力。CRC校验可以检错，也可以纠正单一比特的错误，你知道纠错的原理吗？

#### 参考资料

- https://www.cnblogs.com/liushui-sky/p/9962123.html
- https://segmentfault.com/a/1190000018094567

#### 推荐阅读

- [[踩坑]CMOS器件输入管脚不能悬空？](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247485008&idx=2&sn=df78a50a5a3f1ee0f8f9103536ed49c8&chksm=fadfa03ecda82928a6a40c20fcda2fc04540a164cb1dbf369aa35dec7a5d6208cde2b885faab&token=287158831&lang=zh_CN#rd)
- [[开源]基于STM32F103的疫情监控平台（RT-Thread）](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247484991&idx=1&sn=a6fcd2c81c1768434ebb9ff829c72b5b&scene=21#wechat_redirect)
- [[开源]基于Qt+STM32MP1的疫情监控平台](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247484485&idx=1&sn=4f74131918d92a8bda3ec4c5f83bd451&scene=21#wechat_redirect)
- [[踩坑]STM32外部8M晶体不起振会有什么现象？](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247484868&idx=1&sn=80472832296d126953296de40c9934f6&scene=21#wechat_redirect)