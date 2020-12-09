---
layout:     post
title: 如何向RT-Thread提交一个BSP
subtitle:	SmartFusion移植RT-Thread
date:       2020-12-09 21:57:00 +0800
author:     Wang Chao
header-img: img/pcb_logo.jpg
catalog:    true
tag:
    - FPGA
    - ARM
---

RT-Thread今天的快速发展和所取得成绩，离不开所有开发者的持续贡献和社区小伙伴的竭力支持。

#### 前言

今年6月，我在一款智能混合型的FPGA芯片上，完成了RT-Thread的移植，并向RT-Thread提交了一个完整的BSP，后续又根据审查意见进行了一些完善，最近（11.18）被合并到RT-Thread主分支上。

如果你曾经下载过RT-Thread的源码仓库，在最常用的STM32 BSP上面的smartfusion2，这个BSP就是我提交的了，如果有读者朋友使用过这款芯片，欢迎体验，或者提交BUG。

![BSP包](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/RT-Thread/BSP包.jpg)

有的朋友可能注意到了，我这里使用的是FPGA芯片，FPGA芯片还能运行RT-Thread吗？准备的说，应该是FPGA片上的处理器可以运行RTOS，这里的处理器，从实现方式来看，包括硬核和软核处理器；从内核种类上来看，包括ARM核或其他内核，如ARM硬核，Altera的NIOS软盒，Xilinx的microblaze软核，还有51软核等，关于FPGA片上处理器，可以参考以下文章：

