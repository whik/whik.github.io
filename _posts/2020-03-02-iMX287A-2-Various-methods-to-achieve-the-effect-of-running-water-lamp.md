---
layout:     post
title:    iMX287A多种方法实现流水灯效果
subtitle:	嵌入式Linux
date:       2020-03-02 12:00:00 +0800
author:     Wang Chao
header-img: img/imx287a.jpg
catalog:    true
tag:
    - ARM
    - Linux
---


### 1.流水灯在电子电路中的地位

记得第一次接触单片机时，还是用的AT89S52单片机，第1个程序就是点亮一个LED，然后再实现LED流水灯的效果。在这个过程中，可以了解整个程序的结构，GPIO的使用，延时的使用等。可以说单片机学习中的点灯，就相当于C语言中的"Hello World"！好的，我们来看一下在ARM Linux下如何控制GPIO。

### 2.硬件电路分析

点灯的根本是控制LED对应GPIO输出高低电平，那要控制哪个GPIO呢？这就需要查看原理图，看LED是连接到了哪个GPIO管脚。

iMX287的扩展板AP-283Demo原理图中，LED的驱动电路：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200303194631145.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

与连接器的连接关系：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020030319511460.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

这里我们把：4个LED的管脚和J8C的４个管脚使用跳线帽短接起来，即

```shell
J8A_1=LED1 -> J8C_1=IO3.26
J8A_2=LED2 -> J8C_2=IO3.22
J8A_3=LED3 -> J8C_3=IO3.20
J8A_4=LED4 -> J8C_4=IO2.7
```

即：

```shell
GPIO3_26 = LED1
GPIO3_22 = LED2
GPIO3_20 = LED3
GPIO2_7  = LED4
```

所以，我们只需要控制这几个GPIO的高低电平，可以控制LED了。

连接方式：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200304090817816.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

### 3.先点个灯吧

iMX28 系列处理器的 IO 端口分为 7 个 BANK，其中 BANK0~4 具有 GPIO 功能，每个BANK 具有 32 个 I/O。iMX283 部分 BANK 的引脚不支持 GPIO 功能，具体需要参考《IMX28CEC_Datasheet.pdf》手册。在导出 GPIO 功能引脚时，需要先计算 GPIO 引脚的排列序号，其序号计算公式:

```shell
GPIO序号 = Bank * 32 + N

#LED对应的序号
LED1 -> GPIO3_26 = 3 * 32 + 26 = 122
LED2 -> GPIO3_22 = 3 * 32 + 22 = 118 
LED3 -> GPIO3_20 = 3 * 32 + 20 = 116
LED4 -> GPIO2_7  = 2 * 32 + 7  = 71
```

串口或SSH登录开发板之后，通过如下过程可导出对应GPIO的功能，以LED1对应的122为例：

```shell
#进入到gpio驱动所在的文件夹
cd /sys/class/gpio

#生成LED1操作的相关文件
echo 122 > export

#进入到LED1控制文件夹
cd gpio122

#设置GPIO为输出方向
echo out > direction

#查看当前GPIO的输入输出方向
cat direction

#设置GPIO输出0点亮LED
echo 0 > value

#查看当前GPIO的输出状态
cat value

#设置GPIO输出1熄灭LED
echo 1 > value

#查看当前GPIO的输出状态
```

在这里，就体现了**Linux系统下一切皆文件**的含义。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200303201350859.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

这样，就实现了某个GPIO输出高低电平的目的，其他LED的控制，只需要修改对应的序号即可。

如果想取消LED1的GPIO功能，可以通过如下命令实现：

```shell
echo 122 > unexport
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200303201820272.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

当然，如果是读取输入，如按键等，就设置方向为in输入，通过`cat value`来读取输入的状态。

### 4.shell脚本实现流水灯

好了，知道了如何实现GPIO的控制，那么如何方便、快捷的执行以上步骤呢？我们这里先介绍一种通过shell脚本的方式控制LED实现流水灯的效果：

```shell
#在home目录新建led_blink.sh脚本文件
touch led_blink.sh

