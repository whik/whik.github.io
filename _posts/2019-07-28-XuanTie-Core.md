---
layout:     post
title:    玄铁910是个啥？是芯片吗？
subtitle:	 阿里平头哥首次交货
date:       2019-07-28 10:55:40 +0800
author:     Wang Chao
header-img: img/Ali_Pingtouge.jpg
catalog:    true
tag:
    - 行业资讯
    - RISC-V
---

![](http://pics0.baidu.com/feed/f636afc379310a557d434b61e9946dac83261012.jpeg?token=04579b916adec942ecc3ce775b65fd99&s=121DA16C0558BDCC1C5E8C9E0300B09B)

### 1.平头哥首次交货

2019年7月25日，阿里云上海峰会，平头哥半导体发布新品玄铁910，最高支持16核，2.5GHz，7.1 Coremark/MHz。阿里平头哥，走出了万里长征第一步。


玄铁910的研发绝对不是一蹴而就，其前身中天微自研开发的CK801、CK802、CK803、CK805、CK807、CK810、CK860等7款嵌入式CPU IP核，均已得到大规模量产的验证，授权客户超100家，被广泛应用于机器视觉、工业控制、车载终端、移动通信、多媒体、无线接入和信息安全等领域，主要客户均是一线芯片设计公司。2018年9月，中天微发布第一款基于RISC-V的IP核，CK902处理器，基于RISC-V第三代指令系统架构。可以说，玄铁910的发布和之前的积累是分不开的。

![](https://media.bjnews.com.cn/cover/2019/07/25/4818521426573208294.jpg)

阿里巴巴作为RISC-V基金会的[铂金会员](https://riscv.org/membership/?action=viewlistings)，但是在RISC-V基金会官网上并没有看到CK902和玄铁910这两款IP核的相关信息：

RISC-V基金会Github：[RISC-V Cores & SoC platforms & SoCs List](https://github.com/riscv/riscv-cores-list)

还是说，基金会上公布的只是开源的Core，而这两款IP核目前还没开源，所以没在上面显示，这一点我不了解。

### 2.玄铁910到底是个啥？是芯片吗？

从官方发布会现场介绍，**这是一款RISC-V架构处理器**，这个处理器的意思并不是指的电脑上的Intel/AMD处理器，手机上的高通/麒麟处理器的那个意思，这是一种IP Core，以华为麒麟980芯片为例，它是四核Cortex-A76+四核Cortex-A55，这里的A76和A55，和玄铁910是一个意思。

玄铁910与ARM v8的高性能处理器Cortex-A72，处于同一水平。高通骁龙618（MSM8956）和骁龙620（MSM8976）就是基于Cortex-A72架构核心。官方表示，未来不久，平头哥将开放玄铁910 IP Core，全球开发者可免费下载处理器的FPGA代码，开展芯片原型设计架构创新。所以，**玄铁910只是在FPGA上实现的RISC-V软核**。

关于架构、IP、处理器、SoC的区别和联系，可以参考知乎回答：[ARM 架构、ARM7、STM32、Cortex M3......之间有什么区别和联系？陈俊直的回答](https://www.zhihu.com/question/22464046/answer/21450143)。

### 3.还有哪些RISC-V的Core、SoC或开发板

基于RISC-V的 Core 或 SoC，可以到RISC-V基金会官方网站查询：[RISC-V Core & SoC PLATFORMS & SoCs](https://riscv.org/risc-v-cores/)

ARM处理器按位数可分为32位和64位，按架构又可分为Cortex-A/Cortex-R/Cortex-M三大系列，分别针对不同的应用领域。而RISC-V也有很多内核，截至2019年06月22日RISC-V社区显示的Core共有29个，SOC PLATFORM共有13个，SOC共有7个。

部分RISC-V Core

![](https://pic4.zhimg.com/80/v2-d85074b928b00ed623223cdee98816d3_hd.jpg)

RISC-V SoC

![](https://pic1.zhimg.com/80/v2-8300bca7fc8cc77923e64634bba67200_hd.jpg)

基于FPGA实现的RISC-V开发板

- Perf-V：国内[澎峰科技](http://perfxlab.com)出品，基于Xilinx Artix-7系列FPGA-XC7A50T实现
- 蜂鸟E203：国内[芯来科技](http://www.nucleisys.com/)出品，基于Xilinx Artix-7系列FPGA-XC7A75T，可以配置为RV32IC或RV32EC架构。
- 小脚丫STEP开发板：国内[思得普科技](https://www.stepfpga.com/)出品，基于Intel公司Cyclone 10系列FPGA芯片10CL016YU256C8G
- 小脚丫STEP-MXO2 二代FPGA开发板：基于Lattice公司MXO2系列的FPGA芯片LCMO2-4000HC-4MG132

基于RISC-V芯片实现的开发板

- NXP VEGA织女星开发板：[NXP恩智浦](https://www.nxp.com/cn/)出品，基于NXP四核异构处理器，片上集成两个ARM内核和两个RISC-V内核，板载资源丰富。目前还买不到这块开发板和RV32M1芯片，想体验这块RISC-V板子的朋友，可以免费申请：[NXP恩智浦VEGA织女星开发板免费申请！](http://www.wangchaochao.top/2019/05/22/Vega-Lite/)官方社区：[OPEN-ISA社区](https://open-isa.cn)
- Sipeed M1(荔枝丹)开发板：基于K210 RISC-V芯片，K210是国内[嘉楠耘智](https://kendryte.com/)的团队在2018年研发出一款7nmAI芯片，采用了基于rocket-chip的双核RV64GCC RISC-V CPU。
- Lichee Tang(荔枝糖)开发板：基于国产FPGA芯片EG4S20，EG4S20是国内[安路科技](http://www.anlogic.com/)开发设计的FPGA芯片，20K逻辑单元，130KB SRAM。

### 4.这么多RISC-V处理器，为什么玄铁910这么引人关注？

玄铁910引人关注的原因，不仅是因为作为平头哥首次交货，而且更重要的是性能很强悍。应用领域主要是5G、自动驾驶、人工智能等高端领域。

![](http://pics6.baidu.com/feed/0d338744ebf81a4cd378e47888fb4e5c242da60b.jpeg?token=8a9a9660262dd8e595a15847c99ca9b7&s=921DE16C03F0B7CA5C7E1C100300109B)

但可以看得出，阿里巴巴之所以想把希望寄托给RISC-V，一方面是出于成本的考虑，另一方面则是受到华为事件的
影响，期望更加自主可控。据发布会介绍，玄铁910支持16核，单核性能可达7.1 Coremark/MHz。但是，我并没有在Coremark官网上找到玄铁的跑分记录。

各处理器Coremark分数查询：[coremark_scores](https://www.eembc.org/coremark/scores.php)

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/CoremarkScore.jpg)

### 5.关于RISC-V

**一个时代有一个时代的芯片**。

RISC-V，就被认为是AIoT时代芯片的基础。或许你有所了解，RISC-V全称是精简指令集计算机（reduced instruction set computer），这种微处理器与之前的相比，速度更快、功耗更低。由现任谷歌董事长Hennessy和UC伯克利教授David Patterson于1980年提出，2017年还因此发明荣膺图灵奖，发展至今已演进至第五代，故而简写为RSIC-V。

与大多数指令集相比，RISC-V指令集可以自由地用于任何目的，允许任何人设计、制造和销售RISC-V芯片和软件。虽然这不是第一个开源指令集，但它展现的势能和拥戴，已有王霸之气。

一方面，因为其独特设计适用于更现代的计算设备，如仓库规模云计算机、高端移动电话和微小嵌入式系统。另一方面，RISC设计之初就考虑到了用途中的性能与功率效率。于是该指令集还具有众多支持的软件，这解决了新指令集通常的弱点。所以全世界的处理器开发者，实际上都可以基于这个架构做出理想的处理器产品。这种可扩展、可定制化的特点，对于场景驱动、性能功耗需求各不相同的AIoT芯片特别重要。门槛之外，成本也是RISC-V王霸之气的背后加速度。

### 6.相关新闻

- 华米科技RISC-V芯片“黄山1号”，应用于自家的智能手表

2018年9月17日，华米推出全球首款RISC-V开源指令集可穿戴处理器——黄山1号，主频最高240MHz，制程55nm，608KB RAM，UART/SPI/IIC/I2S/PWM/ADC/TIMER。当时介绍2019年上半年商用，如今时间过去大半年，2019年6月11日，华米发布MAZFIT米动健康手表、AMAZFIT智能手表2新品，搭载黄山1号处理器，支持心电测量等特性。这款芯片主要面向智能穿戴领域，横向和ARM Cortex-M4架构处理器相比，黄山1号的运算效率要高出38%。既然说到华米自研的“黄山1号”，绝对要提一提小米了。华为科技是小米生态链中分量非常重要的一家企业，它肩负着众多非常重大的责任。首先，华米科技是小米手环生产方，根据数据显示，小米手环4全球累计销量达到了6200万只，今年第一季度，小米手环更是以530万的出货量位列全球第一名。其次，2018年6月，华米投资了全球领先的 RISC-V 指令集架构供应商 SiFive。

- NXP推出四核RISC-V芯片RV32M1，含2个ARM核和2个RISC-V核

恩智浦的RV32M1， 四核异构：两个ARM核，两个RISC-V核，自带无线功能。开源社区[open-isa](open-isa.cn)免费开放申请织女星(VEGA)开发板，创意大赛奖品多多。

- 国内芯来科技推出“一分钱计划”

“[一分钱计划](https://www.nucleisys.com/plan.php)”是芯来科技推出了一项RISC-V处理器内核的普惠计划。芯片厂商可以从芯来科技获取入门商用级RISC-V处理器内核N201，无需付任何授权费用(License Fee)便可以用于任何商业和非商业用途的产品研发和设计，只有在后续大规模量产的时候才需要每颗芯片收取人民币一分钱的版税(Royalty)。

### 7.平头哥任重而道远

这是平头哥半导体成立之后的第一款产品，阿里芯片宏图伟业的首次交货，也是RSIC-V开源世界的最新纪录。它以剑神兵器为名，性能和价格都展现大规模竞争力，但更令人惊讶的是背后承载的普惠方案、开源逻辑，以及阿里成功之道的延续.

![](http://pics2.baidu.com/feed/adaf2edda3cc7cd9ffc47acd67d00f3ab90e910b.jpeg?token=90ec9d80fb15a662106e9db6fc24a1cf&s=BA7771848061FCE6DD0C58990300D088)

总之，RISC-V的发展，离不开完善的生态系统，IP核、芯片、IDE、开发板、文档资料、社区论坛等等。

【**本回答参考以下资料，非商业用途，侵权请联系我删除。**】

### 参考资料：

- [阿里平头哥首次交货！“让天下没有难造的芯片”](http://baijiahao.baidu.com/s?id=1640007711630275604&wfr=spider&for=pc)
- [Amazfit 全新系列新品发布会7月16日举行 搭载黄山1号](http://sznews.tetimes.com/tech/20190709/7988.html)
- [一图了解华米AI芯片黄山1号：2019年上半年商用](http://tech.ifeng.com/a/20180917/45171511_0.shtml)
- [ARM 架构、ARM7、STM32、Cortex M3......之间有什么区别和联系？陈俊直的回答](https://www.zhihu.com/question/22464046/answer/21450143)
- [阿里平头哥发布首个产品玄铁910 但这并不是CPU](http://www.bjnews.com.cn/finance/2019/07/25/607822.html)
- [又一个国产芯片崛起，阿里平头哥发布“全球性能最强处理器”！ ](http://www.sohu.com/a/329541709_766672)

### 推荐阅读：

- [国产处理器的逆袭机会——RISC-V](http://www.wangchaochao.top/2019/04/27/ESBF/)
- [NXP恩智浦VEGA织女星开发板免费申请！](http://www.wangchaochao.top/2019/05/22/Vega-Lite/)
- [真正的RISC-V开发板——VEGA织女星开发板开箱评测](http://www.wangchaochao.top/2019/06/22/VEGA-4/)
- [【2019北京国际消费电子博览会】参观总结](http://www.wangchaochao.top/2019/06/30/Beijing-CEE/)
- [Qt实现软件自动更新的一种简单方法](http://www.wangchaochao.top/2019/03/31/Qt-Update/)

--------

我的博客：www.wangchaochao.top

或微信扫码关注我的公众号
