---
layout:     post
title:      玩的就是心跳 —— 使用 PulseSensor 脉搏传感器测量心率
subtitle:	  STM32 ADC 模块实战
date:       2017-6-29
author:     Shao Guoji
header-img: img/post-bg-pulsesensor.jpg
catalog:    true
tag:
    - 单片机
    - STM32
    - 嵌入式
---

![图1 心率测量][1]

### 内容简介

对于 arduino 来说，网上有大量关于 PulseSensor 脉搏传感器的资料，而其他单片机上的实例较少。本文使用 STM32F407 系列芯片的 ADC 模块，**从硬件配置、简易心率算法编写到官方 Processing 上位机的使用，带你全方位玩转  PulseSensor，玩的，就是心跳！**
  
  ---

<br/>
  
###   PulseSensor 脉搏传感器介绍

#### 基本参数

| 供电电压：     | 3.3~5V             |
| -------------- | ------------------ |
| 检测信号类型： | 光反射信号（PPG） |
| 输出信号类型： | 模拟信号           |
| 输出信号大小： | 0~VCC              |
| 电流大小：     | ~4ma（5v 下）      |

#### 功能原理

PulseSensor 是一款用于脉搏心率测量的光电反射式模拟传感器。将其佩戴于手指、耳垂等处，利用人体组织在血管搏动时造成透光率不同来进行脉搏测量。传感器对光电信号进行滤波、放大，最终输出模拟电压值。**单片机通过将采集到的模拟信号值转换为数字信号，再通过简单计算就可以得到心率数值**。

![图2 PulseSensor 脉搏传感器][2]

PulseSensor 是一款开源硬件，目前[国外官网][3]上已有其对应的[开源 arduino 程序][4]和[上位机 Processing 程序][5]，其适用于心率方面的科学研究和教学演示，也非常适合用于二次开发。 网上关于传感器的  arduino 资料已经十分丰富（毕竟同为开源硬件），本文采用 STM32F407系列芯片 的 ADC 模块读取并处理传感器数据，实现心率测量。

#### 引脚定义

![图3 传感器引脚定义][6]

传感器只有三个引脚，分别为信号输出 S 脚 、电源正极 VCC 以及电源负极 GND，供电电压为 3.3V - 5V，可通过杜邦线与开发板连接。上电后， 传感器会不断从 S 脚输出采集到的电压模拟值。**需要注意的是，印有心形的一面才是与手指接触面，在测量时要避免接触布满元件的另一面，否则会影响信号准确性。**

  ---

<br/>

### 读取传感器电压值 —— STM32 ADC 功能配置

#### 硬件配置

开发板使用的是公司的 M4 板子，传感器 3.3V 供电，信号采集选用 ADC1 的 通道 2，硬件连接如下：

| 开发板 | 传感器 |
| ------ | ------ |
| PA2    | S      |
| 3V3    | +      |
| GND    |  -     |


把 PA2 用作模拟功能，配置 ADC 为 12 位分辨率，单次转换，并设置转换序列长度为 1，首次转换通道 2。为确保数据准确性，选择APB2 时钟 6 分频作为 ADC 时钟（即 84M / 6 = 14M），采样时间 480 个周期（使得采样时间更加充分），最后使能 ADC。初始化函数如下：

```c
/******************** ADC通道2初始化函数 ************************/
void ADC_AN2_Init(void)
{
    /* 设置ADC功能对应的GPIO端口 */
    RCC->AHB1ENR |= 1 << 0;
    GPIOA->MODER &= ~(3 << (2 * 2));
    GPIOA->MODER |= 3 << (2 * 2);
    
    /* 配置ADC采样模式 */
    RCC->APB2ENR |= 1 << 8;     //使能ADC模块时钟
    ADC1->CR1 &= ~(3 << 24);    //12位分辨率
    ADC1->CR1 &= ~(1 << 8);     //非扫描模式
    ADC1->CR2 &= ~(3 << 28);    //禁止外部触发
    ADC1->CR2 &= ~(1 << 11);    //右对齐
    ADC1->CR2 &= ~(1 << 1);     //单次转换
    ADC->CCR &= ~(3 << 16);
    ADC->CCR |= 2 << 16;        //6分频
    ADC1->SMPR2 &= ~(0x07 << 6);
    ADC1->SMPR2 |= 0x07 << 6;   //480采样周期
    ADC1->SQR1 &= ~(0x0f << 20); //1次转换
    ADC1->SQR3 &= ~(0x1f << 0);
    ADC1->SQR3 |= 0x02 << 0;     //转换的通道为通道2
    
    /* 使能ADC */
    ADC1->CR2 |= 1 << 0;          //开启ADC
}
```

