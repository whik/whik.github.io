---
layout:     post
title:    基于STM32+RT-Thread的新冠肺炎疫情监控平台
subtitle:	STM32F103疫情监控平台
date:       2020-08-18 23:00:00 +0800
author:     Wang Chao
header-img: img/fr8016h_2019_ncov.jpg
catalog:    true
tag:
    - ARM
---

>  亲朋好友，动动小手，一天3票，开发板谁要？

#### 厚着脸皮插播一条广告

我在疫情期间做的[基于STM32MP1和Qt的新冠肺炎疫情监控平台](https://mp.weixin.qq.com/s/oy6A4SM3OoqI0dlyMSFpMQ)，这个小项目报名参加了[意法半导体首届创客大赛——STM32创客秀](https://mp.weixin.qq.com/s/iQeRmje8r78MEI0l_X1xcg)，最近在投票阶段，如果有幸能入围决赛，ST官方会奖励开发板礼包，届时我会把开发板以抽奖的方式**回馈给大家**。

大家可以长按下面的二维码查看项目详情：

![投票二维码](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/投票二维码.png)

文末有**投票按钮**，如果你觉得我做得不错，欢迎投我3票（一天3票），或者直接跳转到文末点击 **阅读原文** 进入到[投票页面，](https://www.stmcu.com.cn/Vote/Itemsdetails/20
)投票结果只是最终结果的一部分。当然，如果你觉得其他创客项目做的不错，也欢迎多多投票支持。

#### 文章目录

上周末加班，这周末休息，有时间整理一篇之前做的基于RT-Thread的疫情监控平台。上一篇文章我们使用STM32F103 MCU[裸机开发](https://mp.weixin.qq.com/s/P8wfjWGk0YKE20rrQCqYJQ)的方式实现了疫情监控平台。这次我们玩点高端的，使用RT-Thread Studio来实现同样的功能，一起来看看吧！

- 文章目录
- 使用到的软件包
- 0.RT-Thread Studio的下载和安装
- 1.硬件准备
- 2.新建工程
- 3.添加LED闪烁功能
- 4.添加ESP8266软件包
- 5.疫情数据的获取
- 6.疫情数据的解析
- 7.疫情数据的显示
- 开源地址

最终的显示效果：

![显示效果](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/608186048791660758.jpg)

有效文件就这9个，其他的就全是图形化配置：

![有效文件](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-16_184450.jpg)

整个流程下来，如果顺利的话，可以在2个小时内完成。

#### 使用到的软件包

- at device：用于ESP8266配网
- webclient：用于发送HTTPS请求
- mbdetls：用于HTTPS加密
- cJSON：用于JSON数据解析

#### 0.RT-Thread Studio的下载和安装

> 一站式的 RT-Thread 开发工具，通过简单易用的图形化配置系统以及丰富的软件包和组件资源，让物联网开发变得简单和高效。 

![RT-Thread Studio](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-16_131335.jpg)

- 支持多种芯片，STM32全系列
- 支持创建裸机工程、RT-Thread Nano和Master工程
- 强大的代码编辑功能，基于Eclipse框架
- 免费无版权限制，基于开源Eclipse和ARM-GCC编译器。
- 支持多种仿真器，J-Link，ST-link等，支持在线调试，变量观察。
- SDK管理器，图形化配置RT-Thread软件包，同步RT-Thread最新版本。
- 集成Putty串口终端工具

更多的使用教程： https://www.rt-thread.org/page/studio.html 

目前已经最新版本为1.1.3版本，支持3种下载方式，我们选择最后一个下载方式，从RT-Thread 官网服务器上下载。

下载地址：http://117.143.63.254:9012/www/studio/download/RT-Thread%20Studio-v1.1.3-setup-x86_64_20200731-2100.exe

![下载链接](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-15_231958.jpg)

安装过程和常用的软件安装方法一样，选择安装路径，然后Next就行了。

#### 1.硬件准备

开发板用的是我在大四时自己设计的STM32开发板——NiceDay，基于STM32F103RET主控。  这是我设计的第二块板子（第一块是毕业设计两轮平衡车主板），是在大四快毕业时，毕设实物和论文完成之后还有点时间，就设计了这款板子，最开始是准备做桌面天气时钟的。 

![开发板](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/hardware.jpg)

#### 2.新建工程

RT-Thread Studio支持创建裸机工程、包含RT-Thread Nano版本的工程和包含Master版本的工程。这里，我们选择创建`RT-Thread 项目`，即包含完整版RT-Thread的工程。

![新建项目](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-15_232630.jpg)

工程支持基于芯片创建工程，或者基于已有的BSP创建，这里使用的是我自己设计的开发板，所以选择基于芯片，选择芯片型号：`STM32F103RE`，调试串口选择串口1，调试器选择J-Link，SWD接口。 

![新建项目](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-15_233025.jpg)

创建完成之后，直接按Ctrl+B编译整个工程，第一次编译时间会长一点，如果修改很少，下次再进行编译就会很快了，可以看到无警告无错误。

![编译结果](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-15_234339.jpg)

使用SWD接口连接JLink调试器和开发板，开发板上电，直接点击下载按钮，也可以使用快捷键Ctrl+Alt+D下载

![下载程序](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-15_234305.jpg)

底部可以看到下载信息，从LOG来看，下载的程序文件是Bin文件，比较，擦除，编程，验证，复位整个流程耗时13s左右。

![下载LOG](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-15_234214.jpg)

RT-Thread Studio是自带Putty串口终端的，点击终端图标：

![终端按钮](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-15_234123.jpg)

选择串口号、波特率、文字编码方式等。

![配置终端](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-15_234136.jpg)

底部切换到终端窗口，可以看到串口终端输出信息：

![串口终端](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-15_234151.jpg)

这样，不到5分钟，一个基于STM32F103RET6的工程模板就创建好了，包含RT-Thread完整版操作系统，整个过程不需要写一行代码，完全图形化配置。

#### 3.添加LED闪烁功能

作为单片机点灯小能手，RT-Thread下如何点灯是必须掌握的。打开RT-Thread组件图形化配置界面，可以看到默认开启了PIN和串口设备驱动的。

![图形化配置界面](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-15_234729.jpg)

在main.c文件中添加LED闪烁功能。包含头文件和添加宏定义

```c

#include <board.h>
#include <rtdevice.h>

#define LED_RED_PIN     GET_PIN(A, 7)
#define LED_BLUE_PIN    GET_PIN(A,6)

int main(void)
{
    int count = 1;
    rt_pin_mode(LED_RED_PIN, PIN_MODE_OUTPUT);
    rt_pin_mode(LED_BLUE_PIN, PIN_MODE_OUTPUT);

    while (count++)
    {
        rt_pin_write(LED_BLUE_PIN, PIN_LOW);
        rt_pin_write(LED_RED_PIN, PIN_LOW);
        rt_thread_mdelay(100);

        rt_pin_write(LED_BLUE_PIN, PIN_HIGH);
        rt_pin_write(LED_RED_PIN, PIN_HIGH);
        rt_thread_mdelay(100);
    }

    return RT_EOK;
}

```

重新编译，下载。可以看到LED闪烁起来了。工程默认是使用内部RC作为输入时钟，所以无论你的板子是8M还是12M，都可以正常闪烁。我的开发板是8M晶体，这里我们配置使用外部HSE作为输入时钟。打开`drivers->stm32f1xx_hal_conf.h`文件，修改HSE_VALUE宏定义为8M。

![晶体频率修改](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-16_125516.jpg)

打开`drivers->drv_clk.c`文件：

![时钟源修改](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-15_233441.jpg)

配置PLL时钟源为HSE，并设置倍频系数为9。

![时钟源修改](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-15_233749.jpg)

![倍频系数](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-15_233804.jpg)

这里根据实际板子晶体频率来设置，如果是12M晶体，倍频系数应该设置为6，如果是16M，需要参考时钟树，先2倍分频，然后9倍倍频。

```c

#include <rtdbg.h>

void system_clock_config(int target_freq_Mhz)
{
    RCC_OscInitTypeDef RCC_OscInitStruct = {0};

    /** Initializes the CPU, AHB and APB busses clocks
    */
    RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSE;
    RCC_OscInitStruct.HSEState = RCC_HSE_ON;
    RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSE;
    RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
 	........   
    //9倍频
    RCC_OscInitStruct.PLL.PLLMUL = RCC_PLL_MUL9;   //8*9=72M
    ........
}

```

这样就修改为外部8M晶体作为PLL时钟源，再次编译下载，和之前的现象是一样的。

#### 4.添加ESP8266软件包

联网设备，我们选择的是ESP8266-01S，如果看过上一篇[疫情监控三部曲——在STM32F103 MCU上实现（裸机版）](https://mp.weixin.qq.com/s/P8wfjWGk0YKE20rrQCqYJQ)，里面介绍了如何配置ESP8266 GET HTTPS请求， `配置工作模式 > 连接WiFi > 与服务器建立SSL连接 > 发送GET请求获取数据 `等等，整个流程固定而繁琐，那么能不能封装成一个模块，直接拿来使用呢？

![esp8266](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/esp8266.jpg)

这里就要介绍RT-Thread的AT Device软件包了，

> AT device 软件包是由 RT-Thread AT 组件针对不同 AT 设备的移植文件和示例代码组成，目前支持的 AT 设备有：ESP8266、ESP32、M26、MC20、RW007、MW31、SIM800C、W60X 、SIM76XX、A9/A9G、BC26 、AIR720、ME3616、M6315、BC28、EC200X、M5311系列设备等，目前上述设备都完成对 `AT socket` 功能的移植，及设备通过 AT 命令实现标准 socket 编程接口，完成 socket 通讯的功能，具体功能介绍可参考 《RT-Thread 编程指南》AT 命令章节 。
> https://www.rt-thread.org/document/site/programming-manual/at/at/

简单的说，就是我只需要调用这个软件包，然后修改WiFi账号和密码，就可以直接配置ESP8266联网，发送GET请求了。

由于AT Device依赖于libc组件，所以在添加AT Device软件包之前，先开启libc。

在`RT-Thread Settings`中点击libc灰色图标，变成彩色说明已经开启。

![组件配置](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-16_132711.jpg)

添加AT Device软件包，点击`立即添加`

 ![软件包](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-16_132836.jpg)

在弹出的软件包中心，搜索`at_device`，然后点击添加，添加到当前工程。

![软件包](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-16_133021.jpg)

在at_device软件包上右键，选择详细配置：

![软件包](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-16_133040.jpg)

在弹出的页面，选择我们使用的WiFi模块类型，乐鑫的ESP8266系列，并配置WiFi账号和密码，WiFi模块所连接的串口号。

![WiFi配置](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-16_133227.jpg)

点击保存之后，工程会重新进行配置，添加相应的软件包文件到当前工程，重新生成Makefile文件，rtconfig文件等等。

虽然我们在at_device配置中选择了uart2作为at_device设备连接的串口。但此时串口2并没有开启，还需要我们手动使能。

打开`drivers->board.h`文件，通过宏定义的方式使能串口2。

```c

#define BSP_USING_UART2
#define BSP_UART2_TX_PIN       "PA2"
#define BSP_UART2_RX_PIN       "PA3"

```

这样就开启了UART2的片上外设，`Ctrl + B`重新进行编译，时间会有些长，编译完成之后，可以看到flash文件大小明显比之前大了。

![编译结果](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-16_133834.jpg)

`Ctrl + Alt + D`重新下载运行，打开串口终端：

![终端](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-16_134819.jpg)

可以看到，UART2初始化成功，WiFi连接成功。说明我们的串口模块已经可以正常工作了。提示`[E/at.clnt] execute command (AT+CIPDNS_CUR?) failed!`失败信息，是因为当前ESP8266的固件版本不支持`AT+CIPDNS_CUR?`这条命令，把固件升级到最新版本就好了。这个不影响后面的操作，所以就不用在意这个了。

测试一下ifconfig和ping命令，都是正常的。

![终端](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-16_135253.jpg)

在RT-Thread Studio中配置ESP8266模块联网，整个流程只写了3行代码，可以说是非常的快速方便。

#### 5.疫情数据的获取

WiFi模块连接上互联网之后，就可以连接GET疫情数据的API接口` https://lab.isaaclin.cn/nCoV/api/overall `，然后读取返回的疫情数据。在上一篇的裸机工程中，是通过先和服务器建立SSL连接，然后发送GET HTTPS请求，获取到的返回数据，那RT-Thread有没有这样功能的软件包呢？这里就需要添加另一个软件包`webclient`。

> WebClient 软件包是 RT-Thread 自主研发的，基于 HTTP 协议的客户端的实现，它提供设备与 HTTP Server 的通讯的基本功能。
> WebClient 软件包功能特点如下：
>
> - 支持 IPV4/IPV6 地址；
> - 支持 GET/POST 请求方法；
> - 支持文件的上传和下载功能；
> - 支持 HTTPS 加密传输；
> - 完善的头部数据添加和处理方式。

和添加at_device一样，在软件包中心中搜索`webclient`，

![软件包](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-16_135930.jpg)

然后添加到当前工程，右键进行配置，由于我们的` https://lab.isaaclin.cn/nCoV/api/overall `这个疫情数据接口是HTTPS类型的，根据软件包使用手册，我们需要选择TLS模式中的 MbedTLS。勾选添加GET和POST示例。

![软件包配置](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-16_140230.jpg)

保存配置，看一下当前已经添加了哪些功能，可以看到有一些组件我们并没有去打开，但是已经被开启了，这是因为有些软件包是会依赖一些组件的，当前使能软件包时，一些依赖的组件也被同时使能。

![软件包](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-16_140818.jpg)

`Ctrl + B`编译，`Ctrl + Alt + D`下载运行。在终端输入`web_get_test`测试GET请求功能。

![GET示例](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-16_141723.jpg)

可以看到，执行get命令之后，会返回一个字符串，那么GET的是哪个地址呢？打开`packages->webclient-v2.1.2->samples->webclient_get_sample.c`文件，

![示例代码](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-16_141831.jpg)

可以看到GET的是这个地址：`http://www.rt-thread.com/service/rt-thread.txt`，我们用电脑上的浏览器访问一下：

![浏览器访问](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-16_142103.jpg)

经过实际测试发现，GET HTTPS请求，还需要使能软件模拟RTC这个组件，否则会报`assertion failed at function:gettimeofday, line number:19`错误。

![使能RTC](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-16_173811.jpg)

我们重新写一个获取疫情数据的函数，并导出到MSH。

usr_ncov.c文件内容

```c

//usr_ncov.c
#include "usr_ncov.h"

int get_NCOV_Data(void)
{
    char *uri = RT_NULL;
    struct webclient_session* session = RT_NULL;
    uint8_t *buffer = RT_NULL;
    int index, ret = 0;
    int bytes_read, resp_status;
    int content_length = -1;
    int buffer_size = 1600;
    uri = web_strdup(API_NCOV);
    rt_kprintf("start get api: %s\r\n", API_NCOV);
    if(uri != RT_NULL)
    {
        buffer = (unsigned char *) web_malloc(buffer_size);
        if (buffer == RT_NULL)
        {
            rt_kprintf("no memory for receive buffer.\n");
            ret = -RT_ENOMEM;
            goto __exit;
        }

        /* create webclient session and set header response size */
        session = webclient_session_create(buffer_size);
        if (session == RT_NULL)
        {
            ret = -RT_ENOMEM;
            goto __exit;
        }

        /* send GET request by default header */
        if ((resp_status = webclient_get(session, uri)) != 200)
        {
            rt_kprintf("webclient GET request failed, response(%d) error.\n", resp_status);
            ret = -RT_ERROR;
            goto __exit;
        }

        rt_kprintf("webclient get response data: \n");

        content_length = webclient_content_length_get(session);
        if (content_length < 0)
        {
            rt_kprintf("webclient GET request type is chunked.\n");

            do
            {
                bytes_read = webclient_read(session, buffer, buffer_size);
                if (bytes_read <= 0)
                    break;

                for (index = 0; index < bytes_read; index++)
                {
                    rt_kprintf("%c", buffer[index]);
                }
            } while (1);

            rt_kprintf("\n");
        }
        else
        {
            /* 读取服务器响应的数据 */
            bytes_read = webclient_read(session, buffer, content_length);
            rt_kprintf("data length:%d\n", bytes_read);

            buffer[bytes_read] = '\0';
            rt_kprintf("\n\n %s \n\n", buffer);
//            rt_kprintf("parse data\r\n");
            // parseData(buffer);		//解析函数
            rt_kprintf("\n");
        }

        __exit:
        if (session)
            webclient_close(session);

        if (buffer)
            web_free(buffer);
    }
    else
        rt_kprintf("api error: %s\n", API_NCOV);

    return ret;
}
MSH_CMD_EXPORT(get_NCOV_Data, get api ncov);

```

usr_ncov.h文件内容

```c

#ifndef APPLICATIONS_USR_NCOV_H_
#define APPLICATIONS_USR_NCOV_H_

#include <webclient.h>
#include <rtdevice.h>
#include <rtthread.h>

#define API_NCOV     "https://lab.isaaclin.cn/nCoV/api/overall"

int get_NCOV_Data(void);

#endif /* APPLICATIONS_USR_NCOV_H_

```

重新编译，下载，运行。在终端运行这个命令：

![命令获取疫情数据](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-16_174210.jpg)

可以看到获取到了返回的数据，长度1366个字节。下一步就是对这个JSON数据进行解析，获取到我们想要的疫情数据。

#### 6.疫情数据的解析

API返回的数据是JSON格式的，关于JSON的介绍和解析，可以查看[使用cJSON库解析和构建JSON字符串](https://mp.weixin.qq.com/s/_rS_uh1KIrwqqIfRsdSe5A)。数据的解析使用的开源小巧的cJSON解析库，我们可以在软件包管理中心直接添加：

![添加cJSON](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-16_174850.jpg)

在进行解析之前，先来分析一下JSON原始数据的格式：`results`键的值是一个数组，数组只有一个JSON对象，获取这个对象对应键的值可以获取到国内现存和新增确诊人数、累计和新增死亡人数，累计和新增治愈人数等数据。

全球疫情数据保存在`globalStatistics`键里，它的值是一个JSON对象，对象仅包含简单的键值对，这些键的值，就是全球疫情数据，其中`updateTime`键的值是更新时间，这是毫秒级UNIX时间戳，可以[转换为标准北京时间](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247484880&idx=1&sn=3df4ac0a465343abf98bd5b8561dc916&scene=21#wechat_redirect)。


```c

{
    "results": [{
        "currentConfirmedCount": 509,
        "currentConfirmedIncr": 16,
        "confirmedCount": 85172,
        "confirmedIncr": 24,
        "suspectedCount": 1899,
        "suspectedIncr": 4,
        "curedCount": 80015,
        "curedIncr": 8,
        "deadCount": 4648,
        "deadIncr": 0,
        "seriousCount": 106,
        "seriousIncr": 9,
        "globalStatistics": {
            "currentConfirmedCount": 4589839,
            "confirmedCount": 9746927,
            "curedCount": 4663778,
            "deadCount": 493310,
            "currentConfirmedIncr": 281,
            "confirmedIncr": 711,
            "curedIncr": 424,
            "deadIncr": 6
        },
        "updateTime": 1593227489355
    }],
    "success": true
}

```

先定义了结构体NCOV_DATA，用于存储国内和全球疫情数据：

```c

struct NCOV_DATA{
    int currentConfirmedCount;
    int currentConfirmedIncr;
    int confirmedCount;
    int confirmedIncr;
    int curedCount;
    int curedIncr;
    int deadCount;
    int deadIncr;
    int seriousCount;
    int seriousIncr;

    char updateTime[20];
};

```

对应的解析函数：

```c

#include <cJSON.h>

struct NCOV_DATA dataChina = {0, 0, 0, 0, 0, 0, 0, 0, 0, 0, "06-13 16:22"};;
struct NCOV_DATA dataGlobal = {0, 0, 0, 0, 0, 0, 0, 0, 0, 0, NULL};

int parseData(uint8_t *str)
{
    int ret = 0;
    cJSON *root, *result_arr;
    cJSON *result, *global;
    time_t updateTime;
    struct tm *time;

    root = cJSON_Parse((const char *)str);   //创建JSON解析对象，返回JSON格式是否正确

    if (root != 0)
    {
        rt_kprintf("JSON format ok, start parse!!!\n");
        result_arr = cJSON_GetObjectItem(root, "results");
        if(result_arr->type == cJSON_Array)
        {
//            rt_kprintf("result is array\n");
            result = cJSON_GetArrayItem(result_arr, 0);
            if(result->type == cJSON_Object)
            {
//                rt_kprintf("result_arr[0] is object\n");

                /* china data parse */
                dataChina.currentConfirmedCount = cJSON_GetObjectItem(result, "currentConfirmedCount")->valueint;
                dataChina.currentConfirmedIncr = cJSON_GetObjectItem(result, "currentConfirmedIncr")->valueint;
                dataChina.confirmedCount = cJSON_GetObjectItem(result, "confirmedCount")->valueint;
                dataChina.confirmedIncr = cJSON_GetObjectItem(result, "confirmedIncr")->valueint;
                dataChina.curedCount = cJSON_GetObjectItem(result, "curedCount")->valueint;
                dataChina.curedIncr = cJSON_GetObjectItem(result, "curedIncr")->valueint;
                dataChina.deadCount = cJSON_GetObjectItem(result, "deadCount")->valueint;
                dataChina.deadIncr = cJSON_GetObjectItem(result, "deadIncr")->valueint;

                rt_kprintf("**********china ncov data**********\n");
                rt_kprintf("%-23s: %8d, %-23s: %8d\n", "currentConfirmedCount", dataChina.currentConfirmedCount, "currentConfirmedIncr", dataChina.currentConfirmedIncr);
                rt_kprintf("%-23s: %8d, %-23s: %8d\n", "confirmedCount", dataChina.confirmedCount, "confirmedIncr", dataChina.confirmedIncr);
                rt_kprintf("%-23s: %8d, %-23s: %8d\n", "curedCount", dataChina.curedCount, "curedIncr", dataChina.curedIncr);
                rt_kprintf("%-23s: %8d, %-23s: %8d\n", "deadCount", dataChina.deadCount, "deadIncr", dataChina.deadIncr);

                /* global data parse */
                global = cJSON_GetObjectItem(result, "globalStatistics");
                if(global->type == cJSON_Object)
                {
                    dataGlobal.currentConfirmedCount = cJSON_GetObjectItem(global, "currentConfirmedCount")->valueint;
                    dataGlobal.currentConfirmedIncr = cJSON_GetObjectItem(global, "currentConfirmedIncr")->valueint;
                    dataGlobal.confirmedCount = cJSON_GetObjectItem(global, "confirmedCount")->valueint;
                    dataGlobal.confirmedIncr = cJSON_GetObjectItem(global, "confirmedIncr")->valueint;
                    dataGlobal.curedCount = cJSON_GetObjectItem(global, "curedCount")->valueint;
                    dataGlobal.curedIncr = cJSON_GetObjectItem(global, "curedIncr")->valueint;
                    dataGlobal.deadCount = cJSON_GetObjectItem(global, "deadCount")->valueint;
                    dataGlobal.deadIncr = cJSON_GetObjectItem(global, "deadIncr")->valueint;

                    rt_kprintf("\n**********global ncov data**********\n");
                    rt_kprintf("%-23s: %8d, %-23s: %8d\n", "currentConfirmedCount", dataGlobal.currentConfirmedCount, "currentConfirmedIncr", dataGlobal.currentConfirmedIncr);
                    rt_kprintf("%-23s: %8d, %-23s: %8d\n", "confirmedCount", dataGlobal.confirmedCount, "confirmedIncr", dataGlobal.confirmedIncr);
                    rt_kprintf("%-23s: %8d, %-23s: %8d\n", "curedCount", dataGlobal.curedCount, "curedIncr", dataGlobal.curedIncr);
                    rt_kprintf("%-23s: %8d, %-23s: %8d\n", "deadCount", dataGlobal.deadCount, "deadIncr", dataGlobal.deadIncr);

                } else return 1;

                /* 毫秒级时间戳转字符串 */
                updateTime = (time_t )(cJSON_GetObjectItem(result, "updateTime")->valuedouble / 1000);
                updateTime += 8 * 60 * 60; /* UTC8校正 */
                time = localtime(&updateTime);
                /* 格式化时间 */
                strftime(dataChina.updateTime, 20, "%m-%d %H:%M", time);
                rt_kprintf("update: %s\r\n", dataChina.updateTime);/* 06-24 11:21 */
				//数据在LCD显示
                //gui_show_ncov_data(dataChina, dataGlobal);
            } else return 1;
        } else return 1;
        rt_kprintf("\nparse complete \n");
    }
    else
    {
        rt_kprintf("JSON format error:%s\n", cJSON_GetErrorPtr()); //输出json格式错误信息
        return 1;
    }
    cJSON_Delete(root);

    return ret;
}

```

在数据接收完成之后，对JSON数据进行解析。

![解析结果](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/2020-08-16_175808.jpg)

#### 7.疫情数据的显示

数据解析出来之后，剩下的就简单了，把上一篇文章中9341的驱动文件移植过来就好了。

液晶屏使用的是3.2寸 LCD，IL9341驱动芯片，320*240分辨率，16位并口。由于屏幕分辨率比较低，可显示的内容有限，所以只是显示了最基本的几个疫情数据。为了减小程序大小，GUI只实现了基本的画点，画线函数，字符的显示，采用的是部分字符取模，只对程序中用到的汉字和字符进行取模。为了增强可移植性，程序中并没有使用外置SPI Flash存储整个字库。

由于RT-Thread Studio使用的HAL库，所以LCD的GPIO初始化函数需要修改一下：

```c

void lcd_gpio_init(void)
{
    GPIO_InitTypeDef GPIO_InitStructure;

    __HAL_RCC_GPIOA_CLK_ENABLE();
    __HAL_RCC_GPIOC_CLK_ENABLE();
    __HAL_RCC_GPIOB_CLK_ENABLE();
    __HAL_RCC_AFIO_CLK_ENABLE();
    __HAL_AFIO_REMAP_SWJ_NOJTAG();

    GPIO_InitStructure.Mode = GPIO_MODE_OUTPUT_PP;
    GPIO_InitStructure.Pull = GPIO_PULLUP;
    GPIO_InitStructure.Speed = GPIO_SPEED_FREQ_HIGH;
    GPIO_InitStructure.Pin = GPIO_PIN_9 | GPIO_PIN_8 | GPIO_PIN_7 | GPIO_PIN_6;
    HAL_GPIO_Init(GPIOC, &GPIO_InitStructure); //GPIOC

    GPIO_InitStructure.Pin = GPIO_PIN_8;    //背光引脚PA8
    HAL_GPIO_Init(GPIOA, &GPIO_InitStructure); //GPIOC

    GPIO_InitStructure.Pin = GPIO_PIN_All;
    HAL_GPIO_Init(GPIOB, &GPIO_InitStructure);

    HAL_GPIO_WritePin(GPIOC, GPIO_PIN_9 | GPIO_PIN_8 | GPIO_PIN_7 | GPIO_PIN_6, GPIO_PIN_SET);
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_8, GPIO_PIN_SET);
    HAL_GPIO_WritePin(GPIOB, GPIO_PIN_All, GPIO_PIN_SET);
}

```

延时函数换成：

```c

rt_thread_mdelay(nms);

```

还有一点，在Keil中，文字编码选择GBK编码，1个汉字占用2个字节，而RT-Thread Studio为UTF-8编码，1个汉字占用3个字节，汉字显示函数需要调整：

```c

void gui_show_chn(uint16_t x0, uint16_t y0, char *chn)
{
    uint8_t idx = 0;
    uint8_t* code[3]; //UTF-8:国=E59BBD
   
    uint8_t size = sizeof(FONT_16X16_TABLE) / sizeof(FONT_16X16_TABLE[0]);
    /* 遍历汉字，获取索引 */
    for(idx = 0; idx < size; idx++)
    {
        code[0] = FONT_16X16_TABLE[idx].chn;
        code[1] = FONT_16X16_TABLE[idx].chn + 1;
        code[2] = FONT_16X16_TABLE[idx].chn + 2;
        //汉字内码一致
        if(!(strcmp(code[0], chn) || strcmp(code[1], chn+1) || strcmp(code[2], chn+2)))
        {
            gui_show_F16X16_Char(x0, y0, idx, WHITE);
            return;
//            break;
        }
    }
}

```

疫情数据显示函数：

```c

void gui_show_ncov_data(struct NCOV_DATA china, struct NCOV_DATA global)
{
    uint8_t y0 = 20;

    lcd_clear(BLACK);
    gui_show_bar();

    gui_drawLine(0, 18, 320, DIR_X, WHITE);
    gui_drawLine(0, 38, 320, DIR_X, WHITE);
    gui_drawLine(0, 138, 320, DIR_X, WHITE);
    gui_drawLine(0, 158, 320, DIR_X, WHITE);
    gui_drawLine(0, 220, 320, DIR_X, WHITE);

    /* "国内疫情" */
    gui_show_chn_string(128, y0, "国内疫情");
    gui_show_line_data(40, "现存确诊：", china.currentConfirmedCount, "较昨日：", china.currentConfirmedIncr);
    gui_show_line_data(60, "累计确诊：", china.confirmedCount, "较昨日：", china.confirmedIncr);
    gui_show_line_data(80, "累计治愈：", china.curedCount, "较昨日：", china.curedIncr);
    gui_show_line_data(100, "现存重症：", china.seriousCount, "较昨日：", china.seriousIncr);
    gui_show_line_data(120, "累计死亡：", china.deadCount, "较昨日：", china.deadIncr);

    /* 全球疫情 */
    gui_show_chn_string(128, 140, "全球疫情");
    gui_show_line_data(160, "现存确诊：", global.currentConfirmedCount, "较昨日：", global.currentConfirmedIncr);
    gui_show_line_data(180, "累计治愈：", global.curedCount, "较昨日：", global.curedIncr);
    gui_show_line_data(200, "累计死亡：", global.deadCount, "较昨日：", global.deadIncr);
    
    gui_show_chn_string(160, 222, "更新于：");
    gui_show_F8X16_String(230, 222, china.updateTime, GREEN);
}

```

#### 最终显示效果

![最终效果](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/608186048791660758.jpg)

#### 开源地址

代码已经开源，地址在文末，欢迎大家参与，丰富这个小项目的功能！

- 基于STM32+RT-Thread的疫情监控平台
  https://github.com/whik/rtt_2019_ncov 
- 基于STM32F103的疫情监控平台（裸机版）
  https://github.com/whik/stm32_2019_ncov

如果GitHub下载速度太慢，可以关注我的公众号，**电子电路开发学习**（ID: MCU149），在后台回复【**RTT疫情监控**】获取基于RT-Thread的工程源码，或者回复【**STM32疫情监控**】获取裸机版本的工程源码，我会把下载链接发给你。

#### 推荐阅读 

- [疫情监控三部曲——在STM32F103 MCU上实现（裸机版）](https://mp.weixin.qq.com/s/P8wfjWGk0YKE20rrQCqYJQ)
- [[开源]基于桌面版Qt的肺炎疫情监控平台](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247484320&idx=1&sn=c3b2e06a7b96ec2789de50fb3d56b835&scene=21#wechat_redirect)
- [[开源]基于STM32MP1+Qt的疫情监控平台](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247484485&idx=1&sn=4f74131918d92a8bda3ec4c5f83bd451&scene=21#wechat_redirect)
- [使用cJSON库解析和构建JSON字符串](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247484138&idx=1&sn=2c2548595fdd1c187fbdb40469dc3a02&scene=21#wechat_redirect)

#### 文末再厚着脸皮插播一条广告

我在疫情期间做的[基于STM32MP1和Qt的新冠肺炎疫情监控平台](https://mp.weixin.qq.com/s/oy6A4SM3OoqI0dlyMSFpMQ)，这个小项目报名参加了[意法半导体首届创客大赛——STM32创客秀](https://mp.weixin.qq.com/s/iQeRmje8r78MEI0l_X1xcg)，最近在投票阶段，如果有幸能入围决赛，ST官方会奖励开发板礼包，届时我会把开发板以抽奖的方式**回馈给大家**。

大家可以长按下面的二维码查看项目详情：

![投票二维码](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200816/投票二维码.png)

文末有**投票按钮**，如果你觉得我做得不错，欢迎投我3票（一天3票），或者直接跳转到文末点击 **阅读原文** 进入到[投票页面](https://www.stmcu.com.cn/Vote/Itemsdetails/20)，投票结果只是最终结果的一部分。当然，如果你觉得其他创客项目做的不错，也欢迎多多投票支持。

