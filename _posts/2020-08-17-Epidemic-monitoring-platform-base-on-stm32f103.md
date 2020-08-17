---
layout:     post
title:    疫情监控三部曲——在STM32F103 MCU上实现（裸机版）
subtitle:	STM32F103疫情监控平台
date:       2020-08-17 23:00:00 +0800
author:     Wang Chao
header-img: img/fr8016h_2019_ncov.jpg
catalog:    true
tag:
    - ARM
---

好久没更新文章了，看看又做了什么些好玩的东西。

#### 前言

2020，新冠肺炎疫情在全球蔓延，国内得到了有效的控制，最近国内部分地区的疫情形势又紧张起来。

不知道大家是否了解我之前做的一个新冠肺炎疫情监控平台，基于跨平台Qt实现，从桌面Qt，到嵌入式Qt，相关文章：

**基于桌面Qt环境的疫情监控平台开发笔记**：

- [[开源]基于桌面Qt的肺炎疫情监控平台](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247484315&idx=1&sn=5ebbdadb3d048ece3e2fe112e078ffbf&scene=21#wechat_redirect)
- [[开源]基于桌面Qt的肺炎疫情监控平台1.1版本](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247484320&idx=1&sn=c3b2e06a7b96ec2789de50fb3d56b835&scene=21#wechat_redirect)

**基于嵌入式Qt环境的疫情监控平台开发笔记**：

- [[开源]我用STM32MP1做了个疫情监控平台1—交叉编译环境搭建](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247484460&idx=1&sn=abb259af31d32812d5ce11bb94cbc91c&scene=21#wechat_redirect)
- [[开源]我用STM32MP1做了个疫情监控平台2—Qt环境搭建](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247484460&idx=2&sn=6794b0d8deaae450a3ba99c6a0d3fc31&scene=21#wechat_redirect)
- [[开源]我用STM32MP1做了个疫情监控平台3—疫情监控平台实现](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247484460&idx=3&sn=091e96c589679c4f27721d8a6e50170c&scene=21#wechat_redirect)
- [[开源]我用STM32MP1做了个疫情监控平台4—功能完善界面重新设计](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247484485&idx=1&sn=4f74131918d92a8bda3ec4c5f83bd451&scene=21#wechat_redirect)

作为疫情监控三部曲：**桌面PC > 嵌入式ARM Linux > MCU**。在前面两个平台上实现之后，就想着在内存和性能都比较有限的MCU上实现，比如STM32F103，但一直都没有找到一个合适的API接口，直到最近发现了一个数据量比较小，连接比较稳定的API。

于是，设计了这个基于STM32 MCU的疫情监控平台，STM32通过串口和ESP8266进行AT指令交互，连接互联网获取最新的疫情数据，并显示在LCD显示屏上，可以直观方便的了解到最新的疫情数据信息。

最终效果如下：

![显示效果](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200803/fc1bc0fbff63d61432855c44b942b6c.jpg)

![主板拆分](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200803/fc2bb2435835f77f01f0133a7a240ae.jpg)

#### 开发板的选择

开发板用的是我在大四时自己设计的STM32开发板——NiceDay，基于STM32F103RET主控。前几天看大佬说有学生在大一就自己画板打样了，我感到自愧不如啊！

这是我设计的第二块板子（第一块是毕业设计两轮平衡车主板），是在大四快毕业时，毕设实物和论文完成之后还有点时间，就设计了这款板子，最开始是准备做桌面天气时钟的。

![NiceDay](https://pic2.zhimg.com/80/v2-407d66ab0bac87fa9f88606154c3abb2_720w.jpg?source=1940ef5c)

![NiceDay](https://picb.zhimg.com/80/v2-c9a37ab880e6ee9af678c25578cbf3b8_720w.jpg?source=1940ef5c)

![拆分效果](https://pic3.zhimg.com/80/v2-7709afddd6febb8be3a9e5dd421f7382_720w.jpg?source=1940ef5c)

如果你在百度上搜索：**ESP8266** 关键字，其中就有我当时的一个回答。

![ESP8266](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200803/2020-08-02_234933.jpg)

好了，言归正传，换个API就是疫情监控平台了：

![最终效果](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200803/fc1bc0fbff63d61432855c44b942b6c.jpg)

#### 获取疫情数据API接口

2020新冠疫情的爆发，各大互联网IT公司和个人都开发了实时疫情地图平台，腾讯新闻、丁香园、网易、新浪等等，这些数据大小都在几百KB，对于PC和嵌入式Linux来说，不用在意数据量的大小，但是对于存储非常有限的MCU来说，数据量的大小是不得不考虑的一个问题，而且对于ESP8266来说，AT指令的方式，SSL缓存最大只有4096个字节的缓存！

经过网上一番搜索，找到了几个数据量小的API，但是有的接口连接不稳定，刚连上就掉线了，最后终于找到了一个连接稳定，数据量小，数据齐全的接口：`https://lab.isaaclin.cn/nCoV/zh`

这是一位国人使用服务器爬虫获取了丁香园的数据，然后开放了API接口供大家免费使用，目前已经被调用了2千万次，这个网站还包括了多个接口，我只使用到了其中的疫情数据这一个接口：`https://lab.isaaclin.cn/nCoV/api/overall`，数据量大概为1300个字节。

JSON数据内容如下：

![json数据格式](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200626/2020-06-27_121811.png)

为了能使用ESP8266获取这个API返回的内容，我们还需要知道以下信息：TCP连接类型，端口号，API地址。

我们在浏览器中按F12，打开开发者模式，在地址栏输入`https://lab.isaaclin.cn/nCoV/api/overall`这个接口地址，可以很容易的获取到我们想要的信息：

```C

服务器地址：47.102.117.253
端口号：443
API地址：https://lab.isaaclin.cn/nCoV/api/overall

```

关于端口号，如果API地址是http开头的，一般是选择TCP连接类型，80端口；如果是https开头的，一般是选择SSL连接类型，443端口。这个信息在后面会用到。

![API获取](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200626/2020-06-27_113602.png)

#### ESP8266发送HTTPS请求

WiFi模块选择的是乐鑫的ESP8266-01S模组，支持AP、Station和AP&Station混合模式。

![ESP8266-01S](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200626/ESP8266-01S.png)

在进行正式的开发之前，我们先测试一下使用串口模块连接ESP8266，直接发送AT指令的方式来获取疫情数据。

整体流程是：`配置工作模式 > 连接WiFi > 与服务器建立SSL连接 > 发送GET请求获取数据`

0.为了确保模块保持初始状态，在进行配置之前，先让模块恢复出厂设置：`AT+RESTORE`

```C

AT+RESTORE
    
 ets Jan  8 2013,rst cause:2, boot mode:(3,7)

2nd boot version : 1.5
  SPI Speed      : 40MHz
  SPI Mode       : DIO
  SPI Flash Size & Map: 8Mbit(512KB+512KB)
jump to run user1 @ 1000

ready

```

获取AT固件版本信息：`AT+GMR`

```C

AT+GMR

AT version:1.2.0.0(Jul  1 2016 20:04:45)
SDK version:1.5.4.1(39cb9a32)
Ai-Thinker Technology Co. Ltd.
Dec  2 2016 14:21:16
OK

```

有的AT固件版本不支持HTTPS连接。最新版本的AT固件是支持HTTPS连接的，下载地址：https://docs.ai-thinker.com/_media/esp8266/ai-thinker_esp8266_at_firmware_dout_v1.5.4.1-a_20171130.rar

1.WiFi模块设置为Station模式：`AT+CWMODE=1`

2.配网，连接WiFi：`AT+CWJAP="ssid","password"`

```C

AT+CWMODE=1

OK
AT+CWJAP="stm32_2019_ncov","www.wangchaochao.top"

WIFI CONNECTED
WIFI GOT IP

OK

```

3.设置单连接模式：`AT+CIPMUX=0`

4.设置SSL连接大小：`AT+CIPSSLSIZE=4096`

5.与服务器建立HTTPS/SSL连接：`AT+CIPSTART="SSL","47.102.117.253",443`

6.设置为透传模式：`AT+CIPMODE=1`

7.启动透传：`AT+CIPSEND`

8.发送GET HTTPS请求：`GET https://lab.isaaclin.cn/nCoV/api/overall`

![串口指令交互](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200803/2020-06-27_115925.png)

如果以上都配置正确，会收到服务器返回的数据，也就是我们的想要的疫情数据。

如果SSL连接不断开，一直在透传模式，就可以每隔一段时间GET一次API，这样就可以获取到最新的疫情数据了。

经过多次GET请求测试发现，连接还比较稳定，没有出现掉线的情况，但是由于API的访问限制，不要太频繁的发送GET请求，否则可能会被API开发者把IP封掉。

当然，如果连接断开，就要重新执行建立SSL连接，设置透传模式，开始透传这几个操作。如果要主动断开SSL连接，可以先发送不带回车换行的`+++`退出透传，然后使用`AT+CIPCLOSE`关闭SSL连接。

单独的AT指令测试没问题，那我们就可以使用MCU的串口来自动完成和ESP8266的AT指令交互了。 

![2020-08-02_234442](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200803/2020-08-02_234442.jpg)

#### JSON数据的解析

数据是JSON格式的，解析库使用的是开源小巧的[cJSON库](https://mp.weixin.qq.com/s/_rS_uh1KIrwqqIfRsdSe5A)，只有两个文件，使用起来非常方便。

在进行解析之前，先来分析一下JSON原始数据的格式：`results`键的值是一个数组，数组只有一个JSON对象，获取这个对象对应键的值可以获取到国内现存和新增确诊人数、累计和新增死亡人数，累计和新增治愈人数等数据。

全球疫情数据保存在`globalStatistics`键里，它的值是一个JSON对象，对象仅包含简单的键值对，这些键的值，就是全球疫情数据，其中`updateTime`键的值是更新时间，这是毫秒级UNIX时间戳，可以[转换为标准北京时间](https://mp.weixin.qq.com/s/zYVjSn9PRzjESw23Ki6SRA)。

```JSON

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

先定义了结构体ncov_data，用于存储国内和全球疫情数据：

```C

struct ncov_data{
    long currentConfirmedCount;
    long currentConfirmedIncr;
    long confirmedCount;
    long confirmedIncr;
    long curedCount;
    long curedIncr;
    long seriousCount;
    long seriousIncr;
    long deadCount;
    long deadIncr;
    char updateTime[20];
};

```

对应的解析函数：

```C

uint8_t parse_ncov_data(void)
{
    int ret = 0;
    cJSON *root, *result_arr;
    cJSON *result, *global;
    time_t updateTime;
    struct tm *time;

    //root = cJSON_Parse((const char *)str);   //创建JSON解析对象，返回JSON格式是否正确
    printf("接收到的数据:%d\r\r\n", strlen((const char*)USART2_RX_BUF));	//JSON原始数据
    root = cJSON_Parse((const char*)USART2_RX_BUF);
    
    if (root != 0)
    {
        printf("JSON format ok, start parse!!!\r\n");
        result_arr = cJSON_GetObjectItem(root, "results");
        if(result_arr->type == cJSON_Array)
        {
            printf("result is array\r\n");
            result = cJSON_GetArrayItem(result_arr, 0);
            if(result->type == cJSON_Object)
            {
                printf("result_arr[0] is object\r\n");

                /* china data parse */
                dataChina.currentConfirmedCount = cJSON_GetObjectItem(result, "currentConfirmedCount")->valueint;
                dataChina.currentConfirmedIncr = cJSON_GetObjectItem(result, "currentConfirmedIncr")->valueint;
                dataChina.confirmedCount = cJSON_GetObjectItem(result, "confirmedCount")->valueint;
                dataChina.confirmedIncr = cJSON_GetObjectItem(result, "confirmedIncr")->valueint;
                dataChina.curedCount = cJSON_GetObjectItem(result, "curedCount")->valueint;
                dataChina.curedIncr = cJSON_GetObjectItem(result, "curedIncr")->valueint;
                dataChina.deadCount = cJSON_GetObjectItem(result, "deadCount")->valueint;
                dataChina.deadIncr = cJSON_GetObjectItem(result, "deadIncr")->valueint;

                printf("------------国内疫情-------------\r\n");
                printf("现存确诊:   %5d, 较昨日:%3d\r\n", dataChina.currentConfirmedCount, dataChina.currentConfirmedIncr);
                printf("累计确诊:   %5d, 较昨日:%3d\r\n", dataChina.confirmedCount, dataChina.confirmedIncr);
                printf("累计治愈:   %5d, 较昨日:%3d\r\n", dataChina.curedCount, dataChina.curedIncr);
                printf("累计死亡:   %5d, 较昨日:%3d\r\n", dataChina.deadCount, dataChina.deadIncr);
                printf("现存无症状: %5d, 较昨日:%3d\r\n\r\n", dataChina.seriousCount, dataChina.seriousIncr);

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

                    printf("\r\n**********global ncov data**********\r\n");

                    printf("------------全球疫情-------------\r\n");
                    printf("现存确诊: %8d, 较昨日:%5d\r\n", dataGlobal.currentConfirmedCount, dataGlobal.currentConfirmedIncr);
                    printf("累计确诊: %8d, 较昨日:%5d\r\n", dataGlobal.confirmedCount, dataGlobal.confirmedIncr);
                    printf("累计死亡: %8d, 较昨日:%5d\r\n", dataGlobal.deadCount, dataGlobal.deadIncr);
                    printf("累计治愈: %8d, 较昨日:%5d\r\n\r\n", dataGlobal.curedCount, dataGlobal.curedIncr);

                } else return 1;
                
                /* 毫秒级时间戳转字符串 */
                updateTime = (time_t )(cJSON_GetObjectItem(result, "updateTime")->valuedouble / 1000);
                updateTime += 8 * 60 * 60; /* UTC8校正 */
                time = localtime(&updateTime);
                /* 格式化时间 */
                strftime(dataChina.updateTime, 20, "%m-%d %H:%M", time);
                printf("更新于:%s\r\n", dataChina.updateTime);/* 06-24 11:21 */
            } else return 1;
        } else return 1;
        printf("\r\nparse complete \r\n");
        gui_show_ncov_data(dataChina, dataGlobal);
    }
    else
    {
        printf("JSON format error:%s\r\n", cJSON_GetErrorPtr()); //输出json格式错误信息
        return 1;
    }
    cJSON_Delete(root);

    return ret;
}

```

在调用cJSON_Parse()之后，一定要调用cJSON_Delete()释放内存，否则会造成内存泄露。

如果解析失败，可以把启动文件里的堆栈大小设置大一点：

![2020-08-03_001521](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200803/2020-08-03_001521.jpg)

#### LCD显示

液晶屏使用的是3.2寸 LCD，IL9341驱动芯片，320*240分辨率，16位并口。由于屏幕分辨率比较低，可显示的内容有限，所以只是显示了最基本的几个疫情数据。为了减小程序大小，GUI只实现了基本的画点，画线函数，字符的显示，采用的是部分字符取模，只对程序中用到的汉字和字符进行取模。

为了增强可移植性，程序中并没有使用外置SPI Flash存储整个字库，下面是显示效果：

![显示效果](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200803/75c707407b35a54ffa42ec8e3a160e1.jpg)

#### 待优化和调整

目前只使用到了一个API接口，当然，这个平台也提供其他接口可供使用：

- 最新的疫情新闻

`https://lab.isaaclin.cn//nCoV/api/news`

- 最新的各省市疫情数据

`https://lab.isaaclin.cn//nCoV/api/area?latest=1&province=%E5%8C%97%E4%BA%AC%E5%B8%82`

- 最新的辟谣信息

`https://lab.isaaclin.cn//nCoV/api/rumors`

#### 代码下载

代码已经开源，地址在文末，欢迎大家参与，丰富这个小项目的功能！

GitHub开源地址：`https://github.com/whik/stm32_2019_ncov`

或者关注我的公众号：**电子电路开发学习**（ID: MCU149），在后台回复【**STM32疫情监控**】，我会把工程下载链接发送给你。

如果你手上的硬件和我的一样，只需要修改工程中的`USER\config.h`文件中的WiFi信息，就可以直接使用了。
