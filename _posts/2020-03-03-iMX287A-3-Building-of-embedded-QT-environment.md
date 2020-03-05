---
layout:     post
title:    iMX287A嵌入式Qt环境搭建
subtitle:	嵌入式Linux
date:       2020-03-03 12:00:00 +0800
author:     Wang Chao
header-img: img/imx287a.jpg
catalog:    true
tag:
    - ARM
    - Linux
    - Qt
---

### 1.嵌入式Qt简介

Qt 是一个跨平台的应用程序开发框架。使用Qt开发的应用程序，只需要编写一套代码，然后把这套代码放在不同平台的Qt环境去编译，就会生成可以运行在对应平台的应用程序。例如，我在Windows写了一个串口助手，这套代码不用修改，放在Linux环境下的Qt开发环境，重新编译，就可以生成可以在Linux环境下运行的串口助手，当然，Qt支持的环境有很多。不同平台下的移植，只需要修改很小一部分或者不用修改就可以直接运行。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200304173222963.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

嵌入式Qt，即QtE，属于Qt Embedded Linux 分支平台。Qt/E 所面对的硬件平台较多，当开发人员需要在某硬件平台上移植 Qt/E 时，需要下载Qt 源代码，利用交叉编译器编译出 Qt 库。接着需要将 Qt 库复制两份，一份放置在开发主机上，供编译使用；一份放在目标板上，供运行时动态加载使用。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200304172256460.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

### 2.查看开发板Qt库的版本

要想在开发板上运行Qt程序，首先板子上要有Qt的库，而且要确定这个的库的版本。那么怎么看开发板上的Qt库是Qt-4.7.3版本的。可以使用`find`搜索命令，搜索本地所有Qt相关的文件：

```shell
#进入到根目录
cd /

#搜索qt相关的文件
find -name "*Qt*"
#或者
find -name "*qt*"
```

如果搜索结果有很多so类型的文件，说明这个开发板上的系统是支持Qt的，而且后面的数字就是当前Qt库的版本号。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200304181835297.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

可以看出，iMX287A开发板支持Qt，库的版本是4.7.3。

### 3.第一个嵌入式Qt程序——Hello World

又是"Hello World"，无论学习什么东西，都要先来个"Hello World"，当然Qt也不例外。

#### 3.1 主机搭建嵌入式Qt环境

搭建一个最基本的Qt环境，需要两个东西：**qmake和编译器**。编译器用的是交叉编译器，我们在第一节的教程中，已经介绍了，并且已经把交叉编译器的路径添加到了环境变量。下面我们就来安装用来开发嵌入式程序的qmake。

qmake包工具在光盘的位置：`3、Linux\2、工具软件\Linux 工具软件\qt4.7.3.tar.bz2`

```shell
#进入到opt目录
cd /opt

#解压qmake套件，Qt-4.7.3.tar.bz2
sudo tar -jxvf qt4.7.3.tar.bz2

#添加到用户环境变量
sudo vim ~/.bashrc

#文件末尾添加一行，$PATH放在后面，表示路径添加在环境变量最前面
export PATH=/opt/qt4.7.3/bin/:$PATH

#使设置的环境变量生效
source ~/.bashrc

#查看当前的PATH路径
echo $PATH

#查看当前Qt版本
qmake -v
```

**如果本机有多个qmake，那么一定要把嵌入式qmake路径添加到环境变量最前面，否则不能识别**

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020030418552414.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

如果输出了Qt的版本，并且所在路径是我们设置的，说明Qt环境搭建成功。

#### 3.2 编写Hello World源程序

我们来编写一个简单的界面，程序只显示一个标签，标签的内容是“Hello World”。在PC上交叉编译之后，把可执行文件传输到开发板上运行。

```shell
#新建一个文件夹存放qt工程
mkdir hello_qt

#新建cpp文件
touch hello_qt.cpp

#编辑hello_qt文件
vim hello_qt.cpp

```

hello_qt.cpp文件的内容：

```cpp
//Qt图形库
#include <QtGui>

int main(int argc, char *argv[])
{
    QApplication app(argc,argv);
	//新建一个标签
    QLabel label(QString("hello qt"));
    label.show();

    app.exec();
}
```

程序很简单，就是新建了个标签，文本内容是"hello qt"，然后让这个标签show出来。下面开始编译，生成可执行文件：

