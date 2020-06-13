---
layout:     post
title:    ESP8266两种工作模式数据传输测试
subtitle:	Station和AP模式数据传输 
date:       2020-06-13 16:00:00 +0800
author:     Wang Chao
header-img: img/esp8266.jpg
catalog:    true
tag:
    - ESP8266
---

ESP8266支持3种模式：Station模式、AP模式和Station+AP混合模式。关于这三种模式的区别可以类比我们的手机，当手机连接无线网时，此时手机为Station模式，当手机打开移动热点时，此时手机为AP模式。简单的说就是Station模式就是作为终端，AP模式就是作为路由器。而Station+AP混合模式，就和路由器的无线桥接功能是一样的，既可以连接别的无线网，同时也可以自己作为路由器。

![50536968-file_1487411222023_1a9](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200613/50536968-file_1487411222023_1a9.jpg)

本文分享ESP8266的两种工作模式下的数据传输：Station模式作为TCP客户端、AP模式作为TCP服务器，分别和网络调试助手进行通讯的AT指令配置流程。

AT指令可以由MCU的串口来完成，这样就可以实现两块ESP8266之间进行通讯，电脑和ESP8266的无线控制，手机和ESP8266的无线控制等。

![58410289-file_1487417744056_10109](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200613/58410289-file_1487417744056_10109.png)

### ESP8266作为TCP客户端，电脑作为TCP服务器

ESP8266模块配置为Station模式连接WiFi，电脑也连接同一个WiFi，电脑使用网络调试助手建立一个TCP服务器，指定服务器地址和端口号。

ESP8266作为TCP客户端，和电脑上的网络调试助手进行通讯，或者直接透传。实现的效果是模块发送的数据，电脑可以接收到，电脑发送的数据，模块可以接收到。

1.模块配置为Station模式：`AT+CWMODE=1`
2.配置WiFi信息按照信号强度排序：`AT+CWLAPOPT=1,127`
3.扫描附近的WiFi信息：`AT+CWLAP`


```c
//配置当执行AT+CWLAP指令时，WiFi信息按照信号强度排序
AT+CWLAPOPT=1,15
//1表示按照信号强度排序，15表示WiFi信息只显示加密方式,WiFi名称,信号强度,MAC地址

//扫描附近的WiFi信息
AT+CWLAP
+CWLAP:([加密方式],[WiFi名称],[RSSI信号强度],[MAC地址])
+CWLAP:(4,"Tenda_A3AA00",-76,"c8:3a:35:a3:aa:01")
+CWLAP:(4,"Tenda_A3AA00 Sander",-81,"e4:d3:32:9c:e3:c4")
+CWLAP:(3,"EZVIZ_D38296744",-81,"50:13:95:84:e0:16")
+CWLAP:(4,"TP-LINK_4723",-84,"cc:08:fb:c1:47:23")
```


4.连接指定WiFi：`AT+CWJAP="Tenda_A3AA00","password123"`

```C
//连接指定AP
AT+CWJAP="Tenda_A3AA00","password123"

//如果WiFi名称重复，需要指定MAC地址来确定要连接的WiFi
AT+CWJAP="Tenda_A3AA00","password123","c8:3a:35:a3:aa:01"

//如果WiFi名称或密码中含有特殊字符，前面要添加\转义符号
如，目标WiFi名称为: ab\,c，密码为: 0123456789"\，则指令如下：
AT+CWJAP="ab\\\,c","0123456789\"\\"

//查询已经连接的WiFi信息
AT+CWJAP?

//断开当前WiFi连接
AT+CWQAP
```

5.设置单连接模式：` AT+CIPMUX=0 `

```C
//如果之前使用AP模式开启过TCP服务器，要先关闭TCP服务器
AT+CIPSERVER=0

//设置单连接模式
AT+CIPMUX=0
```

6.电脑和模块连接同一WiFi，电脑启动网络调试助手，并建立TCP服务器。

![2020-06-13_122832](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200613/2020-06-13_122832.jpg)

7.模块作为TCP客户端，连接电脑上创建的TCP服务器

```C
//主机地址和端口要和电脑上的TCP服务器保持一致，
AT+CIPSTART="TCP","192.168.43.140",6000
CONNECT

OK
```

8.如果连接成功，网络调试助手会显示有一个客户端上线，并显示了客户端的IP为`192.168.1.105`

![2020-06-13_123534](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200613/2020-06-13_123534.jpg)