编写好初始化函数后还需要写一个进行 AD 转换的函数，这也是我们功能的核心。思路是通过将十次 AD 转换值进行冒泡排序，然后掐头去尾求平均值作为最后的转换输出值，程序如下：

```c
/******************** ADC通道2转换函数 ************************/
u16 Get_ADC_1_CH2(void)
{
    u8 i,j;
    u16 buff[10] = {0};
    u16 temp;
    
    for(i = 0; i < 10; i ++)
    {
        /* 开始转换 */
        ADC1->CR2 |= 1 << 30;
		
        /* 等待转换结束 */
        while( !(ADC1->SR & (1 << 1)) )
        {
            /* 等待转换接收 */
        }
        buff[i] = ADC1->DR;    //读取转换结果
    }
    
    /* 把读取值的数据按从小到大的排列 */
    for(i = 0; i < 9; i++)
    {
        for(j = 0 ; j < 9-i; j++)
        {
            if(buff[j] > buff[j+1])
            {
                temp = buff[j];
                buff[j] = buff[j+1];
                buff[j+1] = temp;
            }
        }
    }
    
    /* 求平均值 */
    temp = 0;
    for(i = 1; i < 9; i++)
    {
        temp += buff[i];
    }
    
    return temp/8;
}
```

#### 串口打印，验证数据读取

是驴是马得拉出来溜溜，配好的 ADC 能不能用也要经过检验。方法是把从传感器读到的转换值在串口打印，以此测试 ADC 转换是否工作正常。为了模拟波形的效果，编写如下波形打印函数 —— 将读出来的数据缩小适当倍数后，用同一行的星号数量来表示。

```c
void Print_Wave(void)
{
    int temp, i;
    
    temp = Get_ADC_1_CH2() / 20;   // 缩小一个倍数
    
    for (i = 0; i < temp; i++)
        printf("*");
    
    printf ("\r\n");
}
```

在主函数的 `while (1)` 循环中不断调用 `Print_Wave()` 函数在串口打印输出，每次打印延时一段时间，代码如下：

```c
int main(void)
{
    Usrat_1_Init(84,115200,0);
    Timer_6_Init();
    ADC_AN2_Init();
    
    while(1)
    {
        Print_Wave();
        Timer_6_Delay_ms(5);  // 延时 5 ms
    }
}
```

*注：由于本文重点是 ADC，所以不对串口与定时器的具体配置展开说明，有需要请查看源代码相应部分程序。*

把开发板连接电脑，下载程序后打开串口工具接收数据，通过对传感器测量面绿光的遮挡，可在串口看到用字符打印的波形，波峰波谷清晰可见，并不断波动，证明 ADC 读取到了传感器输出的模拟电压信号。效果如下图：

![图4 串口字符波形][7]

  ---

<br/>

### 计算心率值 —— 采样数据处理算法

心率指的是一分钟内的心跳次数，得到心率最笨的方法就是计时一分钟后数有多少次脉搏。但这样的话每次测心率都要等上个一分钟才有一次结果，效率极低。另外一种方法是，**测量相邻两次脉搏的时间间隔，再用一分钟除以这个间隔得出心率**。这样的好处是可以实时计算脉搏，效率高。由此引出了 `IBI` 和 `BPM` 两个值的概念：

**IBI： 相邻两次脉搏的时间间隔（单位：ms）
BPM(beats per minute)：心率，一分钟内的心跳次数**

**且：BPM = 60 / IBI**

从网上找来的 arduino 开源算法复杂的一匹，看了一遍感觉一头雾水（反正我暂时没看懂）。由上面的分析可以得出，**我们的最终目的就是要求出 IBI 的值，并通过 IBI 计算出实时心率**。既然知道原理了那就自己来把算法实现吧。

#### 核心操作 —— 识别一个脉搏信号