#输入控制LED实现流水灯效果的代码
vi led_blink.sh
```

**led_blink.sh的文件内容：**

```shell
#!/bin/bash

#LED对应的GPIO
led1_pin=122
led2_pin=118
led3_pin=116
led4_pin=71

#echo $led1_pin $led2_pin $led3_pin $led4_pin

#GPIO操作文件如果不存在则创建,否则不创建
for pin in $led1_pin $led2_pin $led3_pin $led4_pin
do
    #如果文件夹存在
    if [ ! -d "/sys/class/gpio/gpio$pin/" ];
        then
            echo $pin > /sys/class/gpio/export
    else
        echo "$pin dir exist!"
    fi
done

#GPIO方向设置为输出
for pin in $led1_pin $led2_pin $led3_pin $led4_pin
do 
    echo out > /sys/class/gpio/gpio$pin/direction
done 

#死循环
while true
do
    #GPIO输出0/1
    for pin in $led1_pin $led2_pin $led3_pin $led4_pin
    do 
        echo "点亮$pin"
        echo 0 > /sys/class/gpio/gpio$pin/value
        sleep 1
        
        echo "熄灭$pin"
        echo 1 > /sys/class/gpio/gpio$pin/value
    done
done
```

保存退出之后，通过如下命令执行这个脚本：

```shell
#添加可执行权限
chmod +x led_blink.sh

#执行脚本文件
./led_blink.sh
```

然后就可以看到流水灯效果了：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200303203032483.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

这种方式，没什么高深的技术，就是把之前通过一行一行的命令，转换成了自动化的脚本文件。为了写这种shell脚本，需要学习一些基本的shell语法。下面我们来介绍两种比较通用的方法，通过C语言文件读写的方式来实现流水灯效果。

### 5.ANSI C文件操作实现流水灯

只使用标准ANSI C文件操作来实现流水灯效果。标准C语言操作，常用的函数有
`fopen/fclose/fwrite/fread/fseek`等。无论是Linux还是Windows都是通用的。

```c
/* ANSI C文件操作实现流水灯 By whik */

//包含printf和sleep等
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define LED1_PIN 122
#define LED2_PIN 118
#define LED3_PIN 116
#define LED4_PIN 71

#define GPIO_PATH "/sys/class/gpio/"

int main(void)
{
    int led_table[4] = {LED1_PIN, LED2_PIN, LED3_PIN, LED4_PIN};
    char path[100];
    char cmd_export[100];
    int i;
    
    char *str;
    FILE *fp;       //文件指针
    int cnt_w;      //写入的字节数

    //判断LED对应的操作文件夹是否存在，不存在自动创建
    for(i = 0; i < 4; i++)
    {
        sprintf(path, "%sgpio%d", GPIO_PATH, led_table[i]); //sys/class/gpio/gpio122

        if (!access(path, 0))
            printf("%s 文件夹存在\n", path);
        else
        {
            printf("%s 文件夹不存在\n", path);
            sprintf(cmd_export, "echo %d > %sexport", led_table[i], GPIO_PATH);//echo 122 > /sys/class/gpio/export
            system(cmd_export); //执行export命令

            if(!access(path, 0))    //访问文件夹,确认创建成功
                printf("%s 导出成功\n", path);
            else 
                return -1;
        }
    }


    //设置输出方向
    for(i = 0; i < 4; i++)
    {
        sprintf(path, "%sgpio%d/direction", GPIO_PATH, led_table[i]);
        fp = fopen(path, "r+");
        if(fp != NULL)      //打开成功
        {
            printf("%s 打开成功\n", path);
            cnt_w = fwrite("out", 3, 1, fp);
            if(cnt_w == EOF)
            {
                printf("方向写入失败\n");
                return -1;
            }
            else 
                printf("方向写入成功\n");
            fclose(fp);     //文件关闭
        }
        else 
        {
            printf("%s 文件打开失败\n", path);
            return -1;
        }
    }

    //流水灯效果
    while(1)
    {
        for(i = 0; i < 4; i++)
        {
            sprintf(path, "%sgpio%d/value", GPIO_PATH, led_table[i]);
            
            fp = fopen(path, "w+");
            if(fp != NULL)
            {
                fwrite("0", 1, 1, fp);
                fseek(fp, 0, SEEK_SET); //重新定位到文件起始
                printf("%s 写入0\n", path);
                sleep(1);

                fwrite("1", 1, 1, fp);
                printf("%s 写入1\n", path);
                fseek(fp, 0, SEEK_SET);

                fclose(fp);
            }
            else 
            {
                printf("%s 文件打开失败\n", path);
            }  
        }
    }

    return 0;
}
```

实现原理和之前的shell脚本也是一样的思路：

- 先判断LED对应的GPIO操作文件是否导出，如果没有导出则执行导出命令，否则不执行。
- 循环的方式，依次设置4个LED的方向为输出
- 循环的方式，控制4个GPIO的输出高低电平，外面是一个死循环

### 6.Linux 系统调用实现流水灯

Linux系统调用，常用的文件IO操作函数有`open/close/read/write/lseek/ulink`等。函数用法也很简单，和标准C语言用法几乎一样，这里不再介绍。

```c
/* Linux 系统函数实现流水灯 */
//涉及到设备操作的,需包含以下头文件
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
//包含close/read/write/lseek等函数
#include <unistd.h>

