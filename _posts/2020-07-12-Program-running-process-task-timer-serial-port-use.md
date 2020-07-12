---
layout:     post
title:    FR8016H程序运行流程、任务、定时器、串口的使用
subtitle:	富芮坤FR8106H使用
date:       2020-07-12 23:00:00 +0800
author:     Wang Chao
header-img: img/fr8016h_2019_ncov.jpg
catalog:    true
tag:
    - ARM
---


刚拿到开发板的时候，下载了SDK，其中包含了示例工程。按照惯性思维，先找main主函数，怎么也找不到，原来这款芯片的运行流程有点不同，采用的是lib封装和任务的方式。这周末仔细通读了一遍SDK使用指南，感觉豁然开朗。如果你有RTOS使用经验，那么对于这款新品的开发流程会非常熟悉。

#### FR8016H的空间地址分配

内置 128KB ROM空间，主要内容为启动代码、BLE controller 部分协议栈；FLASH 空间用于存储用户程序、用户数据等；RAM 用于存储各种变量、堆栈、重新映射后的中断向量地址、对运行速度较为敏感的代码（中断响应等）等，该空间都支持低功耗的 retention 功能；外设地址空间是各种外设的地址映射，用于进行外设的配置。

![2020-07-12_175519](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200712/2020-07-12_175519.jpg)

在 FR801xH 中 FLASH 空间和 RAM 空间的分配由链接脚本指定，具体分配如下：

![2020-07-12_175650](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200712/2020-07-12_175650.jpg)

其中 JUMP_TABLE 存储的是配置信息；APP CODE 和 RO DATA 可以通过 XIP 被 MCU 直接访问；CRITICAL CODE 和EXCEPTION and INTERRUPT HANDLER 为对运行时间敏感的用户代码，需要在初始化时从 flash 中搬移到 RAM 中；RW DATA 需要进行初始化；ZI 为初始值为 0 的数据段。这些操作均由 SDK 内部进行处理，用户无需做额外操作。HEAP 为动态内存分配空间，SDK 中会根据实际可用空间对内存管理单元进行初始化；STACK 为堆栈空间，生长空间由高到低，大小可由用户指定。

Keil中要使用到的sct链接文件`ble_5_0.sct`内容：

```asm
;256k bytes, which is 2M ROM
;ROM 0x00000000  0x40000	0x30000
ROM 0x01000000  0x80000
{
    ER_TABLE +0
    {
        *(jump_table_0)
        *(jump_table_1)
        *(jump_table_2)
        *(jump_table_3)
        *(jump_table_4)
    }
    ER_RO +0
	{
		*(+RO)
	}
    ER_BOOT 0x20000000
    {
        app_boot_vectors.o (RESET, +FIRST)
    }
    USER_RE_RAM_FRONT 0x20000B60
	{
		*(ram_code_front)
    }
    USER_RE_RAM +0
	{
		*(ram_code)
    }
    ER_RW +0
    {
        *(+RW)
    }
    ER_ZI +0
    {
        *(+ZI)
    }
	HEAP_KE +0 
	{
        *(heap_ke)
    }
}
```

#### 程序运行流程

SDK 包含了四大部分，Application 部分，蓝牙协议栈部分，操作系统抽象层 OSAL 部分，还有 MCU 外设驱动部分。代码结构比较简单，执行流程也很清晰易懂。SDK 的 main 函数主体入口位于 lib 库中，对于应用层以源码形
式开放了一些入口，用于应用开发初始化，基本流程如下图所示：

![2020-07-12_175018](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200712/2020-07-12_175018.jpg)



一下几个函数是程序执行的关键：

```c
/* 设置蓝牙参数，程序的版本信息，系统主频等 */
void user_custom_parameters(void)
/*  */
void user_entry_before_ble_init(void)
void user_entry_after_ble_init(void)
    
__attribute__((section("ram_code"))) void user_entry_before_sleep_imp(void)
__attribute__((section("ram_code"))) void user_entry_after_sleep_imp(void)

```

user_custom_parameters函数用于配置蓝牙参数和系统主频等

