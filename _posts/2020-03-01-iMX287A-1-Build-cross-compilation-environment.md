---
layout:     post
title:    iMX287A交叉编译环境搭建
subtitle:	嵌入式Linux
date:       2020-03-01 12:00:00 +0800
author:     Wang Chao
header-img: img/imx287a.jpg
catalog:    true
tag:
    - ARM
    - Linux
---


### 1.开发套件简介

- 开发板：**EasyARM-i.MX287A**
- 液晶屏：4.3寸TFT液晶，４线电阻触摸，分辨率480 × 272。
- 光盘资料版本：**EasyARM-i.MX28xA_V1.05.iso**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200302130615967.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

### 2.说明：

#### 2.1 开发板配置

- 型号：EasyARM-i.MX287A
- 系统：Linux version 2.6.35.3 gcc 4.4.4
- IP地址：192.168.1.136

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200302132445146.png)

#### 2.2 主机配置

- 系统：Linux主机，如Ubuntu 16.04
- IP地址：192.168.1.111

### 3.主机搭建交叉编译环境

交叉编译器是在PC上运行的编译器，但是编译后得到的二进制程序却不能在PC
上运行，而只能在开发板上运行。交叉编译器命名方式一般遵循“处理器-系统-gcc”这样的
规则，一般通过名称便可以知道交叉编译器的功能。

例如下列交叉编译器:

- arm-none-eabi-gcc,表示目标处理器是 ARM,不运行操作系统,仅运行前后台程序;
- arm-uclinuxeabi-gcc,表示目标处理器是 ARM,运行 uClinux 操作系统;
- arm-none-linux-gnueabi-gcc,表示目标处理器是 ARM,运行 Linux 操作系统;
- mips-linux-gnu-gcc,表示目标处理器是 MIPS,运行 Linux 操作系统。

进行 ARM Linux 开发，通常选择 arm-linux-gcc 交叉编译器。ARM-Linux 交叉编译器可以自行从源代码编译，也可以从第三方获取。在能从第三方获取交叉编译器的情况下，请尽量采用第三方编译器而不要自行编译，一是编译过程繁琐，不能保证成功，二是就算编译成功，也不能保证交叉编译器的稳定性，编译器的不稳定性会对后续的开发带来无限隐患。而第三方提供的交叉编译器通常都经过比较完善的测试，确认是稳定可靠的。

交叉编译器光盘路径：

`EasyARM-i.MX28xA_V1.05/3、Linux/2、工具软件/Linux工具软件/gcc-4.4.4-glibc-2.11.1-multilib-1.0_EasyARM-iMX283.tar.bz2`

主机只需要把这个文件解压到`/opt`目录下，就完成了交叉编译器的安装。

`tar xjvf gcc-4.4.4-glibc-2.11.1-multilib-1.0.tar.bz2 -C /opt/`

> -C：指定解压路径，不指定则默认解压到当前目录

解压完成之后，交叉编译工具链，即gcc/g++/gdb等工具的路径在

`/opt/gcc-4.4.4-glibc-2.11.1-multilib-1.0/arm-fsl-linux-gnueabi/bin`目录下

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200302134140894.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

在安装交叉编译器之后，还需要先安装32 位的兼容库和 libncurses5-dev 库：

```shell
sudo apt-get install ia32-libs
sudo apt-get install libncurses5-dev
```

若 Linux 主机系统没有安装 32 位兼容库,在使用交叉编译工具的时候可能会出现错误：

```shell
arm-fsl-linux-gnueabi-gcc: 没有那个文件或目录
```

### 4.编译第一个ARM Linux程序——Hello World

有了交叉编译工具，和PC平台的gcc使用方法一样，就可以直接编译第一个程序了。

`cd ~`：切换到用户目录
`touch hello.c`：新建一个c文件
`vi hello.c`：输入Ｃ程序，当然也可以使用vim/gedit
```c
#include "stdio.h"

int main(void)
{
    printf("Hello World! -- By arm-gcc \n");
    return 0;
}
```

编写完成之后，先别急着用arm-gcc编译，先用Ubuntu自带的gcc编译一下，看有没有语法错误，能不能正常运行。编译这个C文件，并指定输出文件为pc.o

`gcc hello.c -o pc.o`

看一下这个文件的类型，并执行这个文件。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200302140130382.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

可以看出，没有语法错误，生成了pc.o文件，这个文件是运行在x86_64架构系统上，即PC上的，而且运行结果是我们想要的。

