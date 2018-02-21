---
layout:     post
title:      ESP8266 + Airkiss从零开始实现微信配网（三）
subtitle:   编写单片机程序响应按键、与WiFi模块通讯
date:       2017-02-10 14:50:37 +0800
author:     Shao Guoji
header-img: img/post-bg-wechat-airkiss3.jpg
catalog:    true
tag:
    - ESP8266
    - 物联网
    - 单片机
---

系列文章目录

[ESP8266 + Airkiss从零开始实现微信配网（一） - 目的、背景及准备工作]({% post_url 2017-01-16-ESP8266-wechat-onekey-config-1 %})

[ESP8266 + Airkiss从零开始实现微信配网（二） - 微信公众号开发、JSSDK、JSAPI的使用]({% post_url 2017-01-28-ESP8266-wechat-onekey-config-2 %})

[ESP8266 + Airkiss从零开始实现微信配网（三） - 编写单片机程序与WiFi模块通讯]({% post_url 2017-02-10-ESP8266-wechat-onekey-config-3 %})

---

<br/>

![一键配网电路](http://odaps2f9v.bkt.clouddn.com/17-3-11/10934514-file_1489166914623_2969.jpg)

### 预备知识

* 基础的C语言与51单片机知识
* 51单片机串口、外部中断的知识。
* 在Keil环境下编写、编译、并用STC-ISP下载单片机程序的能力。
* WiFi模块AT指令的使用（可参考文章[ESP8266串口WiFi模块的基本使用]({% post_url 2017-01-15-ESP8266-usage %})）

### 所需材料

| 名称                              | 数量 |
|-----------------------------------|------|
|面包板及连接线（或洞洞板焊接）     |  1   |
|USB-TTL串口下载模块                |  1   |
|STC12C5A60S2单片机                 |  1   |
|ESP8266-01串口WiFi模块             |  1   |
|11.0592MHz晶振                     |  1   |
|30pf瓷片电容（单片机起振）         |  2   |
|104瓷片电容（按键去抖）            |  1   |
|按键（微动开关）                   |  1   |
|5mm LED发光二极管                  |  1   |

### 内容简介

从上一篇最后的测试部分我们了解到，要想让WiFi模块进入Airkiss模式，就要通过电脑串口用sscom软件发送AT指令。串口的话单片机也有啊，我们最终目的是“一键配网”,那再接个按键岂不是成了！下面就来看看如何通过C语言程序控制单片机串口和处理按键实现“一键配网”，以及进一步的“一键透传”功能。

---

<br/>

### AT指令交互的过程

其实上一篇我们已经实现了微信配网了，只不过是“手动微信配网”。而我们最终要做的是“一键自动配网”，那就必须把**“在sscom给WiFi模块发送一堆AT指令”**这个动作变为**“按一个按键”**，也就是按下按键后由单片机给模块发AT指令。

首先让我们回顾一下手动使用AT指令进行智能配置的过程（ `\r\n` 为实际发送指令末的回车换行）：

1. 发送指令 `AT\r\n` 测试模块是否正常响应

2. 发送 `AT+CWMODE=1\r\n` 指令配置模块为sta模式

3. 发送 `AT+CWSTARTSMART\r\n` 开启模块的“SmartConfig”功能 

4. 手机发送WiFi名称和密码进行配网

5. 发送 `AT+CWSTOPSMART\r\n` 停止 SmartConfig

在实际使用中配网只是基础，联网往往是为了实现更多的功能。这里以“一键透传”为例：按下按键后，WiFi模块通过智能配置连网、建立TCP连接、保存连接到flash，然后进入透传模式，下次上电自动连接TCP服务器进入透传模式——所有的这些操作都会在按下按键后，由单片机向模块发送AT指令实现。

交互应该是双方的，**如果单片机只是“一厢情愿”地向模块发送 AT 指令，而不顾模块的“感受”是不行的**。在使用sscom手动发送AT指令时可以留意到模块会返回结果信息，单片机必须处理这些结果信息，根据命令的反馈结果进行下一步设置。

还有一个问题，要怎么验证WiFi模块配网成功了呢？不妨考虑让模块在与服务器建立TCP连接后，主动发送一条字符串消息，这样一来就知道模块配网成功了。

对于不需要经常改变的模块参数，如无线模式、TCP服务器IP端口号和串口波特率等，可以先sscom手动发送指令设置。此外，模块平时是工作在透传模式下，发送AT指令前需要退出透传模式。完整的流程如下：

1. 发送 `+++` 退出透传模式，模块返回 `CLOSED\r\n` 

2. 发送指令 `AT\r\n` 检查模块是否正常响应，模块返回 `OK\r\n` 

3. 发送 `AT+CWSTARTSMART\r\n` 开启模块的“SmartConfig”功能，模块返回 `OK\r\n` 

4. 手机发送WiFi名称和密码进行配网，模块返回一大串信息……

5. 发送 `AT+CWSTOPSMART\r\n` 停止 SmartConfig，模块返回 `OK\r\n` 

6. 发送 `AT+CIPMUX=0\r\n` 设置单连接，模块返回 `OK\r\n` 

7. 发送 `AT+CIPSTART="TCP","服务器IP",端口号\r\n` 与服务器建立TCP连接，模块返回 `OK\r\n` 

8. 发送 `AT+CIPMODE=1` 开启透传模式，模块返回 `OK\r\n` 

9. 发送 `AT+SAVETRANSLINK=1,"服务器IP",端口号,"TCP"\r\n` 保存TCP连接到flash，模块返回 `OK\r\n` 

10. 发送 `AT+CIPSEND\r\n` 进入透传模式

11. 向服务器发送提示信息 `Connected successfully`

弄清了发送AT指令的顺序（具体可根据实际情况修改），事情就基本完成了，接下来只是编程把流程实现而已。

### 电路连接

为简单起见，硬件电路使用面包板搭接，电路仅有少量的元器件，包括单片机、ESP8266模块、电容、晶振、按键、LED。

![电路连接图](http://odaps2f9v.bkt.clouddn.com/17-3-11/10000953-file_1489164371921_e816.png)

LED 起到指示作用，晶振能保证串口波特率的精确。需要注意的一点是，**由于使用外部中断处理按键，引脚的电位改变是直接由硬件处理，故无法用软件进行按键去抖，所以只能用并联电容的方法去抖**（更多关于按键去抖知识请看文章[浅谈如何按键消抖](http://www.eeworld.com.cn/mcu/2012/0806/article_9776.html)）。

### 程序编写

打开Keil，新建工程，芯片选Atmel的AT89C55即可，**在工程设置的“输出”页勾选“产生HEX文件”选项**，新建C源码文件并保存添加到工程，开始写代码！

#### 头文件及全局变量定义

```C
#include <STC12C5A60S2.h>   //STC12C5AxxS2系列单片机头文件
#include <string.h>  // 字符串处理头文件

// 配网指示灯
sbit LED = P0 ^ 2;  // 对应硬件连接

// 串口wifi命令结果处理
unsigned char wifiDataBuf[100];  // 串口数据缓存
unsigned char wifiIndex = 0;  // 数值下标
```

#### 毫秒级延时函数

```C
void DELAY_MS (unsigned int a){
    unsigned int i;
    while( a-- != 0){
        for(i = 0; i < 600; i++);
    }
}
```

#### UART串口初始化函数

```C
void UART_init (void){
    EA = 1; //允许总中断（如不使用中断，可用//屏蔽）
    ES = 1; //允许UART串口的中断

    TMOD = 0x20;    //定时器T/C1工作方式2
    SCON = 0x50;    //串口工作方式1，允许串口接收（SCON = 0x40 时禁止串口接收）
    TH1 = 0xFA;     //定时器初值高8位设置
    TL1 = 0xFA;     //定时器初值低8位设置
    PCON = 0x80;    //波特率倍频（屏蔽本句波特率为4800）
    
    IPH |= 0x10;    // 设置串口1中断优先级为最高，很重要！！！
    PS = 1;

    TR1 = 1;        //定时器启动    
}
```

使用单片机的串口1，设置为工作方式1，波特率由定时器1溢出率决定。配置定时器为模式2——8位自动重装模式，并设置所需波特率对于的初值（详细信息请查阅[STC12C5A60S2数据手册](http://www.doyoung.net/DOC/STC12C5A60S2.pdf)）。波特率与初值的关系可查看下表：

![单片机常用波特率初值表](http://odaps2f9v.bkt.clouddn.com/17-3-10/86973906-file_1489116592716_925f.jpg)

#### 关于中断优先级

**要特别注意中断优先级的问题！！！**要想实现在按键外部中断服务程序内处理串口中断，就必须把串口中断设置为最高优先级，否则按下按键后串口中断会被屏蔽，接收不到模块的反馈信息。这个坑我踩了两次了啊，开始搞的时候不知道，现在写文章又忘了！

到底怎么个回事？数据手册写的很清楚：

> 中断的两条基本原则：
> 
> 1、低优先级中断可被高优先级中断所中断，反之不能。 
> 
> 2、任何一种中断（不管是高级还是低级），一旦得到相应，不会再被它的同级中断所中断。

> STC12C5A60S2系列单片机复位后IP、IP2、IPH和IP2H均为00H，各个中断源均为低优先级中断。

也就是说各个中断默认都是同级的，但只有等级高的中断才能打断等级低的中断，因此如果不手动修改中断优先级的话是不能实现中断嵌套的（即在中断服务程序中再被中断）。根据数据手册寄存器定义，用 `IPH |= 0x10;` 和 `PS = 1;` 两条语句把串口1中断设置为最高优先级。

![数据手册关于中断优先级](http://odaps2f9v.bkt.clouddn.com/17-3-10/24926947-file_1489136090967_aea3.png)

#### UART串口字符、字符串发送函数

```C
void UART_T (unsigned char UART_data){ //定义串口发送数据变量
    SBUF = UART_data;   //将接收的数据发送回去
    while(TI == 0);     //检查发送中断标志位
    TI = 0;         //令发送中断标志位为0（软件清零）
}

void UART_TC (unsigned char *str){
    while(*str != '\0'){
        UART_T(*str);
        *str++;
    }
    *str = 0;
}
```

我们知道AT指令就是字符串，为了简化发送字符和字符串，对串口底层寄存器操作进行封装，使用时直接调用即可。

#### UART串口接收中断处理函数

```C
void UART_R(void) interrupt 4  using 1{ //切换寄存器组到1

    unsigned char UART_data; //定义串口接收数据变量

    if (RI) // 不判断RI有可能出问题
    {
        RI = 0;         //令接收中断标志位为0（软件清零）
        UART_data = SBUF;   //将接收到的数据送入变量 UART_data

        wifiDataBuf[wifiIndex++] = UART_data;
        
        if (UART_data == 0x0A) // \n命令结尾，下标指向开始，为接收下一条命令做准备
        {
            wifiDataBuf[wifiIndex] = '\0'; // 补上字符串结尾'\0'
            wifiIndex = 0;  
        }
    }

}
```

串口在收到数据后由中断程序处理，把收到的数据逐字节存入缓存数组中，用于接收模块返回的指令结果。从之前使用AT指令的经验可知，AT指令都是以 `\r\n` 结尾，所以我们可以**通过判断 `\n` 来确定一条指令的结束**。接收完一条指令后数组下标归零再存储下一条指令，保证 `wifiDataBuf` 数组内的是当前最新接收到的数据。

#### 等待串口命令结果函数

```C
void UART_WAIT_FOR(unsigned char* str, int timeout)
{
    int t = 0;
    while (t < timeout)
    {
        t++;
        DELAY_MS(1);
        if (strcmp(wifiDataBuf, str) == 0)
        {
            // UART_TC("Hello World!\r\n");
            wifiDataBuf[0] = '\0';
            break;
        }
    }           
}
```

单片机需要等待WIFi模块对AT指令的应答消息（如 `OK\r\n`），但要等多久呢？万一模块出现问题不响应又怎么办？这是个问题。而这个函数就完美解决了以上两个问题，**让程序在收到应答后立即向下执行，不浪费一丁点时间。在指定的超时时间后若WiFi模块无回应也让程序向下执行，避免“永远的等待”**。使用 `UART_WAIT_FOR(command, timeout)` 调用，传入期待的字符串和超时时间（好像看过Arduino上类似的实现）。

#### INT0外部中断处理函数(一键配网)

```C
void exint0() interrupt 0           //(location at 0003H)
{

    UART_TC("+++"); // 退出透传模式
    UART_WAIT_FOR("CLOSED\r\n", 1000);

    UART_TC("AT\r\n");  // AT指令测试
    UART_WAIT_FOR("OK\r\n", 1000);

    UART_TC("AT+CWSTARTSMART\r\n"); // 开始智能配网模式
    UART_WAIT_FOR("OK\r\n", 1000);
    LED = 0; // 配网指示灯亮起
    UART_WAIT_FOR("smartconfig connected wifi\r\n", 30000); // 链接成功

    UART_TC("AT+CWSTOPSMART\r\n"); // 结束智能配网模式，释放模块资源(必须)
    UART_WAIT_FOR("OK\r\n", 1000);
    LED = 1; // 配网指示灯熄灭

    UART_TC("AT+CIPMUX=0\r\n");  // 设置单连接模式
    UART_WAIT_FOR("OK\r\n", 1000);

    UART_TC("AT+CIPSTART=\"TCP\",\"192.168.1.147\",1234\r\n");  // 连接到指定TCP服务器

    UART_TC("AT+CIPMODE=1\r\n"); // 设置透传模式
    UART_WAIT_FOR("OK\r\n", 1000);

    UART_TC("AT+SAVETRANSLINK=1,\"192.168.1.147\",1234,\"TCP\"\r\n"); // 保存TCP连接到flash，实现上电透传

    UART_TC("AT+CIPSEND\r\n");   // 进入透传模式
    UART_WAIT_FOR("OK\r\n", 1000);

    UART_TC("Connected successfully"); // 向服务器发送上线消息
}
```

外部中断处理函数决定了按下按键产生中断后要做的事情，对于我们来说就是要**给WiFi模块发送AT指令，并处理模块返回提示信息**，所以这一堆串口收发操作是“一键配网”的核心。只需严格按照文章前面分析的流程来走就行了，不过也有几点需要特别注意。

首先，在用 `AT+CWSTARTSMART\r\n` 开启智能配置模式后模块会返回 `OK\r\n` ，这时就要点亮指示灯提醒用户输入密码（对应网页的提示信息），之后等待手机端的微信配网操作，配网过程中，当用户发送密码后模块会打印以下信息：

```
smartconfig type:AIRKISS
Smart get wifi info
ssid:WiFi名称
password:WiFi密码
WIFI CONNECTED
WIFI GOT IP
smartconfig connected wifi
```

注意到最后会打印一条 `smartconfig connected wifi` ，而我们就可以根据这个标志信息判断配网已经完成，这时就可以退出智能配置、熄灭指示灯进行下一步操作了。用户在手机进行配网操作需要花费较长的时间，所以 `smartconfig connected wifi` 的等待超时设了30秒。

另外，**在发起和保存TCP连接的 AT 指令中包含双引号，要注意字符转义，用 `\"` 表示双引号**。

### 主函数

```C
void main (void){

    // 串口初始化
    UART_init();

    // 外部中断设置
    IT0 = 1;                        // 设置外部中断INT0类型 (1:下降沿 0:低电平)
    EX0 = 1;                        // 开启INT0中断

    while(1)
    {

    }
}
```

主函数只要初始化串口和中断，再加个万年不变 `while(1)` ，没有其他内容。

#### 测试单片机程序

先不接WiFi模块，把写好的程序下载到单片机中，保持与电脑的串口连接，连好按键和LED，打开sscom，设置波特率为9600,后打开串口。这时按下按键，会先收到单片机发来的 `+++` ，然后打印开启智能配置的指令 `AT+CWSTARTSMART\r\n` ，指示灯亮起，由外部中断源码我们知道，此时程序在等待配网成功的 `smartconfig connected wifi` 字符串，我们可以手动发送这条字符串模拟WiFi模块的返回，让程序立即往下执行、打印其余指令。是的，此时你就是一个WiFi模块，在与单片机进行交互！

![模拟WiFi模块返回](http://odaps2f9v.bkt.clouddn.com/17-3-10/27847518-file_1489137247621_169e4.png)

### 软硬联调、见证奇迹

确保程序正常运行后，最后再把WiFi模块接上就大功告成了。在连接模块之前，先用sscom配置一些WiFi模块的基本参数：

* 发送 `AT+CWMODE=1\r\n` 指令配置模块为sta模式
* 发送 `AT+UART=9600,8,1,0,0` 配置模块波特率为9600（与单片机匹配）

![sscom设置常用参数](http://odaps2f9v.bkt.clouddn.com/17-3-11/28498093-file_1489164352896_14c0.png)

把面包板电路连接完整，打开“网络调试助手”配置TCP服务器（可参考文章[ESP8266串口WiFi模块的基本使用]({% post_url 2017-01-15-ESP8266-usage %})），确保单片机程序中TCP指令的服务器地址与“网络调试助手”中的本机地址相同。上电，按下按键指示灯亮起后在手机输入WiFi密码（手机和电脑连接同一WiFi），配网成功后指示灯熄灭。在“网络调试助手”客户端列表中可看到模块连接到TCP服务器，并打印出了上线信息“Connected successfully”。拔掉模块电源重新上电，仍然会自动连接服务器并进入透传，这就是微信一键配网的所有流程。

![服务器收到连接](http://odaps2f9v.bkt.clouddn.com/17-3-11/71901224-file_1489163728810_bb91.png)

当然实际使用中服务器IP不应该写死，可通过串口设置才好，而且仅仅实现透传好像也没什么卵用，但本文只是配网，其他功能不做深入。

源码下载：[一键配网 密码kd09](http://pan.baidu.com/s/1jI0rNp0)

### 大总结

我的实现只是提供一种可参考的思路，具体使用的技术可根据自己的情况选择，比如你Java学的比较6，那微信网页后台可以用Java，单片机也可以用其他的。总之只要把大体的思路弄清楚，细节可自由发挥，能解决实际问题、把功能实现才是硬道理嘛。

说实话写这几篇文章非常折磨人。都说能把知识给别人讲懂才算真正理解，要写出来就更考验文字表达能力了。很多句子刚写出来很不通顺，要反复修改，常见的问题就是无逻辑的话语拼凑，只有自己能读懂，又或者使用多余连词、说太多反而有碍于理解废话……文章没啥技术深度，但写起来是一拖再拖，现在终于要接近尾声。有时候觉得会不会把废话写的太细，而重点却一带而过？或许真的太啰嗦了，连自己都烦了。

---

<br/>
<br/>

>参考文章： 
> 
> * [浅谈如何按键消抖](http://www.eeworld.com.cn/mcu/2012/0806/article_9776.html)
> * [ESP8266 AT指令集](http://espressif.com/sites/default/files/documentation/4a-esp8266_at_instruction_set_cn.pdf)
> * [STC12C5A60S2数据手册](http://www.doyoung.net/DOC/STC12C5A60S2.pdf)






