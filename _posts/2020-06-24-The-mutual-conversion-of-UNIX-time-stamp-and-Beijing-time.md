---
layout:     post
title:    UNIX时间戳和北京时间的相互转换
subtitle:	C语言学习
date:       2020-06-24 22:00:00 +0800
author:     Wang Chao
header-img: img/Code1.jpg
catalog:    true
tag:
    - C语言
---


### 什么是时间戳

准确的说，应该是[unix时间戳](https://baike.baidu.com/item/unix%E6%97%B6%E9%97%B4%E6%88%B3/2078227?fr=aladdin)，是从1970年1月1日（UTC/GMT的午夜）开始所经过的秒数，不考虑闰秒。一个小时表示为UNIX时间戳格式为：3600秒；一天表示为UNIX时间戳为86400秒，闰秒不计算。在很多API接口中，数据的更新时间并不是一个字符串，而是一个长整形数据，如`1593003485`，表示是北京时间`2020-06-24 20:58:05`。

注意这里直接换算出的是北京时间，如果用时间戳直接转换的话，得到的时间UTC/GMT时间，和北京时间相差8个小时，在原始时间戳加上8个小时再进行转换就是北京时间了。大部分时间戳是以秒为单位的，有的时间戳是以毫秒为单位的。

在线转换工具：[北京时间和UNIX时间戳在线转换](https://tool.lu/timestamp/)

![在线转换](https://img-blog.csdnimg.cn/20200624205924132.png)
下面介绍在Keil环境下，或者是C语言环境下，利用`time.h`头文件中的两个函数实现UNIX时间戳和标准北京时间之间的转换方法。

### 头文件time.h介绍

如果使用C库函数进行转换，使用之前先要包含对应的头文件：

```c

#include <time.h>

```

头文件中有一个比较重要的结构体：

```c

/* 时间戳类型，单位为秒，与uint32_t类型一样 */
typedef unsigned int time_t;     

struct tm {
    int tm_sec;   /* 秒钟,范围0-60,偶尔的闰秒 */
    int tm_min;   /* 分钟,范围0-59 */
    int tm_hour;  /* 小时,范围0-23*/
    int tm_mday;  /* 日,范围1-31 */
    int tm_mon;   /* 月份,范围0-11 */
    int tm_year;  /* 年份,自从1900年 */
    int tm_wday;  /* 星期,范围0-6 */
    int tm_yday;  /* 一年的第几天,范围0-365 */
    int tm_isdst; /* 夏令时标志 */
};

```

这里，我们要注意几个时间的修正：

```c

年份自1900算起，转换为实际年份，要+1900
月份范围0-11，转换为实际月份，要+1
星期范围0-6，转换为实际星期，要+1

```

三个函数：

```c

struct tm * localtime(const time_t *);
给定一个毫秒级时间戳，返回时间结构体

time_t mktime(struct tm *);
给定一个初始化完成的时间结构体，返回一个毫秒级时间戳，
转换时不考虑tm结构的tm_wday和tm_yday，仅用tm_mday来决定日期。

size_t strftime(char *strDest, size_t maxsize, const char *format, const struct tm *timeptr);
给定一个时间结构体，格式化输出字符串，格式化字符可参考
https://baike.baidu.com/item/strftime

示例：
char str[100];
strftime(str, 100, "%F %T", time);  /* 2020-07-01 02:16:51 */
strftime(str, 100, "%m-%d %H:%M", time);  /* 06-30 22:16 */
printf("%s\n", str);

```

### UNIX时间戳转北京时间

输入毫秒级时间戳，调用系统函数，把时间戳转换为UTC时间，为了得到北京时间，在转换之前要先加上8个小时的补偿时间：

```c

#include "time.h"
.....
int main(void)
{
    char str[100];
    struct tm *time;
    uint16_t year, yday;
    uint8_t month, day, week, hour, minute, second;
    time_t timestamp = 1592932611;  /*北京时间2020-06-24 01:16:51*/
/*
	几个用于测试的时间戳和北京时间对应
	1592932611 = 2020-06-24 01:16:51(北京时间) 
	1593541011 = 2020-07-01 02:16:51
	1593526611 = 2020-06-30 22:16:51
*/    

    uart_init(115200);	/*printf通过串口输出*/
    /* 北京时间补偿 */
    timestamp += 8*60*60;
	/* 调用系统函数 */
    time = localtime(&timestamp);
	
	year = time->tm_year;   /* 自1900年算起 */
    month = time->tm_mon;   /* 从1月算起，范围0-11 */
    week = time->tm_wday;   /* 从周末算起，范围0-6 */
    yday = time->tm_yday;  /* 从1月1日算起，范围0-365 */
    day = time->tm_mday;    /* 日: 1-31 */
    hour = time->tm_hour;   /* 小时:0-23点,UTC+0时间 */
    minute = time->tm_min;  /* 分钟:0-59 */
    second = time->tm_sec;  /* 0-60，偶尔出现的闰秒 */
    
    /* 时间校正 */
    year += 1900;
    month += 1;
    week += 1;
    
    printf("UNIX时间戳:%d\r\n", timestamp);
    printf("日期:%d-%d-%d 第%d天 星期%d 时间:%d:%d:%d\r\n",
        year, month, day, yday, week, hour, minute, second);
	
	/* 格式化时间字符串 */
    strftime(str, 100, "%F %T", time);  /* 2020-07-01 02:16:51 */
//    strftime(str, 100, "%m-%d %H:%M", time);  /* 06-30 22:16 */
    printf("%s\r\n", str);

    while(1)
    {
        ;
    }
}

```

运行结果：

![运行结果](https://img-blog.csdnimg.cn/20200624213708314.png)

### 北京时间转UNIX时间戳

给定北京时间：`2020-06-24 01:16:51`，输出时间戳`1592932611`，北京时间先转为UTC8时间戳，再去掉8个小时，转为标准的UNIX时间戳。

```c

#include "time.h"
.....

int main(void)
{
    struct tm time;
    time_t timestamp;
    
    /* 2020-6-25 19:11:50 */
    uint16_t str[6] = {2020, 6, 25, 19, 11, 50};
    
    uart_init(115200);	/*printf通过串口输出*/
    
    time.tm_year = str[0] - 1900;	/* 年份修正 */
    time.tm_mon = str[1] - 1;		/* 月份修正 */
    time.tm_mday = str[2];
    time.tm_hour = str[3];
    time.tm_min = str[4];
    time.tm_sec = str[5];
    
    /* 去掉北京时间8个小时 */
    timestamp = mktime(&time) - 8*60*60;	
    /*1593083510 = 2020-6-25 19:11:50*/
    printf("%d\r\n", timestamp);    
  
    while(1)
    {
        ;
    }
}

```

### 写成函数和调用示例

```c

#include "usart.h"
#include "time.h"

/* 定义结构体，时间为北京时间格式 */
typedef struct{
    uint16_t year;
    uint8_t month;
    uint8_t day;
    uint8_t hour;
    uint8_t minute;
    uint8_t second;
}bj_time;

bj_time timestamp_to_bj_time(time_t timestamp);
time_t bj_time_to_timestamp(bj_time time);

int main(void)
{
    time_t timestamp;
    bj_time time;
    
    uart_init(115200);
    
    timestamp = 1593083510;
    printf("%d\r\n", timestamp);
    
    /* 时间戳转北京时间 */
    time = timestamp_to_bj_time(timestamp);
    /* 2020-6-25 19:11:50 */
    printf("%d-%d-%d %d:%d:%d\r\n",
        time.year, time.month, time.day, time.hour, time.minute, time.second);
    
    /* 北京时间转时间戳 */
    timestamp = bj_time_to_timestamp(time);
    printf("%d\r\n", timestamp);
    
    while(1)
    {
        ;
    }
}

bj_time timestamp_to_bj_time(time_t timestamp)
{
    bj_time time;
    
    struct tm *t;
    
    /* 加上8个小时 */
    timestamp += 8*60*60;
    t = localtime(&timestamp);
    
    /* 日期修正 */
    time.year = t->tm_year + 1900;
    time.month = t->tm_mon + 1;
    time.day = t->tm_mday;
    time.hour = t->tm_hour;
    time.minute = t->tm_min;
    time.second = t->tm_sec;
    
    return time;
}

time_t bj_time_to_timestamp(bj_time time)
{
    struct tm t;
    time_t timestamp = 0;
    
    /* 日期修正 */
    t.tm_year = time.year - 1900;
    t.tm_mon = time.month - 1;
    t.tm_mday = time.day;
    t.tm_hour = time.hour;
    t.tm_min = time.minute;
    t.tm_sec = time.second;

    timestamp = mktime(&t) - 8*60*60;
    return timestamp;
}

```

运行结果：

![运行结果](https://img-blog.csdnimg.cn/20200624221843649.png)