//包含printf/sprintf函数
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define LED1_PIN 122
#define LED2_PIN 118
#define LED3_PIN 116
#define LED4_PIN 71

#define GPIO_PATH "/sys/class/gpio/"

int main(void)
{

    int led_table[4] = {LED1_PIN, LED2_PIN, LED3_PIN, LED4_PIN};
    char path[100];
    char cmd_export[100];
    int i;
    
    char *str;
    int fd;
    int cnt_w;      //写入的字节数

    //判断LED对应的操作文件夹是否存在，不存在自动创建
    for(i = 0; i < 4; i++)
    {
        sprintf(path, "%sgpio%d", GPIO_PATH, led_table[i]); //sys/class/gpio/gpio122

        if (!access(path, 0))
            printf("%s 文件夹存在\n", path);
        else
        {
            printf("%s 文件夹不存在\n", path);
            sprintf(cmd_export, "echo %d > %sexport", led_table[i], GPIO_PATH);//echo 122 > /sys/class/gpio/export
            system(cmd_export); //执行export命令

            if(!access(path, 0))    //访问文件夹,确认创建成功
                printf("%s 导出成功\n", path);
            else 
                return -1;
        }
    }

    //设置输出方向
    for(i = 0; i < 4; i++)
    {
        sprintf(path, "%sgpio%d/direction", GPIO_PATH, led_table[i]);
        fd = open(path, O_RDWR);    //linux系统函数,失败返回-1
        if(fd != -1)      //打开成功
        {
            printf("%s 打开成功\n", path);
            cnt_w = write(fd, "out", 3);    //linux系统函数
            if(cnt_w <= 0)
            {
                printf("方向写入失败\n");
                return -1;
            }
            else 
                printf("方向写入成功\n");
            close(fd);     //linux系统函数,成功返回0
        }
        else 
        {
            printf("%s 文件打开失败\n", path);
            return -1;
        }
    }

    //流水灯效果
    while(1)
    {
        for(i = 0; i < 4; i++)
        {
            sprintf(path, "%sgpio%d/value", GPIO_PATH, led_table[i]);
            
            fd = open(path, O_RDWR);    //linux系统函数
            if(fd != -1)
            {
                write(fd, "0", 1);
                lseek(fd, 0, SEEK_SET); //重新定位到文件起始
                printf("%s 写入0\n", path);
                sleep(1);

                write(fd, "1", 1);
                printf("%s 写入1\n", path);
                lseek(fd, 0, SEEK_SET);

                close(fd);
            }
            else 
            {
                printf("%s 文件打开失败\n", path);
            }  
        }
    }

    return 0;
}

```

实现过程也很简单，就是对文件的读写，**Linux下一切皆文件**的含义你理解了吗？

> 我的公众号：mcu149