9.查询当前TCP服务器分配的IP地址：`AT+CIPSTATUS`

```C
AT+CIPSTATUS

STATUS:3		//3表示已经建立TCP传输
+CIPSTATUS:0,"TCP","192.168.1.106",6000,26441,0	//本地IP地址

OK
```

10.此时网络调试助手（TCP服务器）发送的信息，WiFi模块（TCP客户端）已经可以实时收到了。

```C
+IPD,[数据长度]:[数据类型]
+IPD,30:Hello World —— By TCP Server
+IPD,28:MyBlog：www.wangchaochao.top
+IPD,16:MyWeChat：mcu149
```

![2020-06-13_124239](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200613/2020-06-13_124239.jpg)

11.客户端发送数据到服务器。虽然服务器发送的数据客户端可以收到，但此时模块还处于AT模式，不能发送数据到服务器。

```c
//设置本次要发送的字节数
AT+CIPSEND=4
OK
>

//输入要发送的数据，仅前四个字节数据被发出，其他数据无效。    
Recv 4 bytes
SEND OK
```

![2020-06-13_125206](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200613/2020-06-13_125206.jpg)

以上命令只能启动一次数据发送，如果需要数据实时收发，就需要配置成透传模式。

12.开启透传模式。

```c
//开启透传模式，仅支持TCP单连接和UDP固定通信对端的情况
AT+CIPMODE=1

//开始透传
AT+CIPSEND
>
//此时发送的数据会直接给TCP服务器
```

![2020-06-13_125841](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200613/2020-06-13_125841.jpg)

13.退出透传模式。

```c
//输入不带回车换行的三个加号：+++，退出透传模式，返回到普通AT指令模式。
+++
//发送+++退出透传时，请至少间隔1秒再发下⼀条AT指令。
AT

OK
```

14.断开TCP连接。上面虽然退出了透传模式，此时还保持着TCP连接，服务器发送的数据可以实时收到。如果要断开TCP连接可以使用：`AT+CIPCLOSE`，可以看到服务器也显示客户端已经离线。

![2020-06-13_130402](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200613/2020-06-13_130402.jpg)



### ESP8266作为TCP服务器，电脑作为TCP客户端

ESP8266配置成AP模式，并开启TCP服务器，电脑连接ESP8266的WiFi，作为TCP客户端，两者之间数据传输。

1.模块配置成AP模式：`AT+CWMODE=2`
2.设置无线网名称和密码

```c
//设置无线网名称和密码
AT+CWSAP="ESP8266","12345678",5,3
//3表示WPA2_PSK加密方式

//查询无线网信息
AT+CWSAP?

+CWSAP:[WiFi名称],[WiFi密码],[通道数],[加密方式],[最大支持连接数],[广播]
+CWSAP:"ESP8266","12345678",5,3,4,0
```

3.设置无线网IP地址、网关、子网掩码

```c
//设置IP、网关、子网掩码
AT+CIPAP="192.168.5.1","192.168.5.1","255.255.255.0"
```

4.建立TCP服务器，设置端口号
```c
//使用多连接模式
AT+CIPMUX=1
OK

//指定TCP服务器端口为1001
AT+CIPSERVER=1,1001
创建TCP服务器之后，会自动启动TCP服务监听，当有TCP客户端连接时，会有CONNECT提示

0,CONNECT
```

![2020-06-13_144430](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200613/2020-06-13_144430.jpg)

5.网络调试助手配置成客户端模式，连接ESP8266创建的TCP服务器，主机地址和端口要和之前配置的保持一致。

```c
//ESP8266查询当前连接的客户端
AT+CWLIF

[IP地址],[MAC地址]
192.168.5.2,b8:86:87:4e:26:af

OK
```

6.网络调试助手（TCP客户端）发送消息给ESP8266（TCP服务器），因为ESP8266已经开启监听服务，数据会实时显示。

![2020-06-13_144621](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200613/2020-06-13_144621.jpg)

7.ESP8266（TCP服务器）发送消息给网络调试助手（TCP客户端）。

```c
//ESP8266作为服务器，要往客户端发数据，需要指定客户端编号和字节数
//往0号客户端发5个字节的数据
AT+CIPSEND=0,5
OK
>

//输入要发送的数据，仅前五个字节数据被发出，其他数据无效。    
Recv 5 bytes
SEND OK

```

![2020-06-13_151518](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200613/2020-06-13_151518.jpg)