无论是采用计数法还是计时法，只有能识别出一个脉搏，才能数出一分钟内脉搏数或者计算两个相邻脉搏之间的时间间隔。那怎么从采集的电压波形数据判断是不是一个有效的脉搏呢？

显然，可以**通过检测波峰来识别脉搏**。最简单粗暴的方法是设定一个阈值，当读取到的信号值大于此阈值时便认为检测一个脉搏。似乎用一个 `if` 语句就轻轻松松解决。但，事情真的有那么简单么？

其实这里存在两个问题。

![图6 信号波形变化][8]

**问题一：阈值的选取**

作为判断的参考标尺，阈值该选多大？10？100？还是1000？我们不得而知，**因为波形的电压范围是不确定的，振幅有大有小并且会改变，根本不能用一个写死的值去判断**。就像下面这张图一样：

![图7 振幅变化的波形][9]

可以看出，两个形状相同波形的检测结果截然不同 —— 同样是波峰，在不同振幅的波形中与阈值比较的结果存在差异。**实际情况正是如此：传感器输出波形的振幅是在不断随机变化的，想用一个固定的值去判定波峰是不现实的。**

既然固定阈值的方法不可取，那自然想到改变阈值 —— **根据信号振幅调整阈值，以适应不同信号的波峰检测**。通过对一个周期内的信号多次采样，得出信号的最高与最低电压值，由此算出阈值，再用这个阈值对采集的电压值进行判定，考虑是否为波峰。也就是说**电压信号的处理分两步，首先动态计算出参考阈值，然后用用阈值对信号判定、识别一个波峰。**

![图8 动态计算判定阈值][10]

**问题二：特征点识别**

上面得出的是**一段有效波形**，而计算 IBI **只需要一个点**。需要从一段有效信号上选取一个点，这里暂且把它称为特征点，这个特征点代表了一个有效脉搏，只要能识别到这个特征点，就能在一个脉搏到来时触发任何动作。

**通过记录相邻两个特征点的时间并求差值，计算 IBI 便水到渠成**。那这个特征点应该取在哪个位置呢，从[官网算法说明][11]可以看出，官方开源 arduino 代码的 v1.1 版本是选取**信号上升到振幅的一半**作为特征点，我们可以捕获这个特征点作为一个有效脉搏的标志，然后计算 IBI。

![图9 通过特征点计算 IBI][12]

#### 算法整体框架与代码实现

分析得出算法的整体框架如下：

1. 缓存一个波形周期内的多次采样值，求出最大最小值，计算出振幅中间值作为信号判定阈值
2. 通过把当前采样值和上一采样值与阈值作比较，寻找到**「信号上升到振幅中间位置」**的特征点，记录当前时间
3. 寻找下一个特征点并记录时间，算出两个点的时间差值，即相邻两次脉搏的时间间隔 IBI
4. 由 IBI 计算心率值 BPM

代码如下，程序中使用一个 50 长度的数组进行采样数据缓存，在主函数 `while (1)` 中以 20ms 的周期不断执行采样、数据处理，其中的条件语句 `if (PRE_PULSE == FALSE && PULSE == TRUE)` 就表示找到了特征点、识别出一次有效脉搏，串口输出心率计算结果。

