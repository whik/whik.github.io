---
layout:     post
title:      小米2s(MIUI7)安装Xposed框架
subtitle:   折腾手机的神器
date:       2016-02-04
author:     Shao Guoji
header-img: img/post-bg-xposed.jpg
catalog:    true
tag:
    - 手机
---

> Xposed框架是一款可以在不修改APK的情况下影响程序运行（修改系统）的框架服务，基于它可以制作出许多功能强大的模块，且在功能不冲突的情况下同时运作。（需root权限）

# 又是一篇教程？？？

* 写在前面的废话
* 准备工作
* 第三方recovery的刷入
* Xposed框架的安装与使用
* 心得体会

---
<br/>

### 一、写在前面的废话

之前家城来探我时show了一下他手机的红包插件（基于Xposed框架），当时觉得挺有意思，自己也想搞一个。凭借着印象中他不太标准的发音，我还是百度到了“Xposed框架”，开始了苦逼的摸索之旅……

*PS：人家魅族的系统在应用商店下个[安装器](http://www.coolapk.com/apk/com.wuxianlin.xposedinstaller)就搞定，我的MIUI 7却搞了几天，才发现MIUI也有MIUI的不足~~~*

---
<br/>

### 二、准备工作

* 第三方recovery（TWRP Recovery）
* Xposed安装器
* Xposed安装包
* Xposed应用模块

所用到的软件下载地址：[小米2s安装xposed工具包](http://pan.baidu.com/s/1sk68nlr)

我手机系统版本**安卓5.0.2**，MIUI 7.0.4.0稳定版

![手机系统版本](http://odaps2f9v.bkt.clouddn.com/16-10-2/26480955.jpg)

#### 刷机有风险，请先备份手机重要数据（建议先用小米本地备份，再把MIUI\backup文件夹复制到电脑比较保险，配合小米账号云端的桌面布局及APP备份可完美还原）

---
<br/>

### 三、第三方recovery的刷入

> Recovery模式指的是一种可以对安卓机内部的数据或系统进行修改的模式（类似于windows PE或DOS）。在这个模式下我们可以刷入新的安卓系统，或者对已有的系统进行备份或升级，也可以在此恢复出厂设置。          

为什么需要第三方recovery？因为自带的不够强大，而一些**底层应用**（如SuperSu和Xposed）的安装不能使用常规的APP安装方法，而是要从recovery直接刷入手机，自带的recovery不能胜任，所以刷入第三方recovery是基础。

经过各种尝试，刷了好多版本的recovery，都不能用，要么就是读不了内存卡文件，要么就是进入时无背光（用手电照还是有字的），直到今天早上才找到一个MIUI7能用的TWRP Recovery 2.8.7.0，才完美解决。

此外还需要一个很好用的“Mi2/2S_recovery一键刷入工具”来刷recovery。

下载好工具包的TWRP Recovery，文件夹中的**recovery.img**便是我们要刷入的recovery，将它复制到Mi2/2S_recovery一键刷入工具”目录中替换原来自带的recovery.img。关机，按**电源+音量下**键进入手机的fastboot模式，把手机用数据线连接电脑，在电脑打开recovery.bat批处理，按提示操作即可，刷入成功后会有提示。

![刷入工具](http://odaps2f9v.bkt.clouddn.com/16-10-2/81368165.jpg)

![进入fastboot](http://odaps2f9v.bkt.clouddn.com/16-10-2/85720759.jpg)

![刷入成功](http://odaps2f9v.bkt.clouddn.com/16-10-2/45948015.jpg)

刷好后手机会自动重启，重启后再进入recovery就能看到美美的界面了~~~

![recovery](http://odaps2f9v.bkt.clouddn.com/16-10-2/14281528.jpg)

**PS：一定要等到重启进入桌面后再关机进新的recovery，否则有可能导致recovery刷入失败**

---
<br/>

### 四、Xposed框架的安装与使用

#### 1、安装Xposed Installer（Xposed安装器）

在手机安装**de.robv.android.xposed.installer_3.0_alpha4_37.apk**

*Xposed框架是直接从recovery刷入，安装器只是用来查看是否安装成功和管理模块。*

#### 2、安装Xposed框架zip包

复制**xposed-v78-sdk21-arm-MIUI-edition-by-SolarWarez-20151124.zip**到内存卡根目录（**不要解压**），重启进入TWRP Recovery，在“安装”选项找到刚刚的zip文件，滑动刷入，便完成了Xposed框架的安装。

如果手机没有root，会提示刷入SuperSu软件（实际上许多教程都推荐使用SuperSu来管理root权限）

![SuperSu](http://odaps2f9v.bkt.clouddn.com/16-10-3/83851245.jpg)

重启手机，打开Xposed Installer，如果显示“**Xposed framework version 78 (MIUI edition by SolarWarez / 20151124) is active.**”的话，恭喜你成功安装了强大的Xposed框架！

![成功安装](http://odaps2f9v.bkt.clouddn.com/16-10-3/92889572.jpg)

#### 3、安装Xposed应用模块

工具包中有一个**com.wuxianlin.luckymoney_1.3.3_9.apk**红包插件，直接安装，安装后打开Xposed Installer，菜单中的Modules会显示已安装的所有基于Xposed框架的应用，**勾选红包插件，重启手机**，即可使用。

至于更多的Xposed应用，可看这篇帖子：[Xposed框架下37款神级应用](http://www.52pojie.cn/thread-446465-1-1.html)

---
<br/>

### 五、心得体会

从为了装SuperSu苦练刷recovery，到满大街找可用的Xposed框架zip包，手机一次次无法开机不得不线刷恢复，各种艰辛（都是我自找的，其实只要一开始多逛论坛就好），不过**折腾的越多，弯路走的越多，学到的也就越多**，而且对于我那台破小米，我也没什么好顾虑的，最多就弄坏手机没得用，又不是没体验过没有智能机的生活。

翻了下日记，家城是1月31号晚来找的，今天是2月4号，四天，虽说断断续续，但最终还是把问题解决了，虽说不是什么大项目，但起码**自己把一件事情做好了**。昨晚十点多上床十一点多睡觉，今早九点多起，回家以来第一次尝试早睡早起（平时睡到中午十二点多才吃早餐），上午把框架弄好（早起福利啊），吃个午饭，下午和老同学看看聚会场地聊聊天，晚上写写博客，多好的生活啊，**比之前的各种糜烂，正常作息的一天竟是如此珍贵**，回想起假期以来做什么事都是半途而废，这样算来，今天无疑是最美好的一天……

<br/>
<br/>

> 参考文章
> 
> * 救命贴！：[小米2/2s基于安卓5.02版本的MIUI 7 刷入xposed！！！](http://www.miui.com/thread-3151737-1-1.html)
> * [不会刷入Recovery不用愁 三种方法教会你](http://android.tgbus.com/Android/yizhi/201412/511888.shtml)
> * [★写给小白看★教你如何使用ADB(刷ROOT刷recovery刷系统...](http://www.oneplusbbs.com/forum.php?mod=viewthread&tid=315970)
> * [XDA论坛Xposed官方下载](http://forum.xda-developers.com/showthread.php?t=3034811)







