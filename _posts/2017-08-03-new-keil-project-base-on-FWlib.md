---
layout:     post
title:      STM32F4 新建标准库函数工程
subtitle:	  建好工程，代码撸起！！！
date:       2017-8-3
author:     Shao Guoji
header-img: img/post-bg-new-FWLib-project.jpg
catalog:    true
tag:
    - 单片机
    - STM32
    - 嵌入式
---

### 前言

在 STM32 开发中，库函数开发相比寄存器方式具有开发周期短、代码可读性好、便于移植等优点，而使用 Keil 环境的第一步就是新建工程。本文以 STM32F401CE 芯片为例，介绍使用标准库函数新建工程的步骤。

---

<br/>

### 材料准备

 - STM32F4xx 固件库：[STM32F4xx_DSP_StdPeriph_Lib_V1.4.0.rar](http://pan.baidu.com/s/1i4LQmg9) 
 - Keil-MDK
 - 一点耐心

---

<br/>

### 新建库函数工程注意事项

不同芯片在新建工程时的配置略有区别，主要体现在以下几点：

 1. 工程目标 Device 选择的芯片型号不同。
 2. 添加的启动文件不同。要根据芯片型号在 `arm` 目录下选择相应的 `.s` 文件。
 3. C/C++ 选项卡的芯片型号宏定义不同。具体有哪些选择可在 `stm32f4xx.h` 头文件中的条件编译指令中找到。不确定选哪个的话可以根据芯片主频从 `system_stm32f4xx.c` 文件的 PLL 分频参数反推宏定义（要求对时钟树比较熟）。
 4. 工程所包含的外设库函数不同。MDK 会在编译时根据芯片型号宏定义进行寄存器映射，所以要**对芯片所没有的外设库文件要排除编译（如文中的 fmc 和 fsmc），否则会报标识符未定义错误**。

#### 补充：如何确定芯片有无某个外设？

芯片数据手册中描述了芯片的所有资源，**当想要了解具体某一型号芯片的外设时，应该查阅数据手册而不是参考手册**（参考手册针对的是整个系列芯片的通用说明）。

---

<br/>

## Cortex-M4 新建库函数工程步骤

### 一、新建工程文件夹

 1. 新建一个文件夹 `template` 用于存放工程模板。
 2. 在 `template` 文件夹内分别新建 `Doc`（存放文档）、`User`（存放用户文件）两个文件夹。
 3. 在 `User` 文件夹中新建 `inc` 和 `src` 两个文件夹分别存放用户头文件和源文件。

---

<br/>

### 二、复制库函数文件

#### 1. 复制 Libraries 文件夹

打开固件库目录 `STM32F4xx_DSP_StdPeriph_Lib_V1.4.0`， 复制 `Libraries` 文件夹到工程文件夹 `template` 中。

#### 2. 裁剪 Libraries 文件夹

由于固件库的 `Libraries` 文件夹是对整个 Cortex-M4 系列通用的，包含了一些项目所用不到的文件，为了节约空间，可以把用不到的多余文件删除。 

 - 删除 `template\Libraries\CMSIS` 目录下除 `Device` 和 `Include` 外的所有文件夹。
 - 删除 `template\Libraries\CMSIS\Device\ST\STM32F4xx\Source\Templates` 目录下除 `arm` 文件夹和 `system_stm32f4xx.c` 外的所有文件夹。
 - 删除 `template\Libraries\CMSIS\Device\ST\STM32F4xx\Source\Templates\arm` 目录下除 `startup_stm32f401xx.s` 外的其余启动文件。
 - 固件库里类似 `Release_Notes.html` 的说明文件也可以删了。

#### 3. 添加文件到 User 文件夹

往 `template\User` 目录中添加以下三个库文件，并新建 `main.c` 文件。

 - 打开 `STM32F4xx_DSP_StdPeriph_Lib_V1.4.0\Project\STM32F4xx_StdPeriph_Templates` 目录，找到 `stm32f4xx_conf.h`、`stm32f4xx_it.h`、`stm32f4xx_it.c` 三个文件。
 - 复制 `stm32f4xx_conf.h` 文件到 `template\User\inc` 目录。
 - 复制 `stm32f4xx_it.h` 文件到 `template\User\inc` 目录。
 - 复制 `stm32f4xx_it.c` 文件到 `template\User\src` 目录。
 - 在 `template\User\src` 目录中新建 `main.c` 文件，并编写以下主函数代码保存。

```c
#include "stm32f4xx.h"

int main(void)
{
	
    while (1);
}
```

其中 `stm32f4xx_conf.h` 文件包含了所有库函数头文件，`stm32f4xx_it.c` 文件用于编写中断服务函数，便于统一管理，使工程结构更加规范。

完成以上工作后整个工程目录结构如下图：

![图1 库函数工程文件结构][1]

---

<br/>

### 三、Keil 新建工程

把工程文件夹建好并复制相关库文件后，就可以打开 Keil 软件新建工程了。

####  1. 新建工程

打开 Keil 软件，点击 Project 菜单下的 New uVision Project 选项新建工程，并保存到新建的工程文件夹 `template` 中。

 ![图2 新建工程菜单][2]
 
 ![图3 保存工程到文件夹][3]
 
####  2. 选择芯片型号

在弹出的芯片选型窗口中选择目标板的芯片型号，这里选 STMicroelectronics（ST公司）下的 STM32F401CE（根据具体硬件选择）。

![图4 选择芯片型号][4]

关掉弹出的组件添加窗口，因为我们采用的是手动添加库文件的方式。

![图5 关闭组件窗口][5]

####  3. 新建工程分组

点击「品字形」按钮打开工程管理界面，最左侧是工程名称，可以给工程改个名，然后在中间组管理中点虚框图标新建 CMSIS、STM32F4xx_StdPeriph_Driver、USER、DOC 四个分组，对应工程文件夹中的分类。

![图6 新建工程分组][6]

####  4. 添加文件到分组

依然是在工程管理界面下，这一步要做的事情是把准备好的库文件添加到 Keil 工程中，具体操作如下：

 - 往分组 CMSIS 中添加 `system_stm32f4xx.c` 系统配置文件和 `startup_stm32f401xx.s` 启动文件。

![图7 添加 CMSIS 分组的文件][7]

 - 往分组 STM32F4xx_StdPeriph_Driver 中添加 `Libraries\STM32F4xx_StdPeriph_Driver\src` 目录下的所有 `.c` 文件。

![图8 添加 STM32F4xx_StdPeriph_Driver 分组的文件][8]

 - 往分组 USER 中添加文件 `main.c` 和 `stm32f4xx_it.c`。

![图9 添加 USER 分组的文件][9]

---

<br/>

### 四、工程配置

点击「魔术棒」按钮打开工程选项界面，进行必要的工程配置。

####  1. Target 选项卡配置

 - 勾选 Use Micro LIB 选项，为了在工程中使用 printf() 函数。

![图10 Target 选项卡配置][10]

####  2. Output 选项卡配置

 - 如需生成 `.hex` 文件，则需勾选 Create HEX File 选项。

![图11 Output 选项卡配置][11]

####  3. C/C++ 选项卡配置

 - 在「预处理符号」Preprocessor Symbols 下的 Define 一栏中添加 `STM32F401xx` 和 `USE_STDPERIPH_DRIVER` 两个宏定义（用逗号分隔）。
 - 在 Include Paths 中添加以下头文件路径，**注意要具体到头文件上一层目录**。
	 1. `.\Libraries\CMSIS\Device\ST\STM32F4xx\Include`
	 2. `.\Libraries\CMSIS\Include`
	 3. `.\Libraries\STM32F4xx_StdPeriph_Driver\inc`
	 4. `.\User\inc`

![图12 C/C++ 选项卡配置][12]

---

<br/>

### 五、下载器配置

依然是在工程选项界面下，进行下载器配置

####  Debug 选项卡配置

在右上角选择所使用的调试器，根据实际情况选择。这里我用的是 ST-Link，选 ST-Link Debugger。

![图13 Debug 选项卡配置][13]

点击 setting 按钮，在 Falsh Download 选项卡中勾选 Reset and Run 选项，确保程序下载后能自动复位运行，最后点击确定按钮保存所有的工程配置。

![图14 Falsh Download选项卡配置][14]

---

<br/>

### 六、最后小整改

此时编译整个工程依然会有大量错误，为了能使工程顺利编译最后还要稍作修改，具体如下：

####  取消编译 fmc 和 fsmc 库文件

查阅数据手册得知 STM32F401 系列芯片没有 fmc 和 fsmc 外设，所以去掉 fmc 和 fsmc 部分的库文件。

在工程文件视图下展开 STM32F4xx_StdPeriph_Driver 分组，选中 `stm32f4xx_fmc.c` 文件，右键调出 Options for File stm32f4xx_fmc.c 窗口，取消勾选  Include in Target Build 选项，排除 `stm32f4xx_fmc.c` 文件参与编译。`stm32f4xx_fsmc.c` 用文件同样方式处理（或者直接从工程中移除这两个文件）。

![图13 取消 fmc 和 fsmc 文件加入编译][15]

####  修改 stm32f4xx_it.c 文件

 1. 删除 `stm32f4xx_it.c` 文件 32 行的代码 #include "main.h"（因为没有写这个文件）。
 2. 删除 `stm32f4xx_it.c` 文件 144 行的代码 TimingDelay_Decrement();（滴答定时器延时相关，暂时用不到）。

---

<br/>

### 七、编译工程、下载验证

点击编译按钮，如果工程配置正确就会看到令人愉悦的 0 Error(s), 0 Warning(s) ，通过下载器连接板子和电脑，烧写程序检验成果。

---

<br/>
<br/>

> 参考文章
> 
> * 《零死角玩转STM32—F429》- 秉火


  [1]: http://odaps2f9v.bkt.clouddn.com/17-8-27/17939139.jpg
  [2]: http://odaps2f9v.bkt.clouddn.com/17-8-3/35624276.jpg
  [3]: http://odaps2f9v.bkt.clouddn.com/17-8-3/61896201.jpg
  [4]: http://odaps2f9v.bkt.clouddn.com/17-8-3/53271432.jpg
  [5]: http://odaps2f9v.bkt.clouddn.com/17-8-3/99203621.jpg
  [6]: http://odaps2f9v.bkt.clouddn.com/17-8-3/39960355.jpg
  [7]: http://odaps2f9v.bkt.clouddn.com/17-8-7/15510918.jpg
  [8]: http://odaps2f9v.bkt.clouddn.com/17-8-3/72873096.jpg
  [9]: http://odaps2f9v.bkt.clouddn.com/17-8-3/72998181.jpg
  [10]: http://odaps2f9v.bkt.clouddn.com/17-8-3/83732194.jpg
  [11]: http://odaps2f9v.bkt.clouddn.com/17-8-3/29121373.jpg
  [12]: http://odaps2f9v.bkt.clouddn.com/17-8-3/60182548.jpg
  [13]: http://odaps2f9v.bkt.clouddn.com/17-8-3/99970592.jpg
  [14]: http://odaps2f9v.bkt.clouddn.com/17-8-3/83679862.jpg
  [15]: http://odaps2f9v.bkt.clouddn.com/17-8-3/17270014.jpg