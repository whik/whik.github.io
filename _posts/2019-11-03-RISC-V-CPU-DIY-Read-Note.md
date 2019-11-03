---
layout:     post
title:   《手把手教你设计CPU——RISC-V处理器》读书笔记
subtitle:	读书笔记
date:       2019-11-03 23:47:40 +0800
author:     Wang Chao
header-img: img/bg3.jpg
catalog:    true
tag:
    - 读书笔记
---

> Stay Hungry, Stay foolish（求知若饥，虚心若愚）——Steven Jobs（史蒂夫-乔布斯）

### 关于书籍和胡振波

首先感谢面包板社区提供这本**《手把手教你设计CPU——RISC-V处理器篇》**书籍的试读机会。这本书和另外一本**《 RISC-V架构与嵌入式开发 》**是国内最先出版的两本关于RISC-V处理器的书籍，作者是胡振波先生，所以这里要感谢胡老师。胡振波先生是国内最早开始研究RISC-V架构的，有超过8年的CPU以及超过10年的ASIC设计与验证经验，历任Marvell CPU高级设计工程师，Synopsys ARC系列处理器内核研发经理等职务，有着近20年的行业积累。

 ![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/RISC-V_DIY_Book_Read_Note/RISC_V_DIY_Book.jpg) 

第一次了解作者，是在今年4月20日在北航举办的[RISC-V技术沙龙](http://www.wangchaochao.top/2019/04/27/ESBF/)，那是我第一次全面的了解RISC-V架构，也是我第一次参与这种线下的技术交流活动，当时很多业内的大佬都分享了很多知识和见解，其中就包括胡振波老师分享的“RISC-V架构嵌入式开发的特点”。


记得在发言结束后，会上有一位与非网的朋友提问到，“作为国内RISC-V处理器研究的领导者，芯来科技为什么没有选择做芯片而是做IP核呢？”胡振波给出的答案是，“国内现在已经有近2000家芯片公司，如果我们选择做芯片，**只是众多芯片公司中的2000分之一**，现在基本是国外公司SiFive在做基于RISC-V架构的IP，国内公司对底层技术掌握的很少，本土的公司能做IP的也很少，如果没有人来做IP，就会变成从ARM垄断的ARM架构市场转为SiFive垄断的RISC-V架构的市场，我们放弃做芯片，专注做IP，服务国内其它商业公司。虽然芯来科技从创立到现在只有半年的时间，我们开发的IP已经导入国内很多龙头公司的产品中，这样做最终为本土产业带来更大的帮助，这种选择大于我们做芯片的意义。我们选择了看似不是被人理解的方向，是为了更好帮助本土IC产业发展，算是间接实现个人的自我价值。”当时，就很佩服胡老师作为第一个吃螃蟹的人。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/ESBF_190420/%E8%8A%AF%E6%9D%A5_%E8%83%A1%E6%8C%AF%E6%B3%A2.jpg)

### 关于RISC-V

> RISC-V（发音同“risk-five”）是一种免费开源指令集架构(ISA)，通过开放标准协作开创处理器创新的崭新纪元。RISC-V基金会创立 于2015年，由超过235家成员组织组成，建立了首个开放、协作的软硬件创新者社区，开创了处理器创新的新时代。RISC-V ISA发端于深厚的学术研究，将免费且可扩展的软硬件架构自由度提升至新的水平，为未来50年的计算设计与创新铺平了道路。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/ESBF_190420/RISC-V1.png)