```shell
#生成.pro文件
qmake -project

#生成Makefile文件
qmake

#编译生成可执行文件
make
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200304185659487.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

这样，就生成了hello_qt的可执行文件，可以使用file命令看一下文件类型：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200304185739374.png)

支持ARM平台运行的Qt程序。

#### 3.3 开发板运行Hello World

通过scp传输，NFS共享的方式把这个文件在开发板上运行：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200304190101171.png)

在开发板上运行：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020030419013478.png)

实际效果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020030419044434.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

### 4.Linux桌面版本Qt环境的搭建

Qt 是一个跨平台的图形框架,在安装了桌面版本的 Qt SDK 的情况下，用户可以先在PC 主机上进行 Qt 应用程序的开发调试，待应用程序基本成型后，再将其移植到目标板上。

桌面版本的 Qt SDK 主要包括以下两个部分：

- 用于桌面版本的Qt
- Qt Creator

#### 4.1 安装桌面版本的Qt4

由于iMX287A官方系统内的Qt库是Qt-4.7.3版本的库，所以我们也要在桌面Linux安装Qt4版本。官方的下载链接里，只提供了Linux版本的Qt5，而如果想安装Linux版本的Qt4，需要自己使用源码进行编译。这里提供一个简单的方法，那就是Ubuntu自带的命令行`apt-get`安装功能，使用命令安装Qt4版本。在使用前，请确保已经更换为中国的服务器，否则下载速度会很慢。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200304175826207.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

```shell
#更新软件列表
sudo apt-get update

#安装Qt4相关的所有软件
sudo apt-get install qt4*

#安装QtCreator
sudo apt-get install qtcreator
```

耐心等待一会就安装好了，如果安装过程中提示缺少某个库，那就先apt-get安装某个库就可以了。

#### 4.2 配置Qt Creator的构建套件

打开`工具->选项->构建和运行`菜单，添加嵌入式Qt的构建套件，默认桌面环境下的Qt4构建套件已经安装好了。我们只需要设置一下嵌入式环境下的Qt4构建套件

```shell 
qmake路径：/opt/qt4.7.3/bin/qmake

#交叉编译gcc路径
/opt/gcc-4.4.4-glibc-2.11.1-multilib-1.0/arm-fsl-linux-gnueabi/bin/arm-fsl-linux-gnueabi-gccc

#交叉编译g++路径
/opt/gcc-4.4.4-glibc-2.11.1-multilib-1.0/arm-fsl-linux-gnueabi/bin/arm-fsl-linux-gnueabi-g++

#交叉编译gdb路径
/opt/gcc-4.4.4-glibc-2.11.1-multilib-1.0/arm-fsl-linux-gnueabi/bin/arm-fsl-linux-gnueabi-gdb
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200304192244796.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

然后在`构建套件(Kit)`下新建一个构建套件

```shell
#名称
imx287

#设备类型
同样Linux设备

#C/C++编译GDB
上一步设置的对应工具名称
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200304192141799.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

如果构建套件前面有红色或黄色的感叹号，说明构建套件没有设置成功。

#### 4.3 使用QtCreator涉及Hello World程序

构建套件设置完成之后，嵌入式Qt程序的开发就和桌面Qt程序的开发一样了：

- 新建工程时，勾选imx287构建套件：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200304192545823.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

- 界面设计

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200304192645173.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

- 桌面运行效果

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020030419274598.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

- 切换嵌入式构建套件
如果程序效果正常，就可以切换为嵌入式构建套件，编译出可以在嵌入式平台运行的程序了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200304193352535.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

点击下面的锤子按钮，就可以编译出可以在嵌入式平台下运行的程序了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200304193702471.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

#### 4.4 开发板运行Hello World

使用scp或者NFS共享目录的方式把文件传输到开发板：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200304193920480.png)

在开发板运行使用Qt IDE生成的可执行文件：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200304194016766.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

实际运行效果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200304194343822.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

### 5.注意

- iMX287A支持鼠标和触摸操作

如果想使用鼠标来操作，要在系统上电之前，就把鼠标插上，如果在运行过程中连接鼠标是不能使用的。

- 窗口大小自适应屏幕分辨率和隐藏标题栏

```cpp
#include <QDesktopWidget>
...........
    MainWindow::MainWindow(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::MainWindow)
{
    ui->setupUi(this);
	//不显示标题栏
    this->setWindowFlags(Qt::FramelessWindowHint);     
//    this->setWindowFlags(Qt::FramelessWindowHint | Qt::Dialog);  
	
    //获取屏幕分辨率
    const QRect availableSize = QApplication::desktop()->availableGeometry(this);
    qint16 width  = availableSize.width();
    qint16 height = availableSize.height();
    qDebug() << "width: " << width << "height:" << height;
    
	//重新设置窗口充满整个屏幕
	this->resize(width, height);
	//设置窗口大小为屏幕的1/3
//    this->resize(width/3, height/3);

	//窗口位置移动到左上角
    this->move(０, ０);
}
```

> 我的公众号：mcu149