好了，程序运行没问题，就可以使用arm-gcc来编译这个程序，并生成可以在arm开发板上运行的可执行文件了。

使用交叉编译工具编译这个Ｃ程序，指定输出arm.o文件。

`/opt/gcc-4.4.4-glibc-2.11.1-multilib-1.0/arm-fsl-linux-gnueabi/bin/arm-fsl-linux-gnueabi-gcc hello.c -o arm.o`

语法没有错误，生成了arm.o文件，可以通过`file a.out`查看这个文件的信息。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200302140526541.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

因为这个文件是运行在ARM架构的系统上的，所以在PC上不能运行，下面我们通过Ｕ盘把这个文件拷贝到开发板上去运行。

### 5.开发板运行Ｕ盘中的可执行文件

把arm.o文件拷贝到U盘中，并把U盘插到开发板的USB接口。

开发板上电，连接上串口终端，Ubuntu下的串口工具很多，minicom/putty/picocom等等。我使用的是picocom，指定波特率115200和串口号：

`sudo picocom -b 115200 /dev/ttyUSB0`

开发板Linux系统用户名和密码都是`root`

登录进去之后，U盘默认挂载在media目录下。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200302141342525.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

可以看到，程序运行正常。至此，一个简单的hello world程序就完成了。

### 6.配置交叉编译工具到环境变量

在进行交叉编译时，我们是使用的绝对路径来编译C程序，但是这个路径太长了，每次输入很麻烦。
那么能不能像Windows那样，把交叉编译工具所在的路径添加到PATH环境变量呢？当然是可以的。

#### 6.1 环境变量配置的几种方式

**注意：以下涉及到文件修改的地方，需要先执行`mount -o remount rw /`把文件系统挂载为可读写，否则不能保存，因为板子默认所有的文件都是只读的。**

Ubuntu配置环境变量主要以下几种方式：

- export临时设置

以下这两种方式都是可以的，可以在终端直接执行，执行完成之后立即生效，但只在当前终端有效，退出终端自动失效。

```shell
#路径添加在最后面
export PATH=$PATH:/要添加的路径

#路径添加在最前面
export PATH=/要添加的路径:$PATH
```

- 修改用户配置文件bashrc

推荐使用这种方式，修改~/.bashrc文件只对当前用户有效，而不影响其他用户的配置文件。

```shell
#执行如下命令，编辑bashrc文件
sudo vi ~/.bashrc

#在文件最后添加一行
export PATH=$PATH:/要添加的路径

#保存之后soure命令使修改立即生效
source ~/.bashrc
```

- 修改全局配置文件/etc/profile

这种方式修改的是全局环境变量配置文件，针对所有的用户都有效。

```shell
#执行如下命令，编辑/etc/profile文件
sudo vi /etc/profile

#在文件最后添加一行
export PATH=$PATH:/要添加的路径

#保存之后soure命令使修改立即生效
source /etc/profile
```

**注意：修改环境变量配置文件之后，必须执行soure，使修改生效，否则不会立即生效。**

#### 6.2 iMX287A交叉编译工具链添加到环境变量

- 临时设置

`export PATH=$PATH:/opt/gcc-4.4.4-glibc-2.11.1-multilib-1.0/arm-fsl-linux-gnueabi/bin/`

- 对当前用户永久有效

```shell
#执行如下命令，编辑bashrc文件
sudo vi ~/.bashrc

#在文件最后添加一行
export PATH=$PATH:/opt/gcc-4.4.4-glibc-2.11.1-multilib-1.0/arm-fsl-linux-gnueabi/bin/

#保存之后soure命令使修改立即生效
source ~/.bashrc
```

- 对所有用户永久有效

```shell
#执行如下命令，编辑/etc/profile文件
sudo vi /etc/profile

#在文件最后添加一行
export PATH=$PATH:/opt/gcc-4.4.4-glibc-2.11.1-multilib-1.0/arm-fsl-linux-gnueabi/bin/

#保存之后soure命令使修改立即生效
source /etc/profile
```
#### 6.3 查看当前环境变量

修改完成之后，可以通过`echo $PATH`命令查看当前的环境变量路径，以确认是否添加成功。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200302163334504.png)

可以简单理解为$PATH=这些字符串。

当输入`arm-fsl`时，按下TAB键，如果能自动补全，说明环境变量配置成功，否则需要检查是否设置正确。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200302195232557.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

### 7.scp传输文件到开发板

