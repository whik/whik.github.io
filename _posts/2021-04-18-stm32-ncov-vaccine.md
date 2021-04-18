---
layout:     post
title: 基于STM32+ESP8266的全球新冠疫苗接种数据实时监控平台
subtitle:	我们一起苗苗苗
date:       2021-04-18 11:57:00 +0800
author:     Wang Chao
header-img: img/vaccine.jpg
catalog:    true
tag:
    - STM32
---

>**中国疾控中心免疫规划首席专家王华庆说，我国要建立免疫屏障，可能需要10亿以上的人接种新冠疫苗，接种率越高，免疫屏障就越牢固。**

张文宏医生说，**“年轻人打疫苗是为国家做贡献，最好在今年打、尽快打！”**。最近，你打疫苗了吗？

![一起苗苗苗](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/210418/一起苗苗苗.jpg)

**上周三，我去打疫苗了，第一针。**预约，健康状况询问，打针，观察，人不算太多，整个流程下来很顺利，目前没有任何的不良反应。

![接种界面](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/210418/接种界面.jpg)

去年春节期间，宅在家为国家做贡献的期间，我做了一系列的疫情数据监控练手小项目，比如[基于桌面Qt的](https://mp.weixin.qq.com/s/O0ie994xBgtGV3YFldt6MA)，[基于STM32MP157的](https://mp.weixin.qq.com/s/oy6A4SM3OoqI0dlyMSFpMQ)，基于[STM32F103裸机](https://mp.weixin.qq.com/s/P8wfjWGk0YKE20rrQCqYJQ)和[RT-Thread版本的](https://mp.weixin.qq.com/s/QEsMRars9wiLiTgtAt1pPg)，可能很多读者都是从那个时候开始关注我的。

其中PC桌面和ARM Linux版本的程序代码，由于国内疫情的好转，使用到的API接口可能已经关闭，或者JSON数据格式有很大的改动，所以在运行时，会出现程序异常停止，或者无数据显示的情况，这是正常的。

而STM32F103裸机和RTOS版本，前几天我试了一下，还是可以正常运行的。

![疫情监控数据](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/210418/疫情监控数据.JPG)

> 从3月24日起，我国启动了新冠疫苗接种数据的日报制度，国家卫健委每日会在官网公布疫苗接种总数，这也是**人类疫苗接种史上首次启动国家级最大规模的日报制度**。

能不能把之前做的疫情数据监控改为疫苗接种数据呢？放在桌面上，看着每天新增的接种数据，关注着我国逐步建立免疫屏障的进度！先上微信看看，有没有疫苗接种相关的数据，果然，有实时的全球疫苗接种数据。

![疫苗数据](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/210418/疫苗数据.jpg)

由于之前做过[类似的项目](https://mp.weixin.qq.com/s/QEsMRars9wiLiTgtAt1pPg)，所以做起来还是比较简单的。改一下API接口，JSON解析函数，调整一下显示界面就可以了。于是[之前的项目](https://mp.weixin.qq.com/s/QEsMRars9wiLiTgtAt1pPg)就变成了**全球新冠疫苗接种数据实时监控平台**。名字很唬人，其实很简单。效果展示：

![疫苗接种数据](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/210418/疫苗接种数据.JPG)

这个小项目的代码是完全开源的，你可以在文末获取项目的**Gitee仓库地址**，或者在我的公众号后台回复【**STM32疫苗**】关键词，来直接获取代码压缩包的下载链接。

下面介绍如何来制作这样一个**全球新冠疫苗接种数据实时监控平台**，并不需要和我一样的硬件，只需要了解整个项目的实现过程，在任何平台上实现都是一样的。

主要包括以下内容：

- **硬件平台准备**
- **API接口获取**
- **ESP8266配置**
- **JSON数据解析**
- **LCD数据显示**
- **开源地址**

#### 硬件平台准备

基本硬件主要有以下：

- 一款主控芯片。比如ARM、51、AVR等平台，最好是片上外设多一点的，GPIO，串口，定时器这些，如果带有WiFi功能就更好了，这意味着可以省掉一个WiFi模块。如ESP8266就是一款具有WiFi功能的MCU，可以进行SDK开发。
- 一个联网模块。比如WiFi，2G/4G，有线以太网等，如果主控芯片带有WiFi功能，可以省略！
- 一个显示设备。比如LCD，OLED，墨水屏，点阵屏等，用于数据的显示。
- 一些必要的调试工具，如下载器，USB-TTL模块等，最重要的是要有一颗**不怕困难的心**。

这里我使用的是STM32+ESP8266的方案，通过AT指令的方式和WiFi模块进行交互。

主要包括以下硬件模块：

- STM32F103RET6，主控芯片，完成串口交互，LCD控制等
- ESP8266-01，用于连接WiFi获取数据
- 3.2' TFT LCD，用于显示数据
- SPI Flash，用于存储字库
- LED，用于指示状态
- [J-Link](https://mp.weixin.qq.com/s/tnW6wFI6FIuAYxNZVFRNlQ)，USB-TTL，用于程序调试与下载。

实际开发板如下，这是一款我在大学期间自己画的小板子：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/ItLb3ZD7E5sc6YoKmSZ8VhpYnE495vut7kPGK904FSMOXbQZhM3lq3zBfwzl5MJJNakXrGyVCsU5ehzIdqEhTw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### API接口获取

API和之前一样是通过腾讯新闻获取，使用浏览器打开**腾讯新闻-实时接种数据**页面：

`https://news.qq.com/ylh5/worldvaccine.htm#/`

PC和手机端界面：

![PC界面](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/210418/腾讯新闻全球疫苗接种数据.jpg)

F12打开开发者模式，切换到`Network`网络面板，刷新页面，找到请求的URL，可以看到是POST方式：

![POST方式](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/210418/获取疫苗接种数据API.jpg)

并使用浏览器直接访问这个页面，

![访问疫苗接种数据API](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/210418/访问疫苗接种数据API.jpg)

可以看到是GET方式：

![GET方式](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/210418/获取实际的GET请求API信息.jpg)

说明支持两种HTTP请求方式：

GET：

```c
182.254.21.58:443
https://api.inews.qq.com/newsqa/v1/automation/modules/list?modules=VaccineTopData
```

POST:

```c
182.254.21.36:443
https://api.inews.qq.com/newsqa/v1/automation/modules/list?modules=VaccineTopData
```

#### ESP8266配置

有了IP地址，端口号和API地址，下一步就是使用[ESP8266](https://mp.weixin.qq.com/s/XHtvEI7rL7KdIT-EvzWc1Q)来访问这个API接口，并得到返回的数据。为了和之前的程序兼容，这里我选择的是GET请求的方式。EPS8266使用AT指令的方式进行配置，由于之前的项目已经完成了[ESP8266配置](https://mp.weixin.qq.com/s/XHtvEI7rL7KdIT-EvzWc1Q)的部分，只需要修改一下IP地址，端口号和API地址即可，配置流程使用串口打印出来，如下：

![JSON解析](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/210418/JSON解析.jpg)

可以看到返回的数据是常用的[JSON格式](https://mp.weixin.qq.com/s/_rS_uh1KIrwqqIfRsdSe5A)字符串，长度为265字节。有了JSON数据，下一步就是[对数据进行解析](https://mp.weixin.qq.com/s/_rS_uh1KIrwqqIfRsdSe5A)，使用的是[cJSON解析库](https://mp.weixin.qq.com/s/_rS_uh1KIrwqqIfRsdSe5A)，由于数据格式比较简单，解析函数也不多。

格式化之后的数据：

![json原始数据](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/210418/json原始数据.jpg)

#### JSON数据解析

由于数据比较简单，解析函数就不贴出来了。可以点击：[了解JSON格式](https://mp.weixin.qq.com/s/XGRtn0dNzKgGjiEUh8k1Eg)，[使用cJSON解析](https://mp.weixin.qq.com/s/2ho5xsXNzWn8IU8lWdNKPw)，[使用cJSON构建](https://mp.weixin.qq.com/s/_rS_uh1KIrwqqIfRsdSe5A)，获取以往的文章介绍。

在解析数据时，要注意，其中"中国"，"全球"汉子为UTF-8编码，在Keil中进行键值解析时，需要修改字符编码为UTF-8：

![UTF-8](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/210418/UTF-8.jpg)

或者在GBK2312编码状态下，键值对字符串"中国"="涓浗"，这一点要特别注意：

![GBK2312](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/210418/GBK2312.jpg)

#### LCD数据显示

LCD显示部分，和上一个项目有所不同，之前是把使用到的部分字体进行取模，获取点阵数据，然后再显示出来。这次我是读取板子上的SPI Flash字库文件，然后进行显示的，这样可以显示所有的文字，比较方便。UI界面比较简单，没有采用GUI库。

![疫苗接种数据](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/210418/疫苗接种数据.JPG)

#### 开源地址

这个项目的代码工程已经开源在国内的Gitee平台：

- STM32获取疫苗接种数据
  
  `https://gitee.com/whik/stm32_ncov_vaccine`
  
- STM32获取肺炎疫情数据
  
  `https://gitee.com/whik/stm32_2019_ncov`

本来计划根据IP定位的地址，然后查询距离最近的接种点信息，但是没找到合适的API接口，如果腾讯开放这种类型的接口，我也会更新的，欢迎大家关注，Fork & Star！

WiFi和API接口配置信息在config.h文件里：

![config](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/210418/config.jpg)

这个项目，总体来说，没有太多的技术含量和实用价值，但是作为一个学习的小项目还是很不错的！如果修改一下接口，比如改为天气信息，空气质量，新闻动态，股票信息，尾号限行，油价信息等等，放在工作桌面上，也算是一个比较有科技感的摆件了！

#### 一起苗苗苗

> **中国疾控中心免疫规划首席专家王华庆说，如果大家都把接种疫苗往后拖，那么免疫屏障就永远建立不起来，我们想摘掉口罩的愿望可能就不会实现。如果大家接种得快，这个屏障可能就会早一天到来。要尽早恢复到正常的生活，疫苗是目前的最佳选择。**

新冠肺炎是百年难遇的灾难，要全面控制病毒的传播，使人类社会恢复正常的生活，办法只有一个：实现群体免疫！而实现群体免疫最快最好的方法就是：**打疫苗**！为了自己的安全，也为了尽快建立免疫屏障，在这里倡议大家尽快去接种，**建立全民免疫，需要你的一“臂”之力！**

![一臂之力](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/210418/一臂之力.jpg)