```c
int main(void)
{
    Usrat_1_Init(84,115200,0);
    Timer_6_Init();
    ADC_AN2_Init();
    
    while(1)
    {
        //Print_Wave();
        
        preReadData = readData;	        // 保存前一次值
        readData = Get_ADC_1_CH2();		// 读取AD转换值
        
        if ((readData - preReadData) < filter)    // 滤除突变噪声信号干扰
            data[index++] = readData;	// 填充缓存数组

        if (index >= BUFF_SIZE)
        {	
            index = 0;	// 数组填满，从头再填
		
            // 通过缓存数组获取脉冲信号的波峰、波谷值，并计算中间值作为判定参考阈值
            max = Get_Array_Max(data, BUFF_SIZE);
            min = Get_Array_Min(data, BUFF_SIZE);
            mid = (max + min)/2;
            filter = (max - min) / 2;
        }
        
        PRE_PULSE = PULSE;	// 保存当前脉冲状态
        PULSE = (readData > mid) ? TRUE : FALSE;  // 采样值大于中间值为有效脉冲
        
        if (PRE_PULSE == FALSE && PULSE == TRUE)  // 寻找到「信号上升到振幅中间位置」的特征点，检测到一次有效脉搏
        {	
            pulseCount++;
            pulseCount %= 2;	 
            
            if(pulseCount == 1) // 两次脉搏的第一次
            {                         	
                firstTimeCount = timeCount;   // 记录第一次脉搏时间
            }
            if(pulseCount == 0)  // 两次脉搏的第二次
			{                             			
                secondTimeCount = timeCount;  // 记录第二次脉搏时间
                timeCount = 0;	

                if ( (secondTimeCount > firstTimeCount))
                {
                    IBI = (secondTimeCount - firstTimeCount) * SAMPLE_PERIOD;	// 计算相邻两次脉搏的时间，得到 IBI
                    BPM = 60000 / IBI;  // 通过 IBI 得到心率值 BPM
                    
                    if (BPM > 200)    //限制BPM最高显示值
                        BPM = 200;	 				
                    if (BPM < 30)    //限制BPM最低显示值
                        BPM=30;
                }
            
            }
            
            printf("SIG = %d IBI = %d, BMP = %d\r\n\r\n", readData, IBI, BPM);  // 串口打印调试
//          printf("B%d\r\n", BPM);  // 上位机B数据发送
//          printf("Q%d\r\n", IBI);  // 上位机Q数据发送
        }
		
        SIG = readData - 1500;  // 脉象图数值向下偏移，调整上位机图像
        
        
//      printf("S%d\r\n", SIG);  // 上位机S数据发送

        timeCount++;  // 时间计数累加
        Timer_6_Delay_ms(SAMPLE_PERIOD);  // 延时再进行下一周期采样
    }
}
```

将传感器正面轻按在食指上，单片机在每检测到一个脉搏时打印心率值 BPM 和相邻两次脉搏的时间间隔 IBI，实测结果还算稳定（为啥我心率这么高……）。

![图10 通过手指测量心率][13]

![图11 心率计算结果][14]

**注意事项：**

1. **避免手指触碰传感器背面**
2. **传感器与手指之间不要施加过大压力，否则会阻碍血液流动而读不到脉搏信号**
3. **传感器与手指之间的接触要保持稳定，按压力度的轻微变化都会影响电压值**

  ---

<br/>

### 使用 Processing 上位机查看心率

PulseSensor 官方提供了 Processing 语言编写的、超好看的上位机软件，而且项目在 Github 上开源，通过上位机可以查看实时心率图、心率值 BPM 和 脉搏间隔 IBI，那要如何使用这么好用的软件呢？

#### 修改串口数据发送格式

首先要对程序稍作修改，上位机通过解析串口接收到的数据进行心率图绘制、数据图形化显示。要想使用上位机查看心率。单片机串口发送的数据就必须符合上位机的解析格式。具体格式如下：

> 数据格式均为 ASCII 码，由于数据量较大，采用的波特率为 115200。其中包含三种数据：以「S」为前缀的，表示脉搏数据（脉象图的数值化表示）；以「B」为前缀的，表示 BPM 数值（心率值）；以「Q」为前缀的，表示 IBI 数值（相邻两个心跳之间的时间）。这三种数据通过串口发送给上位机 Processing 软件，就会在窗口中显示出来。其中 S 数据 20ms 发送一次，数据量大；B 和 Q 数据只有在检测到有效脉搏后，在每一次心跳后发送一次，数据量下。如下图：

![图12 上位机串口数据格式][15]

前面写的程序中串口波特率已经是 115200，采样频率也是所需的 20ms（代码中的宏定义 `SAMPLE_PERIOD` 的值就是 20），我们只需要计算出上位机所需要的 S、B、和 Q 三个数据，并在适时发送到串口即可。发送代码在上面程序中已经被注释掉，把串口打印调试代码换成注释掉的上位机数据发送  `printf()`  语句，编译程序并下载到开发板。

#### 下载 Processing 上位机软件

访问上位机的 Github 项目页面：[WorldFamousElectronics/PulseSensor_Amped_Processing_Visualizer: Processing code for pulse wave visualization][16]，点击 `Clone or download` 按钮下的 `Download ZIP` 即可下载上位机软件及其源码。

![图13 Github 页面下载 zip 源码][17]

#### 下载 Processing 运行环境、打开上位机