在使用交叉编译工具链，编译出arm.o文件时，我们是通过拷贝到U盘，然后把Ｕ盘插到开发板上来运行程序的，但是这样未免太麻烦了。
那么有没有一种简单的方式，可以在PC Ubuntu主机和开发板快速方便的进行文件传输呢？其实有很多种，nfs，ftp，tftp等等，这里我们使用一种最简单的方式：**scp命令**。

#### 7.1 从本地复制到远程

基本的命令格式主要有以下几种：

```shell
#复制本地文件到远程文件夹，指定了用户名，需要输入密码
scp local_file remote_username@remote_ip:remote_folder 

#复制本地文件到远程文件，指定了用户名，需要输入密码
scp local_file remote_username@remote_ip:remote_file 

#复制本地文件到远程文件夹，未指定用户名，需要输入用户名和密码
scp local_file remote_ip:remote_folder 

#复制本地文件到远程文件，未指定用户名，需要输入用户名和密码
scp local_file remote_ip:remote_file 
```

应用示例：

```shell
scp /home/space/music/1.mp3 root@192.168.1.136:/home/root/others/music 
scp /home/space/music/1.mp3 root@192.168.1.136:/home/root/others/music/001.mp3 
scp /home/space/music/1.mp3 192.168.1.136:/home/root/others/music 
scp /home/space/music/1.mp3 192.168.1.136:/home/root/others/music/001.mp3 
```

复制整个目录及其子文件

```shell
#指定了用户名，需要输入密码
scp -r local_folder remote_username@remote_ip:remote_folder 

#未指定用户名，需要输入用户名和密码
scp -r local_folder remote_ip:remote_folder 
```

应用示例：

```shell
#复制当前music目录下所有文件到远程others目录，指定了用户名，需要输入密码
scp -r /home/space/music/ root@192.168.1.136:/home/root/others/ 

#未指定用户名，需要输入用户名和密码
scp -r /home/space/music/ 192.168.1.136:/home/root/others/ 
```

#### 7.2 从远程复制到本地

从远程复制到本地，只要将从本地复制到远程的命令的后2个参数调换顺序即可，如下实例

应用实例：

```shell
#复制文件
scp root@192.168.1.136:/home/root/others/music /home/space/music/1.mp3 

#复制目录
scp -r 192.168.1.136:/home/root/others/ /home/space/music/
```

说明

- 1.如果远程服务器防火墙有为scp命令设置了指定的端口，我们需要使用 -P 参数来设置命令的端口号。命令格式如下：
```shell
#scp 命令使用端口号 4588
scp -P 4588 remote@192.168.1.136:/usr/local/sin.sh /home/administrator
```
- 2.使用scp命令要确保使用的用户具有可读取远程服务器相应文件的权限，否则scp命令是无法起作用的。

#### 7.3 PC传输文件到ARM开发板

scp命令是基于物理网口的，在进行传输之前，需要确定开发板和PC主机是可以正常通信的。

- 开发板配置eth0网口IP地址：
`ifconfig eth0 192.168.1.136 up`

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200302131904310.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

- 主机配置IP地址
通过**有线连接**选项，手动配置IPv4地址

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020030213211031.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

开发板和主机互相ping，测试网络是否正常。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200302174851111.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)这样就说明是正常的。

把PC主机上的arm.o文件传输到开发板上：

`scp ~/arm.o root@192.168.1.136:/root`

如果出现如下错误：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200302175229587.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

只需要执行一下提示的那一行命令就行了：

`ssh-keygen -f "/home/whik/.ssh/known_hosts" -R 192.168.1.136`

如果还是报错：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200302175417769.png)那就把knows_host文件删除了

`rm ~/.ssh/know_hosts`

再执行scp命令：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200302175803888.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)先输入yes，然后输入开发板的登录密码，下面会显示传输的进度。

到开发板上看一下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200302175926187.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

### 8. SSH登录开发板

如果能传输成功，说明网络是通畅的。那么还可以使用SSH方式登录开发板，和使用串口终端的效果是完全一样的。

`ssh user_name@remot_ip`

如：`ssh root@192.168.1.136`

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200302180410653.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)这样，使用一根网线就完成了终端和文件传输的功能，就不用USB-TTL模块了。

### 9. NFS传输文件

还有一种比较常用的传输方式，那就是在主机搭建启动NFS服务器，把文件夹设置共享目录，然后开发板把服务器(PC)上的这个文件夹挂载到本地，那PC和开发板都可以直接访问这个文件夹。

#### 9.1 Ubuntu主机安装NFS服务器