现在处理器的指令集主要分为RISC和CISC，即精简指令集和复杂指令集，RISC的代表就是著名的ARM架构，专注于高性能，低功耗，小体积，主要应用于移动设备；而CISC的代表是x86架构，像常用的PC、服务器的CPU等等，专注于桌面，高性能和民用市场。而RISC-V是属于RISC阵营的，相比于ARM，RISC-V的历史很短，2010年诞生于加州大学伯克利分校，当时的Krste Asanovic教授希望寻找一个合适的CPU指令架构，但x86架构复杂臃肿、ARM架构需要授权费、开源的OpenRISC架构又太老旧了，所以他最终决定自己做个开源CPU架构，并在2015年成立了[RISC-V基金会(RISC-V Foundation )](https://riscv.org/)，专门推动RISC-V发展，现在的[RISC-V基金会成员](https://riscv.org/members-at-a-glance/)也超过了235个，包括国外的Google、三星、英伟达、微芯、高通、惠普、意法半导体、西数、NXP，国内的阿里巴巴、华为、高云等公司。**一个时代有一个时代的芯片**。RISC-V，就被认为是AIoT时代芯片的基础。可以说，今年是RISC-V快速发展的一年，从最近的兆易发布GD32VF103 MCU，到阿里发布玄铁910 IP，再到华米科技基于RISC-V MCU的智能手表量产等等，都可以看出今年RISC-V是大放异彩，个人猜测，ARM可能觉得RISC-V对他产生了一定威胁，所以计划将在明年上半年在特定CPU内核上推出**自定义指令集功能**，来对抗RISC-V！

正好最近又发布了基于兆易和芯来共同研发的 **Bumblebee RISC-V内核**的MCU——**GD32VF103** 。不知道最近面包板社区有没有考虑开展兆易RISC-V开发板的评测活动，哈哈！ 

### 关于蜂鸟E200

本书介绍的这款RISC-V CPU内核，名称为蜂鸟E200，代码文档全部开源在Github上，开源地址在文末。蜂鸟E200是一个处理器系列，包含了多款不同的具体处理器型号。所有的E200系列处理器核均支持协处理器接口，可用于自定义扩展指令。E201核是面积最小的核，可以配置为RV32IC或者RV32EC架构，不支持其他的扩展指令子集。

- E201核是面积最小的核，可以配置为RV32IC或者RV32EC架构，不支持其他的扩展指令子集。
- E203核可以配置为RV32IMAC或者RV32EMAC架构，使用面积优化的多周期硬件乘除法单元。
- E205核为RV32IMAC架构，使用单周期的硬件乘法单元和多周期的硬件除法单元。
- E205f核为RV32IMAFC架构，在基本的E205基础上加入了单精度浮点运算单元。
- E205fd核为RV32IMAFDC架构，在基本的E205基础上加入了单精度和双精度浮点运算单元。

针对商业用途，芯来科技又发布了商业内核——N200系列。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/RISC-V_DIY_Book_Read_Note/N200_Core.jpg)

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/RISC-V_DIY_Book_Read_Note/N200_Core_Series.jpg)

