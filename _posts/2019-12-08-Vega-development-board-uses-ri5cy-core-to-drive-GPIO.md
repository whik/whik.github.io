---
layout:     post
title:    织女星开发板使用RISC-V RI5CY核驱动GPIO
subtitle:	织女星开发板使用
date:       2019-12-08 22:22:40 +0800
author:     Wang Chao
header-img: img/RISC-V.png
catalog:    true
tag:
    - 织女星开发板
    - RISC-V
---

### 前言

织女星开发板是[OPEN-ISA社区](https://open-isa.cn/)为中国大陆地区定制的一款体积小、功耗超低和功能丰富的 RISC-V评估开发板，基于NXP半导体四核异构RV32M1主控芯片。

- 两个RISC-V核：RI5CY + ZERO_RISCY。
- 两个ARM核： Cortex-M4F + Cortex-M0+ 。

4个核被分为两个子系统，大核CM4F/RI5CY和小核CM0+/ZERO-RISCY，片上集成1.25 MB Flash 、384 KB SRAM，其中1 MB的Flash被大核所使用，起始地址0x0000_0000，另外的256 KB Flash被小核所使用，起始地址0x0100_0000。利用该开发板，用户可以快速建立一个使用 RV32M1 的 RISC-V应用和演示系统。详细的介绍可以参考： [真正的RISC-V开发板——VEGA织女星开发板开箱评测](http://www.wangchaochao.top/2019/06/22/VEGA-4/) ，本篇文章介绍如何基于RISC-V RI5CY内核点亮板载的RGB LED和读取按键状态，演示GPIO外设的使用。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/VegaLite/Pic/vega_2.jpeg)

### 准备工作

- 织女星开发板RISC-V开发环境：Eclipse + riscv32 工具链 + OpenOCD调试工具
- 织女星开发板SDK包：rv32m1_sdk_riscv
- 织女星开发板的原理图
- RV32M1参考手册

以上资料的获取、开发环境搭建和启动模式修改等教程，可以到官方中文论坛查找： www.open-isa.cn

或者是参考我分享的以下文章：

- [真正的RISC-V开发板——VEGA织女星开发板开箱评测](http://www.wangchaochao.top/2019/06/22/VEGA-4/)
- [手把手教你搭建织女星开发板RISC-V开发环境](http://www.wangchaochao.top/2019/05/30/VEGA-3/)
- [织女星开发板启动模式修改——从ARM M4核启动](http://www.wangchaochao.top/2019/05/28/VEGA-2/)
- [织女星开发板调试器升级为Jlink固件](http://www.wangchaochao.top/2019/05/26/VEGA-1/)
- [织女星开发板RISC-V内核实现微秒级精确延时](http://www.wangchaochao.top/2019/06/28/VEGA-5/) 

### 寄存器简介

根据参考手册GPIO章节的介绍，我们可以获得关于GPIO相关寄存器信息：

各GPIO组的基地址：

    GPIOA——4802_0000h
    GPIOB——4802_0040h
    GPIOC——4802_0080h
    GPIOD——4802_00C0h
    GPIOE——4100_F000h

#### GPIO配置PCR寄存器

这是一个32位的寄存器，每一个引脚都有对应的一个PORTx_PCRn，用来配置GPIO的以下功能：

- 上下拉配置
- 翻转速率控制
- 开漏使能
- 无源输入滤波器
- 寄存器锁定
- 复用功能设置

以PA0控制寄存器，PORTA_PCR0为例：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/VEGA_GPIO/PA0_PCR0.jpg)

通过查看参考手册，可以了解到各Bit的功能：

- ISF：1位，中断状态标志
- IRQC：4位，配置中断方式和DMA功能
- LK：1位，是否锁定PCR寄存
- MUX：3位，复用功能配置
- ODE：1位，推挽开漏配置
- PFE：1位，滤波器配置
- SRE：1位，翻转速率配置
- PE：1位，上下拉使能
- PS：1位，上下拉配置

详细的配置介绍可以查看参考手册。官方库fsl_port中的

```

PORT_SetPinConfig(PORT_Type *base, uint32_t pin, const port_pin_config_t *config)
PORT_SetPinMux(PORT_Type *base, uint32_t pin, port_mux_t mux)
PORT_SetPinInterruptConfig(PORT_Type *base, uint32_t pin, port_interrupt_t config)
PORT_SetPinDriveStrength(PORT_Type* base, uint32_t pin, uint8_t strength)

```

这些函数就是控制的这个PCR寄存器。

#### GPIO控制寄存器

主要包括控制GPIO输入输出控制，读取输入，控制输出，方向控制等。

寄存器描述和地址偏移量：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/VEGA_GPIO/%E7%AB%AF%E5%8F%A3%E5%AF%84%E5%AD%98%E5%99%A8%E9%85%8D%E7%BD%AE.jpg)

RV32M1的GPIO共有6个32位的控制寄存器，从字面意思可以直接知道每个寄存器的功能：

- PDOR：数据输出寄存器，指定位写入0/1，输出0/1
- PSOR：端口置位输出寄存器，指定位写1，置位输出1，写0状态不变
- PCOR：端口复位输出寄存器，指定位写1，复位输出0，写0状态不变
- PTOR：端口反转输出寄存器，指定位写1，反转输出，写0状态不变
- PDIR：端口输入寄存器，读取指定位输入状态
- PDDR：端口方向配置寄存器，指定位写0作为输入，写1作为输出

官方库中的fsl_gpio文件中实现的函数就是控制的这几个寄存器。

```

void GPIO_PinInit(GPIO_Type *base, uint32_t pin, const gpio_pin_config_t *config)
void GPIO_WritePinOutput(GPIO_Type *base, uint32_t pin, uint8_t output)
void GPIO_SetPinsOutput(GPIO_Type *base, uint32_t mask)
void GPIO_ClearPinsOutput(GPIO_Type *base, uint32_t mask)
void GPIO_TogglePinsOutput(GPIO_Type *base, uint32_t mask)

```

### 库函数简介

和其他的MCU一样，由于RV32M1的寄存器众多，为了方便使用，增强程序的可读性，官方开发了库函数，来实现对寄存器的控制，本质上还是操作的寄存器。GPIO控制的库主要由fsl_gpio和fsl_port两个文件组成，其中fsl_gpio主要是对GPIO的控制，如读取输入，控制输出，清除中断标志等，而fsl_port主要实现对GPIO工作的模式进行配置，如复用功能，上拉下拉，开漏推挽，中断触发方式，DMA功能等进行设置。

下面简单介绍几个常用的函数：

#### PORT_SetPinConfig

配置GPIO的复用功能，驱动能力，推挽开漏，上下拉，滤波器，翻转速率等功能，基于PCR寄存器实现。

```

port_pin_config_t config;

config.driveStrength = kPORT_HighDriveStrength;		//驱动能力配置
config.mux = kPORT_MuxAsGpio;						//通用GPIO
config.openDrainEnable = kPORT_OpenDrainDisable;	//推挽
config.passiveFilterEnable = kPORT_PassiveFilterDisable;//滤波器
config.pullSelect = kPORT_PullUp;					//上拉
config.slewRate = kPORT_FastSlewRate;				//翻转速率

PORT_SetPinConfig(PORTA, 22, &config);				//配置GPIOA22

```

#### PORT_SetPinMux

配置GPIO的复用功能，基于PCR寄存器实现。

```
//PA22作为普通GPIO使用
PORT_SetPinMux(PORTA, 22, kPORT_MuxAsGpio);

//PA25作为UART1_RX功能
PORT_SetPinMux(PORTA, 25, kPORT_MuxAlt2);

```

具体复用为哪种功能，不同的引脚有不同的复用功能，对应的ALTn，可以查看参考手册RV32M1 Pinout介绍。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/VEGA_GPIO/GPIOA_MUX.jpg)

PORT_SetPinConfig已经包含了PORT_SetPinMux的功能，可以只使用PORT_SetPinConfig来GPIO功能的配置。PORT_SetPinMux函数不推荐和PORT_SetPinsConfig函数一起使用：

> This function is NOT recommended to use together with the PORT_SetPinsConfig, because the PORT_SetPinsConfig need to configure the pin mux anyway (Otherwise the pin mux is reset to zero : kPORT_PinDisabledOrAnalog). This function is recommended to use to reset the pin mux

#### GPIO_PinInit

控制GPIO的输入输出方式，及默认输出电平，基于PDDR、PCOR、PSOR寄存器实现。

```
gpio_pin_config_t io_init;

//配置输出/输出模式
io_init.outputLogic = 0;	//默认输出0
io_init.pinDirection = kGPIO_DigitalOutput;	//数字输出

GPIO_PinInit(LED_RGB_GPIO, LED_RED_Pin, &io_init);	//LED引脚配置

```

#### GPIO_WritePinOutput

指定引脚输出高低电平，基于PCOR和PSOR寄存器实现。

	GPIO_WritePinOutput(GPIOA, 22, 1);	//PA22输出1

#### GPIO_TogglePinsOutput

指定引脚输出翻转，基于PTOR寄存器实现

	GPIO_TogglePinsOutput(GPIOA, 1 << 22);	//PA22输出翻转

#### GPIO_ReadPinInput

读取GPIO输入状态，基于PDIR寄存器实现

	in = GPIO_ReadPinInput(GPIOA, 22);	//读取PA22输入状态

GPIO操作的函数还有很多，详细的介绍和实现可以直接查看库函数源码。

### RGB LED的初始化

从原理图中我们可以得知，织女星开发板上共有4个用户可控制的LED，包括3个RGB LED和1个红色LED，均采用MOS来驱动，引脚输出高电平LED点亮，和GPIO的对应关系如下：

> LED_RED——PTA24
> LED_GREEN——PTA23
> LED_BLUE——PTA22
> LED_STS——PTE0

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/VEGA_GPIO/LED_GPIO.jpg)

所以我们需要配置PTA22/PTA23/PTA24为普通推挽输出方式，然后输出高低电平就可以控制LED闪烁了。

#### led_driver.c文件内容

```
#include "led_driver.h"

void LED_RGB_Init(void)
{
	gpio_pin_config_t io_init;
	port_pin_config_t config;

	//配置输出/输出模式
	io_init.outputLogic  = 0;
	io_init.pinDirection = kGPIO_DigitalOutput;

	config.driveStrength 		= kPORT_HighDriveStrength;	 //驱动能力
	config.lockRegister 		= kPORT_LockRegister;		 //PCR寄存器被锁定，不能再次改变
	config.mux 					= kPORT_MuxAsGpio;			 //通用GPIO
	config.openDrainEnable 		= kPORT_OpenDrainDisable;	 //推挽输出
	config.passiveFilterEnable 	= kPORT_PassiveFilterDisable;//滤波器
	config.pullSelect 			= kPORT_PullUp;				 //上拉
	config.slewRate 			= kPORT_FastSlewRate;		 //翻转速率

	CLOCK_EnableClock(LED_RGB_Clk_Name);

	/*以下两个函数都可以配置端口功能*/
	PORT_SetPinConfig(LED_RGB_Port, LED_RED_Pin, &config);		//配置功能更详细
	PORT_SetPinConfig(LED_RGB_Port, LED_GREEN_Pin, &config);
	PORT_SetPinConfig(LED_RGB_Port, LED_BLUE_Pin, &config);

//	PORT_SetPinMux(LED_RGB_Port, LED_RED_Pin, kPORT_MuxAsGpio);	//只能配置是否复用
//	PORT_SetPinMux(LED_RGB_Port, LED_GREEN_Pin, kPORT_MuxAsGpio);
//	PORT_SetPinMux(LED_RGB_Port, LED_BLUE_Pin, kPORT_MuxAsGpio);

//	CLOCK_DisableClock(LED_RGB_Clk_Name);		//可以在配置完成之后关闭时钟，不影响使用

	GPIO_PinInit(LED_RGB_GPIO, LED_RED_Pin, &io_init);
	GPIO_PinInit(LED_RGB_GPIO, LED_GREEN_Pin, &io_init);
	GPIO_PinInit(LED_RGB_GPIO, LED_BLUE_Pin, &io_init);
}

```

要注意的是，时钟使能要放在GPIO配置之前，否则不能访问GPIO配置寄存器，在配置完成之后可以关闭时钟，也可以一直开启。

#### led_driver.h文件内容

```
#ifndef __LED_DRIVER_H__
#define __LED_DRIVER_H__

#include "fsl_gpio.h"
#include "fsl_port.h"
#include "fsl_clock.h"


/*
LED_RGB_BLUE  	- A22
LED_RGB_GREEN 	- A23
LED_RGB_RED 	- A24
LED_STS 		- E0
*/

#define LED_RED_Pin		24
#define LED_GREEN_Pin	23
#define LED_BLUE_Pin	22

#define LED_RGB_Port		PORTA
#define LED_RGB_GPIO		GPIOA
#define LED_RGB_Clk_Name	kCLOCK_PortA

#define LED_RED_ON			GPIO_WritePinOutput(LED_RGB_GPIO, LED_RED_Pin, 1)
#define LED_RED_OFF			GPIO_WritePinOutput(LED_RGB_GPIO, LED_RED_Pin, 0)
#define LED_RED_TOGGLE		GPIO_TogglePinsOutput(LED_RGB_GPIO, 1 << LED_RED_Pin)

#define LED_GREEN_ON		GPIO_WritePinOutput(LED_RGB_GPIO, LED_GREEN_Pin, 1)
#define LED_GREEN_OFF		GPIO_WritePinOutput(LED_RGB_GPIO, LED_GREEN_Pin, 0)
#define LED_GREEN_TOGGLE 	GPIO_TogglePinsOutput(LED_RGB_GPIO, 1 << LED_GREEN_Pin)

#define LED_BLUE_ON			GPIO_WritePinOutput(LED_RGB_GPIO, LED_BLUE_Pin, 1)
#define LED_BLUE_OFF		GPIO_WritePinOutput(LED_RGB_GPIO, LED_BLUE_Pin, 0)
#define LED_BLUE_TOGGLE		GPIO_TogglePinsOutput(LED_RGB_GPIO, 1 << LED_BLUE_Pin)

void LED_RGB_Init(void);

#endif

```

头文件中通过宏定义的方式实现了LED的亮灭和翻转控制。

### 板载按键初始化

按键部分硬件原理图，默认SW2上拉，按下为低电平。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/VEGA_GPIO/SW_GPIO.jpg)

#### button_driver.c文件内容

```

#include "button_driver.h"

void Button_Init(void)
{
	gpio_pin_config_t io_init;
	port_pin_config_t config;

	io_init.outputLogic  = 0;
	io_init.pinDirection = kGPIO_DigitalInput;

	config.mux 					= kPORT_MuxAsGpio;				//通用GPIO
	config.lockRegister 		= kPORT_LockRegister;			//PCR寄存器被锁定，不能再次改变
	config.pullSelect 			= kPORT_PullUp;					//上拉
	config.slewRate 			= kPORT_FastSlewRate;			//翻转速率
	config.lockRegister 		= kPORT_LockRegister;			//PCR寄存器被锁定，不能再次改变
	config.passiveFilterEnable 	= kPORT_PassiveFilterEnable;	//滤波器

	CLOCK_EnableClock(BTN_SW2_Clk_Name);

	//以下两个函数功能一样
	PORT_SetPinConfig(BTN_SW2_Port, BTN_SW2_Pin, &config);
//	PORT_SetPinMux(BTN_SW2_Port, BTN_SW2_Pin, kPORT_MuxAsGpio);	//设置IO模式为通用GPIO

	GPIO_PinInit(BTN_SW2_GPIO, BTN_SW2_Pin, &io_init);
}

```

SW2按键配置为上拉输入模式。

#### button_driver.h文件内容

```

ifndef __BUTTON_DRIVER_H__
#define __BUTTON_DRIVER_H__

#include "fsl_gpio.h"
#include "fsl_port.h"

/*
 * SW2 - PA0
 *
 * */

//按下为低电平

#define BTN_SW2_GPIO 	GPIOA
#define BTN_SW2_Pin		0
#define BTN_SW2_Port	PORTA
#define BTN_SW2_Clk_Name	kCLOCK_PortA

#define BTN_SW2_IN	GPIO_ReadPinInput(BTN_SW2_GPIO, BTN_SW2_Pin)

/*
#define BTN_SW2_IN	ReadGPIO(BTN_SW2_GPIO, BTN_SW2_Pin)
*/

void Button_Init(void);

#endif

```

通过GPIO读取函数来获取SW2输入状态。

### 主函数应用

主函数实现sw2按键按下时，蓝色LED状态翻转：

```

int main(void)
{
	BOARD_BootClockRUN();	//系统时钟配置

	UART0_Init();
	Delay_Init();
	LOG("Hello RV32M1 \r\n");

	LED_RGB_Init();
	Button_Init();

	while (1)
	{
//		LED_RED_TOGGLE;
//		Delay_ms(100);
		if(BTN_SW2_IN == 0)
		{
			while(!BTN_SW2_IN);
			LOG("sw2 press \r\n");
			LED_BLUE_TOGGLE;
		}
	}
}

```

### 代码下载

![]( https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/VegaLite/BuildIDE/LED_Blink.gif )

织女星开发板VEGA_Lite支持从4个核启动，所以在进行程序下载之前，要确认当前的启动模式是从RISC-V RI5CY核启动的，否则程序烧写进去不会运行。

关于启动模式的修改可以参考：[织女星开发板启动模式修改](http://www.wangchaochao.top/2019/05/28/VEGA-2/)

Eclipse工程源码下载：[RI5CY_GPIO_Demo.rar](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/VEGA_GPIO/RI5CY_GPIO_Demo.rar)

### 总结

GPIOE组的使用，和其他组的GPIO使用不太一样，需要特别设置一下，目前还没有找到相关资料，所以板子上的LED_STS，和SW3/4/5还没成功的驱动起来，等找到相关资料再来补充。

### 参考资料

- [MCUXpresso SDK API参考手册](https://mcuxpresso.nxp.com/api_doc/dev/210/group__gpio__driver.html )
- [RV32M1_Vega_Develop_Environment_Setup.pdf](https://gitee.com/whik/open-isa.org/raw/master/RV32M1_Vega_Develop_Environment_Setup.pdf) 
- [RV32M1数据手册](https://github.com/open-isa-org/open-isa.org/raw/master/Reference Manual and Data Sheet/RV32M1DS_Rev.1.1.pdf) 
- [RV32M1参考手册](https://github.com/open-isa-org/open-isa.org/raw/master/Reference Manual and Data Sheet/RV32M1RM_Rev.1.1.pdf) 
- [织女星开发板快速入门指南.pdf](https://github.com/open-isa-org/open-isa.org/raw/master/RV32M1_VEGA_Quick_Start_Guide.pdf) 

### 推荐阅读

- [真正的RISC-V开发板——VEGA织女星开发板开箱评测](http://www.wangchaochao.top/2019/06/22/VEGA-4/)
- [手把手教你搭建织女星开发板RISC-V开发环境](http://www.wangchaochao.top/2019/05/30/VEGA-3/)
- [织女星开发板启动模式修改——从ARM M4核启动](http://www.wangchaochao.top/2019/05/28/VEGA-2/)
- [织女星开发板调试器升级为Jlink固件](http://www.wangchaochao.top/2019/05/26/VEGA-1/)
- [织女星开发板RISC-V内核实现微秒级精确延时](http://www.wangchaochao.top/2019/06/28/VEGA-5/) 

---

- 个人博客：www.wangchaochao.top
- 我的公众号：mcu149

