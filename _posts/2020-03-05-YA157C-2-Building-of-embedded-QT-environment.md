---
layout:     post
title:    我用STM32MP1做了个疫情监控平台2—Qt环境搭建
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

- [我用STM32MP1做了个疫情监控平台1—交叉编译环境搭建](https://www.wangchaochao.top/2020/03/04/YA157C-1-Build-cross-compilation-environment/)
- [我用STM32MP1做了个疫情监控平台2—Qt环境搭建](https://www.wangchaochao.top/2020/03/05/YA157C-2-Building-of-embedded-QT-environment/)
- [我用STM32MP1做了个疫情监控平台3—疫情监控平台实现](https://www.wangchaochao.top/2020/03/06/YA157C-3-Novel-coronavirus-pneumonia-surveillance-platform-based-on-embedded-Qt/)
- [我用STM32MP1做了个疫情监控平台4—功能完善界面重新设计](https://www.wangchaochao.top/2020/03/02/YA157C-4-Functional-interface-redesign/)

### 1.嵌入式Qt简介

Qt 是一个跨平台的应用程序开发框架。使用Qt开发的应用程序，只需要编写一套代码，然后把这套代码放在不同平台的Qt环境去编译，就会生成可以运行在对应平台的应用程序。例如，我在Windows写了一个串口助手，这套代码不用修改，放在Linux环境下的Qt开发环境，重新编译，就可以生成可以在Linux环境下运行的串口助手，当然，Qt支持的环境有很多。不同平台下的移植，只需要修改很小一部分或者不用修改就可以直接运行。

![](https://img-blog.csdnimg.cn/20200304173222963.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

嵌入式Qt，即QtE，属于Qt Embedded Linux 分支平台。Qt/E 所面对的硬件平台较多，当开发人员需要在某硬件平台上移植 Qt/E 时，需要下载Qt 源代码，利用交叉编译器编译出 Qt 库。接着需要将 Qt 库复制两份，一份放置在开发主机上，供编译使用；一份放在目标板上，供运行时动态加载使用。

### 2.查看开发板Qt库的版本

要想在开发板上运行Qt程序，首先板子的系统要支持Qt图形库，而且要确定这个的库的版本。那么怎么看开发板是否支持Qt呢？可以使用`find`搜索命令，搜索本地所有Qt相关的文件：

```shell
#进入到根目录
cd /

#搜索qt相关的文件
find -name "*Qt*"
#或者
find -name "*qt*"
```

如果搜索结果有很多so类型的文件，说明这个开发板上的系统是支持Qt的，而且后面的数字就是当前Qt库的版本号。

![](https://img-blog.csdnimg.cn/20200306121100381.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

可以看出，现在的系统是支持Qt的，库的版本是5.11.2。

### 3.主机搭建Qt环境

#### 3.1 安装桌面版本Qt开发套件

安装桌面版本的 Qt 开发套件，用户可以先在  PC 主机上进行 Qt 应用程序的开发和调试，待应用程序完成之后，再使用嵌入式Qt套件构建一下，就可以生成可以在开发板上运行的目标程序。由于开发板上的Qt库版本是5.11版本的，建议桌面Qt版本尽量也是5.11版本的，如果不一致影响也不大。**如果你的电脑上已经安装了Qt 5 Linux版本，这一节可以跳过**。由于我的电脑之前已经安装了5.8版本的，所以不再重新安装。

- **下载 Qt**

Qt 安装包从Qt 5版本开始提供Linux版本的独立安装包，而不需要自己编译。在之前的Qt 4版本，是没有Linux安装包的。

官方下载地址：[Index of /archive/qt](http://download.qt.io/archive/qt/)

最好选择Qt 5.8以上，要选择Linux版本的，如`qt-opensource-linux-x64-5.11.0.run`，这个安装包是桌面Qt程序开发套件，包括qmake、QtCreator等工具。

实测官方下载速度还是非常快的，如果下载速度慢，可以转到国内的镜像地址：[清华大学Qt下载镜像地址](https://mirrors.tuna.tsinghua.edu.cn/qt/archive/qt/)，下载速度会很快。

- **安装 Qt**

下载完成之后，直接双击安装就可以了，如果不能安装尝试添加可执行权限，或者以sudo权限执行：

```shell
#添加可执行权限
sudo chmod +x ./qt-opensource-linux-x64-5.11.0.run

#安装
sudo ./qt-opensource-linux-x64-5.11.0.run

#安装路径可根据需要选择
#其他选择默认安装配置就行了

```

安装完成之后：

![](https://img-blog.csdnimg.cn/20200306123631359.png)

其中MaintenanceTool是Qt的安装管理程序，运行这个文件可卸载Qt。

- **启动 Qt**

安装完成之后，可以在Ubuntu搜索Qt关键字，点击**Qt Creator**启动Qt环境。

![](https://img-blog.csdnimg.cn/20200306124057932.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

你也可以进入到`/Qt5.8.0/Tools/QtCreator/bin`文件夹去启动Qt，如果启动失败，添加`sudo `权限试试。

![](https://img-blog.csdnimg.cn/2020030612421827.png)

#### 3.2 添加嵌入式Qt构建套件

搭建一个最基本的Qt环境，需要两个东西：**qmake和编译器**。在安装桌面版本 Qt 时，已经默认添加了桌面环境的Qt构建套件：

- 桌面版本qmake：`Qt5.8.0/5.8/gcc_64/bin/qmake`
- 桌面版本编译器：`ubuntu 自带的GCC`

![](https://img-blog.csdnimg.cn/20200306124910750.png)

为了编译可以在开发板上运行的Qt程序，我们还需要配置一个开发嵌入式Qt程序的构建套件：

```shell
#嵌入式qmake路径
/opt/st/stm32mp1/2.6-snapshot/sysroots/x86_64-openstlinux_eglfs_sdk-linux/usr/bin/qmake

#交叉编译器路径：
/opt/st/stm32mp1/2.6-snapshot/sysroots/x86_64-openstlinux_eglfs_sdk-linux/usr/bin/arm-openstlinux_eglfs-linux-gnueabi/arm-openstlinux_eglfs-linux-gnueabi-gcc
```

![](https://img-blog.csdnimg.cn/20200306125338339.png)

可以看到嵌入式Qt的版本是5.11.2。知道了qmake和交叉编译器的路径，下面我们在桌面版本Qt中添加一个开发套件，用于构建嵌入式Qt程序。

- 添加交叉编译器

打开QtCreator之后，点击菜单栏的`工具->选项->构建和运行->编译器`，添加交叉编译器：

```shell
#添加gcc交叉编译器
名称：ya157c_gcc
路径：/opt/st/stm32mp1/2.6-snapshot/sysroots/x86_64-openstlinux_eglfs_sdk-linux/usr/bin/arm-openstlinux_eglfs-linux-gnueabi/arm-openstlinux_eglfs-linux-gnueabi-gcc

#添加g++交叉编译器
名称：ya157c_g++
路径：/opt/st/stm32mp1/2.6-snapshot/sysroots/x86_64-openstlinux_eglfs_sdk-linux/usr/bin/arm-openstlinux_eglfs-linux-gnueabi/arm-openstlinux_eglfs-linux-gnueabi-g++

#添加gdb调试器
名称：ya157c_gdb
路径：/opt/st/stm32mp1/2.6-snapshot/sysroots/x86_64-openstlinux_eglfs_sdk-linux/usr/bin/arm-openstlinux_eglfs-linux-gnueabi/arm-openstlinux_eglfs-linux-gnueabi-gdb
```

![](https://img-blog.csdnimg.cn/20200306131033647.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

- 添加嵌入式版本qmake

```shell
#嵌入式Qt版本的qmake路径
路径：/opt/st/stm32mp1/2.6-snapshot/sysroots/x86_64-openstlinux_eglfs_sdk-linux/usr/bin/qmake
```

![](https://img-blog.csdnimg.cn/20200306131149335.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

- **添加设备**

这一步可省略。

```shell
#添加一个通用Linux设备
设备类型：通用Linux
设备名称：ya157c
主机名称：192.168.1.136
用户名：root
密码：root
```

![](https://img-blog.csdnimg.cn/20200306131854657.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

- **添加嵌入式Qt开发套件**

以上都是为了添加开发套件而服务的，开发套件需要指定qmake和编译器等。新建一个构建套件

```shell
名称：ya157c
设备类型：通用Linux设备
设备：选择之前添加的ya157c
Sysroot：/opt/st/stm32mp1/2.6-snapshot/sysroots/cortexa7t2hf-neon-vfpv4-openstlinux_eglfs-linux-gnueabi
C编译器：选择之前添加的ya157c_gcc
C++编译器：选择之前添加的ya157c_g++
调试器：选择之前添加的ya157c_gdb
Qt版本：选择之前添加的Qt 5.11.2
Qt mkspec：linux-oe-g++
```

点击Apply之后，如果构建套件前面有红色或黄色的感叹号，说明构建套件没有设置成功，需要检查配置选项。下面，我们来完成第一个Qt应用程序——Hello World。

### 4.第一个Qt程序——Hello World

嵌入式Qt应用程序的开发，可以完全按照桌面程序的开发流程：新建工程、设计界面和功能、编译运行。最后使用嵌入式开发套件构建一下，就生成了可以在嵌入式平台运行的Qt应用程序。

我们来设计一个简单的界面，程序只显示一个标签，标签的内容是“Hello World”。在PC上运行正确之后，然后使用ya157c开发套件交叉编译，再把可执行文件传输到开发板上运行，整个过程不需要写一行代码。

桌面效果：

开发板效果

#### 4.1 新建一个工程

- 新建一个应用程序工程

![](https://img-blog.csdnimg.cn/20200306134030266.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

- 输入工程名称和保存路径

![](https://img-blog.csdnimg.cn/20200306134234813.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

- 选择构建套件

就是这个程序在哪些平台上运行，我们选择桌面(Desktop Qt 5.8)和开发板(ya157c)这两个套件，如果只选择了一个，在开发过程中也可以再添加其他的构建套件。

![](https://img-blog.csdnimg.cn/20200306134324291.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

- 工程创建

![](https://img-blog.csdnimg.cn/20200306134604327.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

- 界面设计

拖入一个Label，内容是"Hello World"，并调整一下字体和布局。

![](https://img-blog.csdnimg.cn/20200306134754705.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

这样就创建完成了一个最简单的Hello World应用程序。

#### 4.2 PC运行Qt程序

点击左下绿色三角符号，构建并运行，实际效果：

![](https://img-blog.csdnimg.cn/20200306134956472.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

#### 4.3 开发板运行Qt程序

桌面版本运行正常之后，点击左下角电脑标志，切换为ya157c构建套件，再点击底部锤子按钮，交叉编译这个工程。

![](https://img-blog.csdnimg.cn/20200306135510784.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

注意，由于这是交叉编译，所以编译出来的程序不能在本地 PC 机上运行或调试。因此不能点击运行按钮运行程序，也不能点击调试按钮调试程序。

如果构建成功，编译输出的文件默认在当前工程目录的上一级。

![](https://img-blog.csdnimg.cn/20200306140447573.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

可以看到，成功输出了ARM平台下运行的可执行文件。通过scp或其他方式把文件传输到开发板：

```shell
#scp传输可执行文件到开发板
scp hello_world root@192.168.1.136:/home/root
```

![](https://img-blog.csdnimg.cn/20200306140648647.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

连接HDMI显示器或RGB显示屏，我使用的是7寸IPS屏，1024*600分辨率。

开发板运行效果：

![](https://img-blog.csdnimg.cn/20200306141951842.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

### 5.一些问题

- 交叉编译时报错

桌面Qt套件编译时，正常。但是使用交叉编译套件编译会提示错误：

![](https://img-blog.csdnimg.cn/20200306143529385.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

可以通过执行以下命令，复制相应的库文件：

```shell
#切换到库所在的文件夹
cd /opt/st/stm32mp1/2.6-snapshot/sysroots/cortexa7t2hf-neon-vfpv4-openstlinux_eglfs-linux-gnueabi/vendor/lib/

#复制库文件
cp -d * ../../lib/
```

![](https://img-blog.csdnimg.cn/20200306143905767.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

复制完成之后，再编译就不会报错了。

- 液晶屏不能显示程序界面

如果在运行Qt程序时，出现如下提示：

```shell
QStandardPaths: XDG_RUNTIME_DIR not set, defaulting to '/tmp/runtime-root'
qt.qpa.input: X-less xkbcommon not available, not performing key mapping
Could not queue DRM page flip on screen HDMI1 (Device or resource busy)
```

或者可以运行，但字体太小了。可以尝试在运行程序之前，先执行以下命令，再运行Qt程序

```shell
psplash-drm-quit
export QT_QPA_EGLFS_ALWAYS_SET_MODE="1"
export QT_QPA_EGLFS_PHYSICAL_WIDTH=150
export QT_QPA_EGLFS_PHYSICAL_HEIGHT=90
```

其中，150和90是显示屏的物理尺寸，长150mm，宽90mm。

- 编译输出到当前工程文件夹下

Qt工程编译输出的Debug/Release目录是在当前工程目录的上一级：

```shell
../build-%{CurrentProject:Name}-%{CurrentKit:FileSystemName}-%{CurrentBuild:Name}
```

可以改为和工程文件同一目录下：

```shell
./build-%{CurrentProject:Name}-%{CurrentKit:FileSystemName}-%{CurrentBuild:Name}
```

去掉一个`.`就好了。

![](https://img-blog.csdnimg.cn/20200306144312449.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

然后关闭工程，删除工程目录下的.user文件，重新导入，编译。

![](https://img-blog.csdnimg.cn/20200306144453396.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

这样编译目录就在工程目录下了：

![](https://img-blog.csdnimg.cn/2020030614455552.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

### 系列教程

- [我用STM32MP1做了个疫情监控平台1—交叉编译环境搭建](https://www.wangchaochao.top/2020/03/04/YA157C-1-Build-cross-compilation-environment/)
- [我用STM32MP1做了个疫情监控平台2—Qt环境搭建](https://www.wangchaochao.top/2020/03/05/YA157C-2-Building-of-embedded-QT-environment/)
- [我用STM32MP1做了个疫情监控平台3—疫情监控平台实现](https://www.wangchaochao.top/2020/03/06/YA157C-3-Novel-coronavirus-pneumonia-surveillance-platform-based-on-embedded-Qt/)
- [我用STM32MP1做了个疫情监控平台4—功能完善界面重新设计](https://www.wangchaochao.top/2020/03/02/YA157C-4-Functional-interface-redesign/)