(来自官网 [https://www.nucleisys.com](https://www.nucleisys.com/) )

### 全书结构

全书共20章节，可分为三大部分，第一部分1-4章，普及处理器、CPU、指令集、内核、架构、RISC-V基础知识，并介绍了多款RISC-V内核，以及蜂鸟E200内核系列。

第二部分5-16章，详细介绍了设计CPU的通用流程，及对应的Verilog代码实现，非常适合深入理解CPU内部的工作原理，总线，指令，译码，中断异常，调试等，至于TIMER，UART、IIC、SPI等接口属于外设部分，并没有提到，毕竟本书介绍的CPU部分的实现过程，就像ARM MCU一样，内核都是一个，但是不同的半导体厂家，外设配置是不同的，有的是2个串口，有的3个串口，有了总线接口，这些外设根据需要来自主设计添加就行了。

第三部分17-20章，介绍在FPGA上实现CPU原型，并使用IDE工具来进行开发和调试，当然更详细的应用开发，还是要参加另一本姊妹书籍——《 **RISC-V架构与嵌入式开发** 》，最后一章节，介绍了和ARM M系列的性能跑分对比。

**蜂鸟E200ARM Cortex-M核性能对比**

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/RISC-V_DIY_Book_Read_Note/ARM_E200.jpg)

#### **国产CPU**

- MIPS——龙芯和君正
- x86系——北大众志、兆芯、海光
- Power——中晟宏芯
- Alpha——申威
- ARM——飞腾、华为海思、展讯、华芯通

#### CISC和RISC的区别

指令集架构主要分为复杂指令集（Complex Instruction Set Computer，CISC）和精简指令集（Reduced Instruction Set Computer，RISC），两者的主要区别如下：CISC不仅包含了处理器常用的指令，还包含了许多不常用的特殊指令。其指令数目比较多，所以称为复杂指令集。RISC负包贪处理器常用的指令，而对于不常用的操作，则通过执行多条常用指令的方式来达到回样的效果。由于其指令数目比较精简，所以称为精简指令集。典型程序的运算过程中所使用到的80%指令，只占所有指令类型的20%，也就是说，CISC指令集定义的指令，只有20%被经常使用到，而有80%则很少被用到。那些很少被用到的特殊指令尤其让CPU设计变得极为复杂，大大增加了硬件设计的时间成本与面积开销。

#### RISC-V商业版本与开源版本

- Rocket Core 
- BOOM Core
- Freedom SoC
- LowRISC SoC
- PULPino Core and SoC
- PicoRV32 Core
- SCR1 Core
- ORCA Core
- Andes Core（商业IP）
- Microsemi Core（商业IP）
- Codasip Core（商业IP）
- 蜂鸟E200 Core与SoC（开源）

### 配套源码

本书配套的Verilog代码Github开源地址：https://github.com/whik/e200_opensource.git`

- Git下载命令：`git clone https://github.com/whik/e200_opensource.git`

如果下载速度太慢，我已经把这个仓库同步到了我个人的Gitee码云上，下载速度会快一些：https://github.com/whik/e200_opensource

- Git下载命令：`git clone https://github.com/whik/e200_opensource.git`

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/RISC-V_DIY_Book_Read_Note/e200_github.jpg)

### 配套开发板购买

开发板基于Xilinx XC7A100T，板载Xilinx Platform Cable USB下载器，用于对FPGA进行程序烧写。

- FPGA评估板和JTAG调试器购买链接：https://item.taobao.com/item.htm?id=580813056318
- 蜂鸟RISC-V开源处理器软硬件演示视频：https://www.bilibili.com/video/av41835638
- 作者胡振波先生演讲——“RISC-V架构嵌入式开发的特点”，直播回看：[RISC-V架构嵌入式开发研究与实践](https://live.maodou.io/course/j8Guikogngo9mY9nD?from=timeline&isappinstalled=0)
- 作者ID：**硅农亚历山大**，微信公众号，CSDN博客等平台。

### 总结

这本书主要是介绍国产开源RISC-V架构CPU——蜂鸟E200，通用CPU的设计流程和基于Verilog的代码具体实现，可以说是理论和实践相结合的一本好书，代码和文档都在Github上开源，文末有地址。无论是对于嵌入式开发，还是IC设计验证，都是很有价值的参考。虽然日常工作中也会接触到一些Verilog FPGA开发，但都是一些采集和通信的简单程序，另外个人也并不是微电子专业的，所以对于这种CPU的设计和实现，只能看个大概，很多的设计技巧理解起来还是有些困难，所以我的读书心得也仅限于此了，欢迎各位朋友互相交流。

### 推荐阅读

- [详解串行通信协议及其FPGA实现](http://www.wangchaochao.top/2019/08/23/UART-Simple/)
-  [Jlink使用技巧系列教程索引](http://www.wangchaochao.top/2019/01/17/Jlink-series/) 
-  [手把手教你制作Jlink-OB调试器（含原理图、PCB、外壳、固件）](http://www.wangchaochao.top/2019/05/10/Open-JlinkOB/) 
-  [两行代码搞定博客访问量统计](http://www.wangchaochao.top/2018/10/15/Busuanzi/) 

----

- 我的博客：www.wangchaochao.top
- 我的公众号