```c
void user_custom_parameters(void)
{
    /* 设置本机蓝牙MAC地址 */
    __jump_table.addr.addr[0] = 0xBD;
    __jump_table.addr.addr[1] = 0xAD;
    __jump_table.addr.addr[2] = 0xD0;
    __jump_table.addr.addr[3] = 0xF0;
    __jump_table.addr.addr[4] = 0x80;
    __jump_table.addr.addr[5] = 0x10;
    
    __jump_table.image_size = 0x19000; // 100KB
	__jump_table.firmware_version = 0x00010000;
    __jump_table.system_clk = SYSTEM_SYS_CLK_48M;
    
    /* 取消sleep模式 */
    __jump_table.system_option &= ~(SYSTEM_OPTION_SLEEP_ENABLE);
	jump_table_set_static_keys_store_offset(0x7d000);
}
```

如果不想让系统进入休眠低功耗模式，可以使用以下语句配置取消休眠：

```
    __jump_table.system_option &= ~(SYSTEM_OPTION_SLEEP_ENABLE);
```

user_entry_before_ble_init函数用于配置基本外设，如GPIO、UART、I2C等，配置PMU中断等。

```c
void user_entry_before_ble_init(void)
{    
    /* 设置芯片供电选择 */
    pmu_set_sys_power_mode(PMU_SYS_POW_BUCK);
    /* 使能PMU的一些中断类型 */
    pmu_enable_irq(PMU_ISR_BIT_ACOK
                   | PMU_ISR_BIT_ACOFF
                   | PMU_ISR_BIT_ONKEY_PO
                   | PMU_ISR_BIT_OTP
                   | PMU_ISR_BIT_LVD
                   | PMU_ISR_BIT_BAT
                   | PMU_ISR_BIT_ONKEY_HIGH);
    NVIC_EnableIRQ(PMU_IRQn);
    
    /* 设置PA2/3为串口1功能, 115200 */
    system_set_port_mux(GPIO_PORT_A, GPIO_BIT_2, PORTA2_FUNC_UART1_RXD);
    system_set_port_mux(GPIO_PORT_A, GPIO_BIT_3, PORTA3_FUNC_UART1_TXD);
    uart_init(UART1, BAUD_RATE_115200); 
    NVIC_EnableIRQ(UART1_IRQn);    
    
    if(__jump_table.system_option & SYSTEM_OPTION_ENABLE_HCI_MODE)
    {
        /* use PC4 and PC5 for HCI interface */
        system_set_port_pull(GPIO_PA4, true);
        system_set_port_mux(GPIO_PORT_A, GPIO_BIT_4, PORTA4_FUNC_UART0_RXD);
        system_set_port_mux(GPIO_PORT_A, GPIO_BIT_5, PORTA5_FUNC_UART0_TXD);
    }

    /* used for debug, reserve 3S for j-link once sleep is enabled. */
    if(__jump_table.system_option & SYSTEM_OPTION_SLEEP_ENABLE)
    {
        co_delay_100us(10000);
        co_delay_100us(10000);
        co_delay_100us(10000);
    }
}
```

user_entry_after_ble_init用于配置蓝牙协议栈，用户任务的创建，定时器的创建等等。

```c
void user_entry_after_ble_init(void)
{
    co_printf("BLE Peripheral\r\n");

    user_task_init();
    user_timer_init();
    simple_peripheral_init();
}
```

在执行完以上三个函数之后，如果使能了系统睡眠模式，LIB 中的主代码会判断是否满足进入睡眠条件，针对开始睡眠前和唤醒后分别提供了入口供用户进行自定义系统行为。

睡眠前会执行user_entry_before_sleep_imp函数，睡眠唤醒后执行user_entry_after_sleep_imp函数。

用户可以在user_entry_before_sleep_imp函数中实现GPIO的状态保持等。

```c
__attribute__((section("ram_code"))) void user_entry_before_sleep_imp(void)
{
}
```

用户可以user_entry_after_sleep_imp函数中在重新进行外设的初始化，因为进入睡眠模式之后，外设的状态都会因为调用而丢失。