```shell
#安装NFS服务器 
sudo apt-get install nfs-kernel-server

#安装NFS客户端
sudo apt-get install nfs-common
```

#### 9.2 Ubuntu主机设置共享目录

创建或设置要共享的目录

我的共享目录：`/home/whik/imx287/share`

并设置最宽松的权限:

```shell
#加上所有的权限
sudo chmod -R 777 /home/whik/imx287/share

#任何用户
sudo chown –R nobody /home/whik/imx287/share
```

指定NFS共享目录：

```shell
#打开exports文件
sudo vi /etc/exports

#在文件最后添加一行
/home/whik/imx287/share *(rw,sync,no_root_squash)
#表示任意IP网段都可以访问，权限最宽松
```

主机启动NFS服务器：

```shell
#启动
sudo /etc/init.d/nfs-kernel-server start

#重启
sudo /etc/init.d/nfs-kernel-server restart

#停止
sudo /etc/init.d/nfs-kernel-server stop
```

在 NFS 服务已经启动的情况下,如果修改了“/etc/exports”文件,需要重启 NFS 服务，以刷新 NFS 的共享目录。当然在下一次启动系统时,NFS 服务是自动启动的。

查看当前所有的共享目录：`showmount -e`

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200302181853649.png)

#### 9.3 测试NFS服务器启动成功

测试是否启动成功，就是先把PC主机共享目录挂载到PC本地，PC主机地址:192.168.1.111

```shell
#把共享目录挂载到/mnt目录
sudo mount -t nfs 192.168.1.111:/home/whik/imx287/share /mnt -o nolock
```

如果/mnt目录和共享目录下的内容完全一致，说明挂载成功。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200302182257102.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)这样，就说明PC主机上的NFS服务器成功启动，并且共享目录是可以正常访问的。

那么，ARM开发板上，如何挂载呢？在进行挂载之前，先确认主机和开发板网络是通畅的，即互相能ping通。

串口或ssh登录开发板之后，执行以下命令：

```shell
#开发板挂载主机共享目录到mnt
mount -t nfs 192.168.1.111:/home/whik/imx287/share /mnt -o nolock
#或者
mount -t nfs -o nolock,vers=2 192.168.1.111:/home/whik/imx287/share /mnt
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200302183042799.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

挂载成功之后，在开发套件的/mnt 目录下也可以看见主机共享的目录。然后开发套件就可以像操作本地目录一样去操作主机的共享目录了。

如果不想挂载了，可以使用`umount /mnt`来取消挂载。有一点要特别注意，无论是执行mount还是umount，执行命令的当前路径都不能是操作的目标路径。即不能在/mnt目录去执行mount和umount命令。

主机和开发板传输文件的方式还有很多种，我个人经常用scp命令，不需要设置那么多。其他的方式如ftp/tftp/sftp等都是一样的原理，这里不再介绍。

### 10.开机启动脚本配置

开发板ip配置，在开发板掉电重启，不会默认配置，所以每次启动之后，都需要重新`ifconfig eth0 192.168.1.136`来配置一下IP地址。这样还是太麻烦了，那么能不能开机自动执行这些配置呢？这就涉及到开发板初始化脚本文件了。

```shell
#重新挂载所有文件为可读写权限
mount -o remount rw /

#编辑启动脚本文件
vi /etc/init.d/rcS 

#文件末尾添加以下内容
#启动界面
/usr/share/zhiyuan/zylauncher/start_zylauncher &

#配置eth0的IP地址
ifconfig eth0 192.168.1.136 &

#自动获取IP，需要连接路由器联网时使用
#udhcpc &

#挂载linux主机共享目录到mnt
mount -t nfs -o nolock,vers=2 192.168.1.111:/home/whik/imx287/share /mnt &

#重新挂载文件为可读写权限
mount -o remount rw / &
```

以上命令，可以根据需要来配置。

注意，如果程序是一个阻塞程序(运行后不会退出或返回的程序)，则可能会导致位于其后的指令或程序无法执行。再者，若该程序始终占用串口终端,将会造成其他程序，无法通过串口终端与用户交互。对于此类应用程序，可以在其后面添加` &`

几个文件的位置：

- 开发板字库文件路径：`/usr/lib/fonts`
- 桌面系统启动界面：`/usr/share/zhiyuan/zylauncher`
- Qt 4库文件路径：`/usr/lib`
- 查找字库文件：`find / -name "*.ttf"`

> 我的公众号：**mcu149**