- [FPGA硬核和软核处理器的区别](https://mp.weixin.qq.com/s/Y2cHZ6VYFngVPwO9upr69g)
- [除了ZYNQ还有哪些内嵌ARM硬核的FPGA？](https://mp.weixin.qq.com/s/C7y9y5XCid3HxK6yKdYVwg)

此次提交的这个BSP是我第一次向开源项目贡献代码，而且是向这么优秀的国产RTOS操作系统，还是很有成就感的~本篇文章记录如何向RT-Thread或其他开源项目贡献代码，有不准确的地方欢迎大家指正，希望大家支持国产RTOS的发展！

#### RT-Thread遵循的许可协议

RT-Thread的开源协议是进行过调整的，在2018年RT-Thread官方公众号发布的一篇文章[1]中，我们可以知道当时是使用的GPLv2协议，

![GPLV2](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/RT-Thread/GPLV2.jpg)

但是现在已经是Apache-2.0协议了。

![rt-thread所遵循的开源协议](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/RT-Thread/rt-thread所遵循的开源协议.jpg)

在贡献代码之前，我们有必要先来了解一下开源项目所遵循的协议，如果你提交成功，开源协议将会约束这些代码被如何使用。从RT-Thread官方GitHub页面，我们可以了解到RT-Thread所遵循的开源协议为：**Apache-2.0 License**，这个协议有以下特点：

- **永久权利**
  一旦被授权，永久拥有。
- **全球范围的权利**
  在一个国家获得授权，适用于所有国家。假如你在美国，许可是从印度授权的，也没有问题。
- **授权免费，且无版税**
  前期，后期均无任何费用。
- **授权无排他性**
  任何人都可以获得授权
- **授权不可撤消**
  一旦获得授权，没有任何人可以取消。比如，你基于该产品代码开发了衍生产品，你不用担心会在某一天被禁止使用该代码。

有很多人认为开源就是免费，可以随意的使用，其实这个观点是错误的。如果你有自己的开源项目，关于协议的选择可以参考**黄工大佬**之前总结的[2]：[程序猿如何选择开源协议？](https://mp.weixin.qq.com/s/4zE4QN__2CSnPKJD_v6dkw)

开源协议虽然不一定具备法律效力，但是当涉及软件版权纠纷时，开源协议也是非常重要的证据之一。

#### SmartFusion2 BSP简介

这个BSP是移植 RT-Thread 操作系统到一款 **FPGA 芯片——M2S010** ，该芯片属于 Microsemi（现Microchip）SmartFusion2系列，是一款**智能混合型FPGA**，片上除了 FPGA Fabric 逻辑部分，还包括一个 **ARM® Cortex™-M3 内核的 MCU**，主频最高 166MHz ，256KB eNVM，64KB eSRAM，集成GPIO、UART、I2C、SPI、CAN、USB等基本外设。

> 关于 Microsemi，第三大 FPGA 厂商，原 Actel 半导体，2010 年，Microsemi 收购 Actel，2018 年， Microchip 收购 Microsemi。

SmartFusion2 内部框图

![Microsemi_Smartfusion2_BD](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/RT-Thread/Microsemi_Smartfusion2_BD.jpg)

移植了 RT-Thread 内核，支持线程调度、线程间同步和通信等，已经完成了PIN、Serial设备驱动，FinSH组件默认使用uart0设备。支持GPIO和UART外设，支持SCons构建系统，可以输入`scons`调用env工具中包含的arm-gcc编译器构建工程，支持以下scons命令：

- `scons`：使用arm-gcc编译BSP
- `scons -c`：清除执行 scons 时生成的临时文件和目标文件。
- `scons --target=mdk4`：重新生成Keil MDK4环境下的工程。
- `scons --target=mdk5`：重新生成Keil MDK5环境下的工程。
- `scons --dist`：打包BSP工程，包括RT-Thread源码及BSP相关工程文件。

通过添加Kconfig文件，可以使用menuconfig来配置外设，用于生成rtconfig.h。

#### 如何提交你的BSP包

**0.准备工作**

进行提交之前，需要做一些准备工作：

- 一个GitHub账号

- Git Windows客户端(git-scm.com/download/win)

- 一些基本Git命令的使用，如`git clone/add/commit/pull/push/checkout`等。


- 了解所使用处理器的启动流程，熟悉基本外设的使用，如GPIO、UART等。

**1.Fork并Clone到本地PC**

登录自己的GitHub账号，Fork RT-Thread仓库到个人仓库，Fork的意思可以理解为复制一份。

![fork](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/RT-Thread/fork.jpg)

将远程仓库下载到本地：`git clone https://github.com/username/rt-thread`，这样就会把远程代码下载到本地。

![fork](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/RT-Thread/clone.jpg)

**2.创建分支**

从 master 分支创建自己的开发分支，如`whik_sf2`，可以使用命令：`git checkout -b whik_sf2`

**3.开发你的BSP包**

这是整个开发过程中最重要，也是最耗时的一步，如果是ARM内核，可以参考STM32的移植过程，如果是其他内核，就需要多用一点时间了。

编码风格参考：https://github.com/RT-Thread/rt-thread/blob/master/documentation/coding_style_cn.md

一个最基本的BSP包，至少应该包括以下部分：

- 内核移植，支持线程调度、线程间同步和通信
- 支持GPIO/UART外设，PIN/Serial设备驱动
- 支持SCons构建系统，可以使用arm-gcc进行编译，支持生成MDK工程，支持dist打包，通过SConscript、SConstruct、rtconfig.py文件实现
- 支持menuconfig配置外设，用于生成rtconfig.h，通过Kconfig文件实现
- README文件用于指导开发者如何使用这个BSP包，可以参考其他BSP文件夹下的README文件

提交关于BSP的代码，尽量确保代码改动仅限制于BSP中，而不影响到其他代码，否则可能会被拒绝[3]。

**4.提交到远程并发起PR**

如果本地进行测试没问题，就可以同步到远程了，三部曲：`git add/commit/push `，更新到远程之后，就可以发起PR了，在 git 仓库中选择自己修改了的分支，点击 create Pull Request 按钮发起 pull request。

![image](https://www.rt-thread.org/document/site/development-guide/github/figures/pullrequest.png)

![image](https://www.rt-thread.org/document/site/development-guide/github/figures/create_pull_request.png)

在正式发起 Pull Request 之前，需要根据 Preview 里面默认的描述信息即 checklist 仔细核对代码，在没问题的 checklist 对应选项复选框填写[x]确认，**注意[x]两边没有空格**。比如若代码是成熟版本，请选择成熟版本，且可以添加相应的描述信息，checklist 核对完成才可发起 Pull Request。

![image](https://www.rt-thread.org/document/site/development-guide/github/figures/checklistyes.png)

![image](https://www.rt-thread.org/document/site/development-guide/github/figures/checklist.png)

第一次为 RT-Thread 贡献代码需要需要签署 Contributor License Agreement。

![image](https://www.rt-thread.org/document/site/development-guide/github/figures/cla.png)

请确认 CLA 显示签署成功及 CI 编译通过，如下图所示：

![image](https://www.rt-thread.org/document/site/development-guide/github/figures/checkok.png)

提交PR之后，就会获得一个PR#号码，在https://github.com/RT-Thread/rt-thread/pulls可以看到所有的PR请求，其中应该会包含你的。如果是Open状态，说明正在进行代码审查，还没有合并到主分支。

**5.代码审查**

一个完善的BSP，往往不是一次性就能提交成功的。提交PR后，要多看看反馈， 项目管理者会对提交的代码进行审查，如果有问题会在对应的PR下面进行评论，提出修改意见，就像下面这样：

![修改意见](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/RT-Thread/修改意见.jpg)

PR只需要提交一次，每次根据修改意见进行修改之后，项目管理者会看到你的修改，并再次审查修改之后的代码，一般需要2个或以上的项目管理者进行审查，如果代码没有问题，就可以进行以下步骤了。

**6.添加到CI自动化编译**

如果是提交的完整BSP，可以将BSP添加到CI编译脚本，使用远程主机对BSP进行编译，和本地使用arm-gcc scons编译是一样的，如果本地编译正常，这一步基本也会通过。

![修改意见](https://wcc-blog.oss-cn-beijing.aliyuncs.com/Libero/RT-Thread/ci.jpg)

**7.等待合并**

如果CI编译成功，而且审查通过，这个PR会依次被标记为+1、+2，此时只需要耐心等待几天，直到最终被合并到主分支上。

我提交的这个BSP过程可以参考：

https://github.com/RT-Thread/rt-thread/pull/3661

#### 除了代码还能向开源项目贡献什么？

为开源项目做贡献我们可以分为两大类:代码类贡献和非代码类贡献。

**代码贡献**

- BUG修复
- 软件包开发
- BSP提交
- 内核开发

**非代码贡献**

- 撰写和改进文档
- 通过实例来展示RT-Thread的使用
- 为RT-Thread撰写教程，如学习笔记、常见问题等
- 官方社区发布自己的经验文章，或积极回复帖子的问题

#### 注意事项

- 不要使用非GitHub账号提交
- 不要使用不同账号提交Commit之后提交PR，会导致CLA签署失败

- 编程风格遵守documentation 目录下的 coding_style_cn.txt文件。
- 不接受5个以上的Commit

#### 总结

向开源项目贡献代码，提交PR，可以通俗的理解，这里摘自知乎[4]网友的一段解释：

> 我尝试用类比的方法来解释一下pull reqeust。想想我们中学考试，老师改卷的场景吧。你做的试卷就像仓库，你的试卷肯定会有很多错误，就相当于程序里的bug。老师把你的试卷拿过来，相当于先fork。在你的卷子上做一些修改批注，相当于git commit。最后把改好的试卷给你，相当于发pull request，你拿到试卷重新改正错误，相当于merge。
>
> 当你想更正别人仓库里的错误时，要按照下面的流程进行：
>
> 先 fork 别人的仓库，相当于拷贝一份别人的资料。因为不能保证你的修改一定是正确的，对项目有利的，所以你不能直接在别人的仓库里修改，而是要先fork到自己的git仓库中。clone 到自己的本地分支，做一些 bug fix，然后发起 pull request给原仓库，让原仓库的管理者看到你提交的修改。
>
> 原仓库的管理者 review 这个 bug，如果是正确的话，就会 merge 到他自己的项目中。merge 的意思就是合并，将你修改的这部分代码合并到原来的仓库中添加代码或者替换掉原来的代码。至此，整个 Pull Request 的过程就结束了。

#### 参考资料

[1]. 如何开启RT-Thread社区贡献之路
https://mp.weixin.qq.com/s/JfVYB0yUcbyxa5EVWY4DKw

[2]. 五种开源协议(GPL,LGPL,BSD,MIT,Apache)
https://www.oschina.net/question/54100_9455

[3]. 在Github上为RT-Thread贡献代码，为自己的人生涂色
https://mp.weixin.qq.com/s/pPGunFzGcfz01pugNnWCiA

[4]. GitHub 的 Pull Request 是指什么意思？
https://www.zhihu.com/question/21682976