```c
__attribute__((section("ram_code"))) void user_entry_after_sleep_imp(void)
{
    /* set PA2 and PA3 for AT command interface */
    system_set_port_pull(GPIO_PA2, true);
    system_set_port_mux(GPIO_PORT_A, GPIO_BIT_2, PORTA2_FUNC_UART1_RXD);
    system_set_port_mux(GPIO_PORT_A, GPIO_BIT_3, PORTA3_FUNC_UART1_TXD);
    
    system_sleep_disable();

    if(__jump_table.system_option & SYSTEM_OPTION_ENABLE_HCI_MODE)
    {
        system_set_port_pull(GPIO_PA4, true);
        system_set_port_mux(GPIO_PORT_A, GPIO_BIT_4, PORTA4_FUNC_UART0_RXD);
        system_set_port_mux(GPIO_PORT_A, GPIO_BIT_5, PORTA5_FUNC_UART0_TXD);
        uart_init(UART0, BAUD_RATE_115200);
        NVIC_EnableIRQ(UART0_IRQn);

        system_sleep_disable();
    }

    uart_init(UART1, BAUD_RATE_115200);
    NVIC_EnableIRQ(UART1_IRQn);

    // Do some things here, can be uart print

    NVIC_EnableIRQ(PMU_IRQn);
}
```

#### 任务的创建和消息的传递

如果学习过RTOS的使用，那么FR8016H的使用会非常书序，采用的任务方式，可以通过发送消息的方式，进行任务的处理，支持不同类型参数的传递。

头文件路径：`components\modules\os\include\os_task.h`

在创建任务之前，先定义一个变量，用于保存任务ID：

	uint16_t task1_id;

使用os_task_create创建一个任务，如果创建成功，会返回一个任务ID，返回0xFF表示用于：

```c
task1_id = os_task_create(task1_fun);
if(task1_id != 0xFF)
    co_printf("task 1 create ok: %d\r\n", task1_id);
```

os_task_create支持最多20个任务的创建，任务不分优先级。消息按抛送的顺序进行处理。

任务的执行函数：

```c
static int task1_fun(os_event_t *event)
{
    uint16_t id = event->event_id;	//获取消息ID
    int *param;
    
    /* 读取传递的参数 */
    param = event->param;

    switch(id)
    {
        case 12:
            co_printf("id: %d, param: %d\r\n", id, param);
            break;
        default: break;
    }

    co_printf("task1 trigger\r\n");
    return EVT_CONSUMED;
}
```

消息的发送。

定义一个消息体：

```c
os_event_t event;
```

定义要传递的参数：

```c
uint8_t param = 109;
```

消息体的初始化：

```c
event.event_id = 12;		//消息的ID号
event.param = &param;
event.param_len = sizeof(param);
event.src_task_id = task1_id;	//任务的ID号
```

发送一条消息：

```c
//向task1发送一条消息
os_msg_post(task1_id, &event);
```

也可以传递一个结构体类型的变量。定义一个结构体：

```c
struct NCOV_DATA
{
    uint16_t data1;
    uint16_t data2;
    char *data3;
    uint8_t data4;
};
```

结构体变量的初始化：

```c
struct NCOV_DATA param;

param.data1 = 111;
param.data2 = 222;
param.data3 = "hello event!";
param.data4 = 124;
```

消息体的初始化：

```c
event.event_id = 12;		//消息的ID号
event.param = &param;
event.param_len = sizeof(param);
event.src_task_id = task1_id;	//任务的ID号
```

发送一条消息：

```c
//向task1发送一条消息
os_msg_post(task1_id, &event);
```

任务处理函数中消息的读取：

```c
static int task1_fun(os_event_t *event)
{
    uint16_t id = event->event_id;
    struct NCOV_DATA *param;
    
    param = event->param;

    switch(id)
    {
        case 12:
            co_printf("data1:%d, data2:%d, data3: %s, data4: %d\r\n",
                param->data1, param->data2, param->data3, param->data4);
            break;
        default: break;
    }

    co_printf("event trigger\r\n");
    return EVT_CONSUMED;
}
```

#### 软件定时器的使用

头文件路径：`components\modules\os\include\os_timer.h。`

定义一个软件定时器：

```c
os_timer_t timer;  
```

设置定时时间，单位为毫秒ms，绑定回调函数，并启动定时器：

```c
os_timer_init(&timer, timer_fun, NULL);
os_timer_start(&timer, 1000, true);
```

回调函数的实现：

```c
void test_timer(void *parg)
{
    co_printf("1s timer");
}
```

定时器的停止：

```c
os_timer_stop(&timer);
```

#### 硬件定时器的使用

FR8016H共有两个定时器TIMER0/1，单次最大定时时间：

![2020-07-12_182608](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200712/2020-07-12_182608.jpg)



配置定时器定时时间和中断模式：

