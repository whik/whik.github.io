---
layout:     post
title:    织女星开发板使用RISC-V ZERO核驱动GPIO
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

4个核被分为两个子系统，大核CM4F/RI5CY和小核CM0+/ZERO-RISCY，片上集成1.25 MB Flash 、384 KB SRAM，其中1 MB的Flash被大核所使用，起始地址0x0000_0000，另外的256 KB Flash被小核所使用，起始地址0x0100_0000。利用该开发板，用户可以快速建立一个使用 RV32M1 的 RISC-V应用和演示系统。详细的介绍可以参考： [真正的RISC-V开发板——VEGA织女星开发板开箱评测](http://www.wangchaochao.top/2019/06/22/VEGA-4/) ，本篇文章介绍如何基于RISC-V ZERO内核点亮板载的RGB LED和读取按键状态，演示GPIO外设的使用。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/VegaLite/Pic/vega_2.jpeg)

### 准备工作

- 织女星开发板RISC-V开发环境：Eclipse + riscv32 工具链 + OpenOCD调试工具
- 织女星开发板SDK包：rv32m1_sdk_riscv
- 织女星开发板的原理图
- RV32M1参考手册

### ZERO和RI5CY核的区别

与RI5CY核一样，ZERO也是属于RISC-V架构的内核，寄存器、库函数这些都是通用的，可以参考上一篇文章：[织女星开发板使用RISC-V RI5CY核驱动GPIO](http://www.wangchaochao.top/2019/12/08/Vega-development-board-uses-ri5cy-core-to-drive-GPIO/) ，里面有详细的寄存器、库函数介绍。


ZERO核驱动GPIO，部分内容可以参考：[织女星开发板使用RISC-V RI5CY核驱动GPIO](http://www.wangchaochao.top/2019/12/08/Vega-development-board-uses-ri5cy-core-to-drive-GPIO/) 
，不同地方的就是，startup文件夹，主要包括启动文件、system文件等。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/VEGA_GPIO/zero_ri5cy.jpg)

还有一点就是SysTick定时器不同：

 * ZERO : **LPIT1_CH0**
 * RI5CY: **LPIT0_CH0**

在延时函数中需要对应的修改，可参考文章：[织女星开发板RISC-V内核实现微秒级精确延时](http://www.wangchaochao.top/2019/06/28/VEGA-5/) 

### LED的驱动

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

	CLOCK_DisableClock(LED_RGB_Clk_Name);		//可以在配置完成之后关闭时钟，不影响使用

	GPIO_PinInit(LED_RGB_GPIO, LED_RED_Pin, &io_init);
	GPIO_PinInit(LED_RGB_GPIO, LED_GREEN_Pin, &io_init);
	GPIO_PinInit(LED_RGB_GPIO, LED_BLUE_Pin, &io_init);
}

```

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

### 按键的驱动

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

#### button_driver.h文件内容

```

#ifndef __BUTTON_DRIVER_H__
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

### 主函数

#### main.c文件内容

```

#include "main.h"

int main(void)
{
	BOARD_BootClockRUN();	//系统时钟配置

	UART0_Init();
	Delay_Init();
	LOG("Hello RV32M1 RISC-V ZERO CORE\r\n");

	LED_RGB_Init();
	Button_Init();

	while (1)
	{
		LED_RED_TOGGLE;
		Delay_ms(100);
		if(BTN_SW2_IN == 0)
		{
			while(!BTN_SW2_IN);
			LOG("sw2 press \r\n");
			LED_BLUE_TOGGLE;
		}
	}
}

```

#### main.h文件内容

```

#ifndef __TEMPLATE_H__
#define __TEMPLATE_H__

/*
 * GPIO控制LED和读取按键输入示例程序
 *
 * */

#include <button_driver.h>		//按键
#include "clock_config.h"
#include "fsl_debug_console.h"

/* 实现的延时和GPIO控制函数 */
#include "delay.h"		//包含精确延时us和ms函数
#include "sys.h"		//包含GPIO操作，读取输入和输出，设置方向等
#include "usart.h"

#include "led_driver.h"	//LED
#include "button_driver.h"

#include "timer.h"


#endif

```

### 启动模式的修改

织女星开发板默认是从RI5CY核启动，如果开发ZERO核的程序，需要将启动模式修改为**ZERO核**启动，可以参考：

- [织女星开发板启动模式修改](http://www.wangchaochao.top/2019/05/28/VEGA-2/)

### 代码下载

![]( https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/VegaLite/BuildIDE/LED_Blink.gif )

织女星开发板VEGA_Lite支持从4个核启动，所以在进行程序下载之前，要确认当前的启动模式是从RISC-V **ZERO核**启动的，否则程序烧写进去不会运行。

Eclipse工程源码下载：[RI5CY_GPIO_Demo.rar](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/VEGA_GPIO/ZERO_GPIO_Demo.rar)

### 总结

GPIOE组的使用，和其他组的GPIO使用不太一样，需要特别设置一下，目前还没有找到相关资料，所以板子上的LED_STS，和SW3/4/5还没成功的驱动起来，等找到相关资料再来补充。

### 参考资料

- [MCUXpresso SDK API参考手册](https://mcuxpresso.nxp.com/api_doc/dev/210/group__gpio__driver.html )
- [RV32M1_Vega_Develop_Environment_Setup.pdf](https://gitee.com/whik/open-isa.org/raw/master/RV32M1_Vega_Develop_Environment_Setup.pdf) 
- [RV32M1数据手册](https://github.com/open-isa-org/open-isa.org/raw/master/Reference Manual and Data Sheet/RV32M1DS_Rev.1.1.pdf) 
- [RV32M1参考手册](https://github.com/open-isa-org/open-isa.org/raw/master/Reference Manual and Data Sheet/RV32M1RM_Rev.1.1.pdf) 
- [织女星开发板快速入门指南.pdf](https://github.com/open-isa-org/open-isa.org/raw/master/RV32M1_VEGA_Quick_Start_Guide.pdf) 

### 推荐阅读

- [织女星开发板使用RISC-V RI5CY核驱动GPIO](http://www.wangchaochao.top/2019/12/08/Vega-development-board-uses-ri5cy-core-to-drive-GPIO/)
- [真正的RISC-V开发板——VEGA织女星开发板开箱评测](http://www.wangchaochao.top/2019/06/22/VEGA-4/)
- [手把手教你搭建织女星开发板RISC-V开发环境](http://www.wangchaochao.top/2019/05/30/VEGA-3/)
- [织女星开发板启动模式修改](http://www.wangchaochao.top/2019/05/28/VEGA-2/)
- [织女星开发板调试器升级为Jlink固件](http://www.wangchaochao.top/2019/05/26/VEGA-1/)
- [织女星开发板RISC-V内核实现微秒级精确延时](http://www.wangchaochao.top/2019/06/28/VEGA-5/) 

---

- 个人博客：www.wangchaochao.top
- 我的公众号：mcu149