由于上位机是用 Processing 语言编写，不能直接打开，需要通过  Processing 运行环境加载运行。

从  Processing 官网下载对于版本的运行环境，地址：[Download \ Processing.org][18]

打开下载的 Processing 运行环境，点击右上角「文件」菜单的「打开」按钮，选择下载并解压后的上位机程序 `PulseSensorAmpd_Processing_150.pde`。

![图14 Processing 运行环境][19]

![图15 打开上位机软件][20]

#### 运行上位机，显示心率值

点击运行按钮运行软件，即可看到上位机软件界面。把开发板上电连接电脑，在软件中选择对应的 COM 口，随后软件开始接收串口数据并显示。

![图16 运行上位机][21]

![图17 选择 COM 口][22]

![图18 上位机心率显示][23]


*实测的波形极容易受传感器与手指的接触力度影响，可用透明胶带进行缠绕固定，数据会稳定许多。*

![图19 利用胶带固定传感器][24]

  ---
  
<br/>

### 总结

与许多可穿戴设备的心率传感器相比， PulseSensor 还存在很大差距，而自己写程序也仅仅是达到「勉强可用」的程度，输出数据偶尔还是会有大波动。代码也还有许多可改进的地方（比如将 20ms 的数据采样处理用定时器中断实现）。突然觉得**传感器采集到数据只是前提，对数据的处理才是一切应用的核心**，不断地调整参数、改良算法也是整个过程中最有趣的部分。

这次折腾脉搏传感器让我明白了一件事：**为什么我的数学这么渣！！！**

keil 工程源码下载：[测心率_脉搏传感器 密码 8ut8][25]

---

<br/>
<br/>

> 参考文章
> 
> * Pulse Sensor使用说明书V5.0
> * [Pulse Sensor Amped – Arduino Code v1.2 Walkthrough][26]


  [1]: http://odaps2f9v.bkt.clouddn.com/17-7-1/77669431.jpg
  [2]: http://odaps2f9v.bkt.clouddn.com/17-7-1/8118458.jpg
  [3]: https://pulsesensor.com/
  [4]: https://github.com/WorldFamousElectronics/PulseSensor_Amped_Arduino
  [5]: https://github.com/WorldFamousElectronics/PulseSensor_Amped_Processing_Visualizer
  [6]: http://odaps2f9v.bkt.clouddn.com/17-7-1/11580432.jpg
  [7]: http://odaps2f9v.bkt.clouddn.com/17-7-1/71887395.jpg
  [8]: http://odaps2f9v.bkt.clouddn.com/17-7-1/96596151.jpg
  [9]: http://odaps2f9v.bkt.clouddn.com/17-7-1/21470548.jpg
  [10]: http://odaps2f9v.bkt.clouddn.com/17-7-1/77092253.jpg
  [11]: https://pulsesensor.com/pages/pulse-sensor-amped-arduino-v1dot1
  [12]: http://odaps2f9v.bkt.clouddn.com/17-7-1/43185876.jpg
  [13]: http://odaps2f9v.bkt.clouddn.com/17-7-1/41986651.jpg
  [14]: http://odaps2f9v.bkt.clouddn.com/17-7-1/441961.jpg
  [15]: http://odaps2f9v.bkt.clouddn.com/17-7-1/5990022.jpg
  [16]: https://github.com/WorldFamousElectronics/PulseSensor_Amped_Processing_Visualizer
  [17]: http://odaps2f9v.bkt.clouddn.com/17-7-1/68621082.jpg
  [18]: https://processing.org/download/
  [19]: http://odaps2f9v.bkt.clouddn.com/17-7-1/62538313.jpg
  [20]: http://odaps2f9v.bkt.clouddn.com/17-7-1/54095781.jpg
  [21]: http://odaps2f9v.bkt.clouddn.com/17-7-1/30161575.jpg
  [22]: http://odaps2f9v.bkt.clouddn.com/17-7-1/35279678.jpg
  [23]: http://odaps2f9v.bkt.clouddn.com/17-7-1/25948724.jpg
  [24]: http://odaps2f9v.bkt.clouddn.com/17-7-1/87212409.jpg
  [25]: http://pan.baidu.com/s/1miC4QtM
  [26]: https://pulsesensor.com/pages/pulse-sensor-amped-arduino-v1dot1