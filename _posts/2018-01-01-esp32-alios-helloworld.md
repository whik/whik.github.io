---
layout:     post
title:      ESP32 DevKitC 编译烧写 AliOS Things
subtitle:	基于官方 Hello World 例程（Windows 环境）
date:       2018-01-01
author:     Shao Guoji
header-img: img/post-bg-esp32-aos.jpg
catalog:    true
tag:
    - 单片机
    - ESP32
    - 物联网
    - 嵌入式
---

本文介绍 Windows 下基于 AliOS Things 的 ESP32 应用开发流程，包括环境搭建、程序编译、固件烧写。

### AliOS Things

![图1 AliOS Things](http://odaps2f9v.bkt.clouddn.com/18-1-1/40637909.jpg)

> AliOS Things 是一款由阿里巴巴开发的轻量级物联网操作系统。具备极致性能，极简开发、云端一体、丰富组件（包括实时操作系统内核，连接协议库、文件系统、libc接口、FOTA、Mesh、语音识别）、安全防护等关键能力，并支持终端设备连接到阿里云IoT云服务平台。可广泛应用在智能家居，智慧城市，工业等领域，降低物联网终端开发门槛，使万物互联更容易，终端设备上云更简单。

项目地址：[alibaba/AliOS-Things: AliOS Things released by Alibaba is an open-source implementation of operating system (OS) for Internet of Things (IoT).](https://github.com/alibaba/AliOS-Things/)

在嵌入式实时操作系统大家族中，**常见的 µC/OS-III、FreeRTOS 等 RTOS 严格意义上只能算一个 kernel（仅包含 OS 基本服务）**，随着物联网时代到来，出现了像 AliOS Things、RT-Thread 这些「时髦」的操作系统，**大佬们在实时内核的基础上增加了大量组件，囊括通信协议栈、低功耗管理、安全加密算法、FOTA（远程固件升级）等功能，可以说目的十分明确 —— 直奔物联网**。

*更多关于物联网操作系统的知识，可以参考何小庆老师的 PPT [物联网操作系统研究与思考.pdf](http://allanhe.xtreemhost.com/wp-content/uploads/2016/05/%E7%89%A9%E8%81%94%E7%BD%91%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E7%A0%94%E7%A9%B6%E4%B8%8E%E6%80%9D%E8%80%83-201706.pdf)*

---

### ESP32 

![图2 Espressif ESP32](http://odaps2f9v.bkt.clouddn.com/18-1-1/33324134.jpg)

物联网的大潮下，MCU 迎来一个新的发展机遇。继 ESP8266 之后，乐鑫在 2015 年底又推出了更强大的 ESP32 系列 WiFi 芯片，从参数描述可以看出：

> ESP32 SoC 为双核 32 位 MCU，主频高达 240 MHz，计算性能可达 600 DMIPS，采用 40 nm 工艺，集成 520 KB SRAM，16 MByte flash。工作电压 2.2 V to 3.6 V。ESP32 专为移动设备、可穿戴电子产品和 IoT 应用而设计，拥有业内最高水平的低功耗芯片的所有特征，例如精细分辨时钟门控、省电模式和动态电压调整等。ESP32 SoC工作温度范围从-40°C 到 +125°C。此外，ESP32 还集成了先进的自校准电路，实现了动态自动调整，可以消除外部电路的缺陷以及适应外部条件的变化。

早在 2016 年乐鑫 ESP32 和阿里云物联网系统 YoC 已经有了合作。去年 10 月份的云栖大会上阿里提出了 AliOS Things，不久之后项目开源便支持了 ESP32，同时为开发者提供了许多开发工具。

#### ESP32 DevKitC 开发板

![图3 ESP32 DevKitC 开发板](http://odaps2f9v.bkt.clouddn.com/18-1-2/79192501.jpg)

ESP32-DevKitC 是搭载了乐鑫最新的 ESP-WROOM-32 模组的 MINI 开发板，能够轻松地插接到面包板，板子包含了用户所需的最小系统，只需连上 USB 线，即可进行开发。此外还具有 USB-UART 转换器 ，复位和下载模式按钮，LDO 稳压器 和微型 USB 连接器 。每个 GPIO 都可供开发者使用。

*开发板购买地址：[ESP32-DevkitC (Core board开发板)发票不含快递费-淘宝网](https://item.taobao.com/item.htm?_u=2oh9n95b094&id=542143157571)*

那如何把 AliOS Things 编译烧写到 ESP32 DevKitC 呢？来不及解释了，赶紧上车！

---

### 所需工具

在 Windows 下进行基于 AliOS Things 开发 ESP32 应用需要准备 

* 安装有 Windows、Linux 或者 Mac 操作系统的 PC
* 用于编译 ESP32 应用程序的工具链
* AliOS Things SDK —— 包含 ESP32 的 API 和用于操作工具链的脚本
* 编写 C 语言程序的文本编辑器，例如 VS Code
* ESP32 开发板，例如 ESP32-DevKitC

---

### 开发步骤

1. 安装 Visual Studio Code
2. 安装 alios-studio 扩展
3. 获取 AliOS Things SDK（Github 项目源码）
4. 下载 ESP32 工具链
5. 配置 SDK path，新建 helloworld 工程
6. 编译、构建项目
7. 烧写 bin 固件

---

### Step 1：安装 VS Code、alios-studio 扩展

项目 Wiki 已经有详细软件安装说明文档，按照步骤把 VS Code 和 alios-studio 扩展装好即可：[AliOS Things Studio · alibaba/AliOS-Things Wiki](https://github.com/alibaba/AliOS-Things/wiki/AliOS-Things-Studio)

![图4 安装 VS Code 与 alios-studio](http://odaps2f9v.bkt.clouddn.com/18-1-2/56288242.jpg)

---

### Step 2：获取 AliOS Things SDK 和 ESP32 工具链

#### 下载 aos 源代码

**SDK 即项目仓库源码**，从 Github 上 Download Zip 或 Clone 到本地，拷贝到 `D:\AliOS-Things-master` 目录下（目录自定）。

#### 下载 ESP32 工具链

乐鑫 ESP-IDF 文档中详细描述了如何搭建 ESP32 开发环境，我们需要工具链 Windows all-in-one toolchain 用于编译源代码。

直接下载官方提供的 zip 包即可：[https://dl.espressif.com/dl/esp32_win32_msys2_environment_and_toolchain-20171123.zip](https://dl.espressif.com/dl/esp32_win32_msys2_environment_and_toolchain-20171123.zip)

同样解压到 `D:\msys32` 目录下，SDK 和 Toolchain 用于后续配置 VS Code 开发环境。

---

### Step 3：配置 SDK path 与 Toolchain path

打开装好 aos-studio 扩展的 VS Code，点击右下角边状态栏上的 Create Project 新建工程按钮，首次使用会弹出配置 SDK Path 的窗口，把刚才下载的 SDK 目录 `D:\AliOS-Things-master` 和 ToolChain 目录 `D:\msys32\opt\xtensa-esp32-elf\bin` 复制到两个 path 文本框中， SDK Version 会自动识别。

![图5 添加 SDK path 与 Toolchain path](http://odaps2f9v.bkt.clouddn.com/18-1-2/83573576.jpg)

#### 20180110 更新：V1.2.0 版本 SDK 设置工具链方式

由于新版本发布后增加了多编译器支持，在 SDK 设置界面去掉了 Toolchain 的选择，而是通过工程配置文件指定工具链目录。在左侧项目管理处找到 `.vscode` 下的 `setting.json` 文件，打开后在第一项 `aliosStudio.build.toolchain` 中填入目录 `D:\msys32\opt\xtensa-esp32-elf\bin` 即可**（注意反斜杠需要转义）**。

```json
{
    "aliosStudio.build.toolchain": "D:\\msys32\\opt\\xtensa-esp32-elf\\bin",
    "aliosStudio.build.output": "${workspaceRoot}/out",
    "aliosStudio.inner.yosBin": "aos",
    ...
}
```

嫌麻烦的话还可以在系统高级设置中将 ToolChain 路径添加到 TOOLCHAIN_PATH 环境变量中。

#### （已过时）修改 alios-studio Toolchain 判断规则（项目完善后可跳过此步）

**此步骤非必须，由于我目前使用的 v1.1.2 版本的 SDK 尚未完善，多少会存在一些小 bug。**比如对所添加 Toolchain path 的合法性判断，目前只校验了 arm Toolchain，所以对 ESP32 的 xtensa 工具链会误报非法路径。

阿里工程师在 issue [[AliOS-studio][ESP32] tool chain path. · Issue #55 · alibaba/AliOS-Things](https://github.com/alibaba/AliOS-Things/issues/55) 里给出了临时解决方法 —— 把 `C:\Users\用户名\.vscode\extensions\alios.alios-studio-0.6.6\src\yang-sdk\main.js` 文件中以下判断语句注释掉： 

```javascript
if (options.toolChain && !fsPlus.existsSync(path.join(dir, `arm-none-eabi-gcc${process.platform === 'win32' ? '.exe' : ''}`))) {
    return {error: 1, msg: 'path is not a tool chain directory!'};
}
```

**修改完毕后重启 VS Code，按 Ctrl+Shift+P 快捷键调出命令模式，输入 alios-studio: set SDK 继续配置，可正常添加 Toolchain path。**

![图6 成功添加 Toolchain path](http://odaps2f9v.bkt.clouddn.com/18-1-2/17069465.jpg)

---

### Step 4：新建 helloworld 工程

继续点击新建工程按钮，这回我们终于看到了待选工程模板，勾选 helloworld 工程，滚到下面填写工程保存目录（**不能带中文**），目标板卡选 esp32devkitc，最后点 Submit 按钮确认提交。

![图7 新建工程](http://odaps2f9v.bkt.clouddn.com/18-1-2/27413940.jpg)

新建工程需要把 sdk 赋值到工程文件夹，比较耗时，请耐心等待。完成后 VS Code 会新打开一个文件夹视图窗口，表示一个 alios-studio 工程。在左侧的目录中打开 `helloword.c` 文件，其中 `application_start` 函数是应用程序的入口。**helloworld 程序的运行现象是在串口以 5 s 的间隔打印调试字符串。**

![图8 工程代码](http://odaps2f9v.bkt.clouddn.com/18-1-2/7029919.jpg)

新建工程视频演示：[AliOS Things Tutorial: 1 Hello World 应用](http://v.youku.com/v_show/id_XMzI2MTYyNDAwOA)

---

### Step 5：编译、构建项目

点击窗口下方状态栏上的 Build 按钮开始编译、构建项目，期间会输出相应信息。**如果 SDK 和工具链路径配置 ok 的话项目是可以成功编译的。**经过 67.57 s 的漫长等待，终于 Build 完了……

![图9 构建项目](http://odaps2f9v.bkt.clouddn.com/18-1-2/95932236.jpg)

编译生成的文件在工程的 out 目录下，`out\helloworld@esp32devkitc\binary\helloworld@esp32devkitc.bin` 是要烧到板子上的固件。

---

### Step 6：烧写 bin 固件

*说明：由于官方 IDE 暂不支持 Upload 按钮烧录 ESP32，目前只能手动烧录。*

固件烧录是相对独立的过程，原理适用于所有 bin 文件。烧写 ESP32 固件可以通过图形界面的 ESPFlashDownloadTool 软件或者 Python 命令行工具 esptool，两者都十分好上手，下面分别说明烧录方法。

#### bin 文件烧录地址

在烧写前需要准备 3 个 bin 文件，分别是引导程序（bootloader.bin）、分区表（custom_partitions.bin）和用户程序（helloworld@esp32devkitc.bin）。

引导程序和分区表的 bin 文件在 SDK 目录 `D:\AliOS-Things-master\platform\mcu\esp32\bsp` 下，用户程序 bin 由 alios-studio 编译得到。3 个 bin 文件烧录地址如下：

| bin 文件                    | 烧录地址 |
|-----------------------------|----------|
| bootloader.bin              | 0x1000   |
| custom_partitions.bin       | 0x8000   |
| helloworld@esp32devkitc.bin | 0x10000  |

系统启动时会从 0x1000 地址处开始执行，引导程序读取分区表确定内存分布及启动规则，然后执行用户程序代码。**因此全部 bin 都要烧到正确的地址程序才能正常执行**，这一点需要特别注意。

*PS：bootloader.bin 和 custom_partitions.bin 首次必须烧写，之后仅烧用户 bin 即可。*

#### 20180110 更新：使用 IDE 烧录

好消息！好消息！在更新后的 alios-studio 中支持通过下方 Upload 按钮一键烧录（然后选择对应 COM 号，波特率默认），你不点一下试试么？

#### 使用 ESPFlashDownloadTool 工具烧录

ESPFlashDownloadTool 工具可在 [Tools - 乐鑫 Flash 下载工具](http://espressif.com/zh-hans/support/download/other-tools)下载，打开软件后选择 ESP32 DownloadTool，设置不同固件及对应地址、晶振频率、SPI 模式、Flash大小，波特率（决定烧写速度），如下图所示：

![图10 使用 ESPFlashDownloadTool 工具烧录](http://odaps2f9v.bkt.clouddn.com/18-1-2/13291389.jpg)

将 ESP32 DevKitC 开发板用 Micro-USB 线与电脑连接，安装[串口驱动](https://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers)，在烧写软件中选择对应 COM 号，点击 Start 按钮开始下载。

**提示：大部分电脑在点击 Start 后会自动复位 ESP32 DevKitC 进入下载模式，如果出现一直等待的情况，请尝试按住 Boot 键不放再下载，或者按住 BooT 键的的同时按一下 EN 键再松开。**

#### 使用 esptool 工具烧录

esptool 是采用 Python 语言编写的开源工具（源代码：[espressif/esptool: ESP8266 and ESP32 serial bootloader utility](https://github.com/espressif/esptool)），提供方便易用的命令行方式进行操作，常用操作命令：

单个 bin 写 flash 命令：

```
格式：esptool.py --port 串口号 write_flash 地址 文件名.bin

示例：esptool.py --port COM4 write_flash 0x1000 my_app-0x01000.bin
```

多个 bin 写 flash 命令：

```
格式：esptool.py --port 串口号 write_flash 地址1 文件名1.bin 地址2 文件名2.bin

示例：esptool.py --port COM4 write_flash 0x00000 my_app.elf-0x00000.bin 0x40000 my_app.elf-0x40000.bin
```

esptool.py 在 `D:\AliOS-Things-master\platform\mcu\esp32\esptool_py\esptool` 目录下，可通过「计算机 - 属性 - 高级系统设置 - 环境变量」添加到系统环境变量 Path 中（分号隔开后粘贴路径），以便在命令行中直接使用。

在 CMD 中敲入以下命令，将 bootloader.bin、custom_partitions.bin 和 helloworld@esp32devkitc.bin 三个文件下载到 Flash 对应地址。

```
esptool.py --port COM30 --baud 921600 write_flash 0x1000 D:\AliOS-Things-master\platform\mcu\esp32\bsp\bootloader.bin 0x8000 D:\AliOS-Things-master\platform\mcu\esp32\bsp\custom_partitions.bin 0x10000 E:\CodeBase\ESP32\AliOS-Things\hello\out\helloworld@esp32devkitc\binary\helloworld@esp32devkitc.bin
```

![图11 esptool 烧写固件](http://odaps2f9v.bkt.clouddn.com/18-1-2/18579232.jpg)

#### 脚本实现

每次输命令太麻烦？将以下代码保存为批处理脚本 `upload.bat` ，并拷贝到工程目录 hello 下，最后在 VS Code 内置的终端中执行脚本实现一键烧录：

```
for /f "delims=" %%t in ('dir /A:-D /S /B out\*@esp32devkitc.bin') do set binPath=%%t

esptool.py --port COM30 --baud 921600 write_flash 0x1000 D:\AliOS-Things-master\platform\mcu\esp32\bsp\bootloader.bin 0x8000 D:\AliOS-Things-master\platform\mcu\esp32\bsp\custom_partitions.bin 0x10000 %binPath%
```

![图12 VS Code 脚本下载](http://odaps2f9v.bkt.clouddn.com/18-1-2/19319700.jpg)

*固件 `bootloader.bin` 和 `custom_partitions.bin` 从 SDK 目录获取，用户 bin 通过子目录下搜索 "@esp32devkitc.bin" 文件后缀得到。*

---

### 运行结果

点击 VS Code 下方的 Connect Device 按钮（选好 COM 号，波特率 115200），通过 alios-studio 自带串口工具连接开发板（或使用其他串口工具），**如果收到 ESP32 每隔 5 s 发过来的调试信息，说明 helloworld 运行成功！**

![图13 运行效果](http://odaps2f9v.bkt.clouddn.com/18-1-2/45756837.jpg)

*提示：如果板子不断重启打印错误信息，请检查固件及烧写地址的正确性。*


> 参考资料
> 
> * [RT-Thread物联网操作系统](https://www.rt-thread.org/)
> * [物联网操作系统研究与思考 - 何小庆](http://allanhe.xtreemhost.com/wp-content/uploads/2016/05/%E7%89%A9%E8%81%94%E7%BD%91%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E7%A0%94%E7%A9%B6%E4%B8%8E%E6%80%9D%E8%80%83-201706.pdf)
> * [乐鑫发布ESP32芯片 整合WIFI和低功耗蓝牙 为物联网创客带来新福利 - 吴川斌的博客](http://www.mr-wu.cn/espressif-esp32-soc/)
> * [物联网 WIFI 芯片乐鑫 ESP32 结合阿里云物联网系统 YoC](http://www.mr-wu.cn/espressif-esp32-soc-and-yun-on-chip/)
> * [Home · alibaba/AliOS-Things Wiki](https://github.com/alibaba/AliOS-Things/wiki)
> * [ESP32-DevKitC Getting Started Guide](https://cdn.sos.sk/productdata/05/82/cd413a9e/esp32-devkitc.pdf)