```c
timer_init(TIMER0, 10000, TIMER_PERIODIC); 	//10ms单次中断
```

使能定时器中断：

```c
NVIC_EnableIRQ(TIMER0_IRQn);
```

启动定时器：

```c
timer_run(TIMER0);
```

定时器中断服务函数：

```c
__attribute__((weak)) __attribute__((section("ram_code"))) void timer0_isr_ram(void)
{
    timer_clear_interrupt(TIMER0);
    co_printf("timer0 enter interrrupt\r\n");
}
```

#### 串口的使用

FR8016H的每个GPIO都有非常的复用功能，使用起来非常灵活：

![2020-07-12_182944](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200712/2020-07-12_182944.jpg)

把PD4配置成串口0的RXD，PD5配置成串口1的TXD模式：

```c
system_set_port_mux(GPIO_PORT_D, GPIO_BIT_4, PORTD4_FUNC_UART0_RXD);
system_set_port_mux(GPIO_PORT_D, GPIO_BIT_5, PORTD5_FUNC_UART0_TXD);
```

配置串口0的波特率：

```c
uart_init(UART0, BAUD_RATE_115200);  
```

使能串口0中断：

```c
NVIC_EnableIRQ(UART0_IRQn);   
```

串口0的中断服务函数：

```c
#define RX_BUF_SIZE     2000

uint8_t rx_buf[RX_BUF_SIZE];
uint16_t rx_sta = 0;

__attribute__((weak)) __attribute__((section("ram_code"))) void uart0_isr_ram(void)
{
    uint8_t int_id;
    uint8_t c;
    volatile struct uart_reg_t *uart_reg = (volatile struct uart_reg_t *)UART0_BASE;
    int_id = uart_reg->u3.iir.int_id;
    if(int_id == 0x04 || int_id == 0x0c )   
    {
        c = uart_reg->u1.data;
//        uart_putc_noint(UART0,c);	//数据回传
        rx_buf[rx_sta++] = c;	//读取接收
    }
    else if(int_id == 0x06)
    {
        volatile uint32_t line_status = uart_reg->lsr;
    }
}
```

阻塞方式从串口接收：

```c
void uart_read(uint32_t uart_addr, uint8_t *buf, uint32_t size);
```

阻塞方式串口发送：

```c
void uart_write(uint32_t uart_addr, const uint8_t *bufptr, uint32_t size);
```

串口发送一个字节并等待发送完成：

```c
void uart_putc_noint(uint32_t uart_addr, uint8_t c);
```

串口发送字符串并等待完成：

```c
void uart_put_data_noint(uint32_t uart_addr, const uint8_t *d, int size);
```

基于串口发送一个字节，我们可以自己实现一个串口printf函数：

```c
#include <stdarg.h>
#include <string.h>
#include <stdio.h>

void LOG(char *fmt,...)
{
	unsigned char UsartPrintfBuf[296];
	va_list ap;
	unsigned char *pStr = UsartPrintfBuf;
	
	va_start(ap, fmt);
	vsnprintf((char *)UsartPrintfBuf, sizeof(UsartPrintfBuf), fmt, ap);	//格式化
	va_end(ap);
	
	while(*pStr != 0)
		uart_putc_noint(UART0, *pStr++);//通过串口0发送出去
}
```

#### 片上LDO的配置

设置芯片自带LDO的输出电压，以及GPIO管脚高电平时的电压：

```c
void pmu_set_aldo_voltage(enum pmu_aldo_work_mode_t mode, enum pmu_aldo_voltage_t value);
```

设置芯片LDO管脚输出电压为电池电压：

```
pmu_set_aldo_voltage(PMU_ALDO_MODE_BYPASS, PMU_ALDO_VOL_3_3);
```

设置芯片LDO管脚输出电压为3.3v：

```c
pmu_set_aldo_voltage(PMU_ALDO_MODE_NORMAL, PMU_ALDO_VOL_3_3);
```

#### 片上LED2管脚的配置

LD2管脚输出0：

```c
pmu_set_led2_value(0);
```

LED2管脚输出1：

```c
pmu_set_led2_value(1);
```

不知道是LED2管脚输出的电压太低，还是电流太小，导致外部的LED点不亮，但是同一个管脚控制的LCD背光是可以点亮的，手册中描述的最好是采用灌电流的方式驱动。

![2020-07-12_184211](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/200712/2020-07-12_184211.jpg)