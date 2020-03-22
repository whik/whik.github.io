---
layout:     post
title:    我用STM32MP1做了个疫情监控平台1—交叉编译环境搭建
subtitle:	嵌入式Linux
date:       2020-03-05 12:00:00 +0800
author:     Wang Chao
header-img: img/ya157c.jpg
catalog:    true
tag:
    - ARM
    - Linux
---

### 0.系列教程

- [我用STM32MP1做了个疫情监控平台1—交叉编译环境搭建](https://www.wangchaochao.top/2020/03/05/YA157C-1-Build-cross-compilation-environment/)
- [我用STM32MP1做了个疫情监控平台2—Qt环境搭建](https://www.wangchaochao.top/2020/03/05/YA157C-2-Building-of-embedded-QT-environment/)
- [我用STM32MP1做了个疫情监控平台3—疫情监控平台实现](https://www.wangchaochao.top/2020/03/06/YA157C-3-Novel-coronavirus-pneumonia-surveillance-platform-based-on-embedded-Qt/)
- [我用STM32MP1做了个疫情监控平台4—功能完善界面重新设计](https://www.wangchaochao.top/2020/03/22/YA157C-4-Functional-interface-redesign/)

### 1.开发板简介

- 开发板型号：MYD-YA157C，512MB DDR3，4GB eMMC
- 主控芯片：STM32MP157AAC
- 光盘资料版本：MYD-YA157C-20191225.iso 

![](https://img-blog.csdnimg.cn/20200305214128578.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

MYD-YA157C开发套件由核心板MYC-YA157C和底板MYB-YA157C组成，主控芯片是ST目前最高配置的MPU——STM32MP157AAC3，双核Corte-A7+Cortex-M4，主频最高可达650Mhz。

硬件准备

- 12v电源适配器
- USB-TTL模块：115200/8/1/无
- 网线

 ![](https://img-blog.csdnimg.cn/20200305214559782.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

开发板和主机配置

- 开发板Linux版本：Linux 4.19.9
- 开发板IP：192.168.1.136
- 主机配置：Ubuntu 16.04
- 主机IP：192.168.1.111

![](https://img-blog.csdnimg.cn/20200305214955466.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

### 2.主机搭建交叉编译环境

所谓交叉编译，其实是相对于本地编译，即在一种平台上编译出来的程序，可以在另外一个平台下运行，即编译的环境和运行的环境不一样，属于交叉的。在进行嵌入式开发时，常常是在PC(x86架构)上使用交叉编译工具编译，编译出来的可执行文件在开发板(ARM)平台下运行。

交叉编译工具包，位于光盘资料的`03-Tools/Complie Toolchain`目录下，是一个压缩包，直接右键提取，或者使用tar解压命令都可以把压缩包解压。解压完成之后有以下几个文件：

```shell
#解压sdk
tar xvf qt-sdk.tar.xz

#进入sdk目录之后可以看到以下文件
meta-toolchain-qt5-openstlinux-eglfs-stm32mp1-x86_64-toolchain-2.6-snapshot.host.manifest
meta-toolchain-qt5-openstlinux-eglfs-stm32mp1-x86_64-toolchain-2.6-snapshot.sh
meta-toolchain-qt5-openstlinux-eglfs-stm32mp1-x86_64-toolchain-2.6-snapshot.target.manifest
meta-toolchain-qt5-openstlinux-eglfs-stm32mp1-x86_64-toolchain-2.6-snapshot.testdata.json
```

因为后面我们会进行Qt应用的开发，所以这里我们选择带Qt图形库支持的交叉编译工具包

安装交叉编译工具包：

```shell
#切换到解压之后的文件夹执行安装脚本
./meta-toolchain-qt5-openstlinux-eglfs-stm32mp1-x86_64-toolchain-2.6-snapshot.sh

#按[ENTER]键选择默认的安装配置，默认安装在/opt目录下
```

![](https://img-blog.csdnimg.cn/20200305181833931.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

来看一下是否安装成功了：

```shell
#切换到安装目录
cd /opt/st/stm32mp1/2.6-snapshot/

#临时设置环境变量
source ./environment-setup-cortexa7t2hf-neon-vfpv4-openstlinux_eglfs-linux-gnueabi
#这样会把GCC交叉编译器临时添加到环境变量，退出终端失效

#查看GCC交叉编译器版本
arm-openstlinux_eglfs-linux-gnueabi-gcc --version
#或者使用$CC --version

#输出信息
arm-openstlinux_eglfs-linux-gnueabi-gcc (GCC) 8.2.0
Copyright (C) 2018 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

![](https://img-blog.csdnimg.cn/20200305183621927.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

如果能输出版本信息，就说明安装成功了。

### 3.编译第一个ARM Linux程序——Hello World

有了交叉编译工具，和PC平台的gcc使用方法一样，就可以直接编译第一个程序了。

```shell
#切换到用户目录
cd ~

#新建一个目录
mkdir hello

#切换到hello目录
cd hello

#新建一个C文件
touch hello.c

#输入Hello World程序
vim hello.c
```

hello.c文件的内容：

```c
#include <stdio.h>

int main(void)
{

    printf("Hello STM32MP1 ! -- By arm-gcc\n");
    return 0;
}

```

编写完成之后，先别急着用arm-gcc编译，先用Ubuntu自带的gcc编译一下，看有没有语法错误，能不能正常运行。编译这个C文件，并指定输出文件为pc.o

```shell
gcc hello.c -o pc.o
```

看一下这个文件的类型，并执行这个文件。

![](https://img-blog.csdnimg.cn/20200305185055927.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

可以看出，没有语法错误，生成了pc.o文件，这个文件是运行在x86_64架构系统上，即PC上的，而且运行结果是我们想要的。好了，程序运行没问题，就可以使用arm-gcc来编译这个程序，并生成可以在arm开发板上运行的可执行文件了。再使用交叉编译工具编译这个Ｃ程序，指定输出arm.o文件。

```shell
$CC hello.c -o arm.o
```

语法没有错误，生成了arm.o文件，可以通过file命令查看这个文件的信息。

![](https://img-blog.csdnimg.cn/20200305185343574.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

因为这个文件是运行在ARM架构的系统上的，所以在PC上不能运行，下面我们把这个文件放到开发板上去运行。

### 4.在开发板上运行Hello World程序

怎么能在开发板上运行这个程序呢？也就是怎么能把这个文件传输到开发板上呢？

#### 4.1 U盘拷贝

这恐怕是最简单的方法了。把生成的arm.o文件复制到U盘里，把U盘插到板子上的USB接口，并挂载到mnt目录

```shell
#查看当前的设备
ls /dev/sda*

#挂载U盘到mnt目录
mount /dev/sda /mnt

#如果没有挂载成功，尝试挂载另外一个设备
mount /dev/sda4 /mnt

#挂载成功之后切换到mnt目录
cd /mnt

#运行arm.o
./arm.o
```

实际运行：

![](https://img-blog.csdnimg.cn/20200305201058392.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

同样取消挂载：

```shell
#先退出/mnt目录
cd ~

#取消挂载
umount /mnt
```

![](https://img-blog.csdnimg.cn/20200305201322169.png)

这种方式有点麻烦，我们来使用另外一种方法。

#### 4.2 scp文件传输

在使用交叉编译工具链，编译出arm.o文件时，我们是通过拷贝到U盘，然后把Ｕ盘插到开发板上来运行程序的，但是这样未免太麻烦了。

那么有没有一种简单的方式，可以在PC Ubuntu主机和开发板快速方便的进行文件传输呢？其实有很多种方法，nfs，ftp，tftp等等，这里我们使用一种最简单的方式：scp命令。

scp命令是基于物理网口的，在进行传输之前，需要确定开发板和PC主机是可以正常通信的。开发板和电脑使用网线连接，或者开发板连接路由器，电脑连路由器的WiFi，这两种方式都是可以的。

- 开发板配置eth0网口IP地址：

```shell
ifconfig eth0 192.168.1.136 up
```

![](https://img-blog.csdnimg.cn/20200305201733509.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

- 主机配置IP地址

通过**有线连接**选项，手动配置IPv4地址

![](https://img-blog.csdnimg.cn/2020030213211031.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

开发板和主机互相ping，测试网络是否正常。

![](https://img-blog.csdnimg.cn/20200305202447325.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

这样就说明是正常的。把PC主机上的arm.o文件传输到开发板上：

```shell
scp ~/arm.o root@192.168.1.136:/root
```

如果出现如下错误：

![](https://img-blog.csdnimg.cn/2020030520295183.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

只需要执行一下提示的那一行命令就行了：

```shell
ssh-keygen -f "/home/whik/.ssh/known_hosts" -R 192.168.1.136
```

如果还是报错：

![](https://img-blog.csdnimg.cn/20200302175417769.png)那就把knows_host文件删除了

```shell
rm ~/.ssh/know_hosts
```

再执行scp命令：

![](https://img-blog.csdnimg.cn/20200305203033212.png)

先输入yes，下面会显示传输的进度。

到开发板上看一下：

![](https://img-blog.csdnimg.cn/20200305203106619.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

可以看到，和PC上的运行结果是一样的。

关于scp的其他用法：

```shell
#复制本地文件到远程文件夹
scp local_file remote_username@remote_ip:remote_folder 

#复制本地文件到远程文件
scp local_file remote_username@remote_ip:remote_file 

#复制整个目录及其子文件
scp -r local_folder remote_username@remote_ip:remote_folder 
```

从远程复制到本地，只要将从本地复制到远程的命令的后2个参数调换顺序即可。

- 1.如果远程服务器防火墙有为scp命令设置了指定的端口，我们需要使用 -P 参数来设置命令的端口号。命令格式如下：

```shell
#scp 命令使用端口号 4588
scp -P 4588 remote@192.168.1.136:/usr/local/sin.sh /home/administrator
```

- 2.使用scp命令要确保使用的用户具有可读取远程服务器相应文件的权限，否则scp命令是无法起作用的。

#### 4.3 tftp文件传输

另一种文件传输方式，比scp麻烦一些，先在主机配置tftp服务器，并配置共享目录，然后就可以开始文件传输了。

```shell
#主机安装tftp服务器
sudo apt-get install tftpd-hpa

#创建共享目录
mkdir ftp

#修改目录权限
chmod 777 ftp

#在配置文件中添加共享目录
sudo vim /etc/default/tftp-hpa

#添加共享文件夹
TFTP_DIRECTORY="/home/whik/ya157c/ftp"                                         

#启动tftp服务器
sudo service tftpd-hpa restart
```

![](https://img-blog.csdnimg.cn/2020030520462013.png)

开发板获取主机192.168.1.111上共享目录下的a.cpp文件，并重新命名为b.cpp保存到本地

```shell
#把远程的a.cpp文件保存到本地b.cpp
tftp 192.168.1.111 -g -r a.cpp -l b.cpp

#把远程的a.cpp保存到本地，不重命名
tftp 192.168.1.111 -g -r a.cpp
```

![](https://img-blog.csdnimg.cn/20200305205402263.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

参数说明：

```shell
-g 表示下载文件(get)
-p 表示上传文件(put)
-l 表示本地文件名(local file)
-r 表示远程主机的文件名(remote file)
```

### 5.ssh登录开发板

如果scp和tftp都可以正常传输，我们还可以使用ssh命令登录开发板，和串口登录是一样的。

```shell
ssh root@192.168.1.136
```

![](https://img-blog.csdnimg.cn/20200305213445532.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

### 6.注意

如果遇到无法正常安装程序，请尝试安装以下程序。

```shell
#更新源
sudo apt-get update
sudo apt-get upgrade

#安装所需要的软件
sudo apt-get install libusb-1.0-0

sudo apt-get install bison flex sed wget curl cvs subversion git-core
coreutils unzip texi2html texinfo docbook-utils gawk python-pysqlite2 diffstat
help2man make gcc build-essential g++ desktop-file-utils chrpath libxml2-utils
xmlto docbook bsdmainutils iputils-ping cpio python-wand python-pycryptopp
python-crypto

sudo apt-get install libsdl1.2-dev xterm corkscrew nfs-common nfs-kernel-
server device-tree-compiler mercurial u-boot-tools libarchive-zip-perl

sudo apt-get install ncurses-dev bc linux-headers-generic gcc-multilib
libncurses5-dev libncursesw5-dev lrzsz dos2unix lib32ncurses5 repo libssl-dev
```

### 7.shell脚本点灯

简单写一个shell脚本闪个灯，没什么技术含量，led_blink.sh文件内容：

```shell
#!/bin/bash

echo none > /sys/class/leds/heartbeat/trigger

#死循环
while true
do
    echo 0 > /sys/class/leds/heartbeat/brightness
    echo "点亮"
    sleep 0.1
    echo 1 > /sys/class/leds/heartbeat/brightness
    sleep 0.1
    echo "熄灭"
done
```

在开发板上运行

```shell
#scp传输
scp led_blink.sh root@192.168.1.136:/home/root

#开发板给这个脚本添加可执行权限
chmod +x led_blink.sh

#开发板执行这个脚本
./led_blink.sh
```

![](https://img-blog.csdnimg.cn/20200306120648741.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

### 系列教程

- [我用STM32MP1做了个疫情监控平台1—交叉编译环境搭建](https://www.wangchaochao.top/2020/03/05/YA157C-1-Build-cross-compilation-environment/)
- [我用STM32MP1做了个疫情监控平台2—Qt环境搭建](https://www.wangchaochao.top/2020/03/05/YA157C-2-Building-of-embedded-QT-environment/)
- [我用STM32MP1做了个疫情监控平台3—疫情监控平台实现](https://www.wangchaochao.top/2020/03/06/YA157C-3-Novel-coronavirus-pneumonia-surveillance-platform-based-on-embedded-Qt/)
- [我用STM32MP1做了个疫情监控平台4—功能完善界面重新设计](https://www.wangchaochao.top/2020/03/22/YA157C-4-Functional-interface-redesign/)
