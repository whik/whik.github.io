---
layout:     post
title:    [踩坑]STM32外部8M不起振会有什么现象？
subtitle:	STM32踩坑记录 
date:       2020-06-20 16:00:00 +0800
author:     Wang Chao
header-img: img/post-bg-pulsesensor.jpg
catalog:    true
tag:
    - STM32
---

### 8M晶体不起振是什么现象？

最近公司做了几块基于STM32的板子，芯片是用的F103CBT6，打样焊接回来，先测试一下硬件是否能正常工作，简单写了个测试代码，看看程序下载运行，GPIO控制这些是否正常，很简单的一个程序，LED每100ms翻转一次：

```c

#include "main.h"

int main(void)
{
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);
    delay_init();
    led_init();

    while(1)
    {
		led_set(1, ON);
		led_set(2, ON);
		led_set(3, ON);
		led_set(4, ON);
		delay_ms(100);

		led_set(1, OFF);
		led_set(2, OFF);
		led_set(3, OFF);
		led_set(4, OFF);
		delay_ms(100);  
	}
}

```

程序下载，运行，有一些奇怪的地方，程序中是每100ms变化一次，可实际观察却是近1s闪烁一次。

示波器一测，实际上是900ms闪烁一次。改了个其他的时间1ms，10ms等，发现都是实际设置的9倍时间，这是为什么呢？

### 8M晶体为什么不起振

示波器探头一量晶振的两个管脚，没有波形！

难道是焊接问题，我又拿了另外一块新板子，没烧程序的，同样是没有波形。

为了排除程序配置的问题，我又找了一块正常的开发板，运行正常，延时时间也能对上，说明程序是没问题的！

我又量了开发板上的晶振波形，两个管脚都是1v-3.3v，8M频率的正弦波，如下图所示：

![F0075TEK](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200620/F0075TEK.BMP)

于是便开启了硬件调试模式，一顿操作猛如虎：先拆了外部8M无源晶振和两颗匹配电容，使用信号发生器输出3.3v的8M方波，接到OSC_IN上，再次上电，完美运行，延时是准确的！

可以确定是晶振部分电路的问题，一共就3个元件，两个电容和1个8M的无源晶体，晶体一般不会有什么问题，最有可能的就是匹配电容的大小不对。

拿起万用表一量，高高的100nF！换上个39pF的电容，焊接上晶振，波形完美，程序运行正常！

最后一查，是**硬件工程师的物料BOM错了**，误把这两颗关键性的电容和100nF的电容合并到一起了。

### 怎么看8M晶体是否起振了

当然，最简单的方法，就是烧录好程序，直接使用示波器测量晶振的两端。如果是焊接的全新的芯片，还没有烧写程序，直接测量晶振是没有波形的。或者是使用调试器进行全片擦除，也是量不到波形的。

能不能从程序中读出当前晶振是否起振了呢？

	printf("当前系统主频:%d, 外部晶振状态: %d\r\n",SystemCoreClock, RCC->CR & RCC_CR_HSERDY);

从STM32的启动流程可以看出，在执行main主函数之前，会通过SystemInit()函数完成系统时钟的配置，`RCC->CR & RCC_CR_HSERDY`这个值就表示当前外部晶振是否准备就绪，0为异常，1为正常。

当外部晶振无法就绪时，会自动启用内部HSI 8M RC晶振作为系统主频，即主频只有8MHz，这也就是为什么延时时间相差9倍的原因！

```c

static void SetSysClockTo72(void)
{
    __IO uint32_t StartUpCounter = 0, HSEStatus = 0;

    /* SYSCLK, HCLK, PCLK2 and PCLK1 configuration ---------------------------*/
    /* Enable HSE */
    RCC->CR |= ((uint32_t)RCC_CR_HSEON);

    /* 如果外部晶振没有其中，RCC->CR & RCC_CR_HSERDY恒为0*/
    do
    {
        HSEStatus = RCC->CR & RCC_CR_HSERDY;
        StartUpCounter++;
    }
    while ((HSEStatus == 0) && (StartUpCounter != HSE_STARTUP_TIMEOUT));

    if ((RCC->CR & RCC_CR_HSERDY) != RESET)
    {
        HSEStatus = (uint32_t)0x01;
    }
    else /* 满足这个条件 */
    {
        HSEStatus = (uint32_t)0x00;	
    }

    /* HSEStatus=0，不满足，无法完成PLL配置 */
    if (HSEStatus == (uint32_t)0x01)
    {
        /*  PLL configuration: PLLCLK = HSE * 9 = 72 MHz */
        RCC->CFGR &= (uint32_t)((uint32_t)~(RCC_CFGR_PLLSRC | RCC_CFGR_PLLXTPRE |
                                            RCC_CFGR_PLLMULL));
        RCC->CFGR |= (uint32_t)(RCC_CFGR_PLLSRC_HSE | RCC_CFGR_PLLMULL9);
    }
}

```

顺手量了RS232串口的波形 ：

![image-20200620174106185](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200620/F0073TEK.BMP)

上面是3.3v TTL串口信号，也就是普通的单片机IO口串口信号，下面的是MAX232转换之后的232电平的串口信号，大小正负5v，上升和下降时间比TTL电平要长一些 。

### 总结

> 一般来说，无源晶体的负载电容越大，其振荡越稳定，但是会增加起振时间，太大会导致完全不能起振，为了稳定波形，可以在晶振两端并联一个1M到10M的反馈电阻。

这次遇到的问题，可总结为两点：

- 新板子+新芯片，没烧程序，晶振没有波形是正常的
- 新板子烧写正确配置的程序，延时时间相差9倍，是因为外部晶振无波形，主频不对
- 外部晶振无波形是因为匹配电容100nF太大了，无法起振。

以STM32F103CBT6，外部8M无源晶振为例，以下是我实践得出的结论：

- 刚做回来的板子，STM32还没有下载程序，8M晶振是测不到波形的。
- STM32芯片下载过程序，并配置正确，8M晶振会有波形，最小1v，最大3.3v，8M频率的正弦波，两个管脚都可以测到。
- STM32芯片下载过程序，再整片完全擦除，8M晶振测不到波形。
- STM32芯片8M无源晶振匹配电容太大，会导致晶振不能起振，无波形。
- 一般无源晶振是正弦波，有源晶振是方波。

