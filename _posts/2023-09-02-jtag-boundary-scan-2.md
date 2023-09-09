---
layout:     post
title: 强大的JTAG边界扫描（2）：BSDL文件
subtitle: BSDL文件
date:       2023-09-02 11:57:00 +0800
author:     Wang Chao
header-img: img/jtag.jpg
catalog:    true
tag:
    - JTAG
---


### 1. 什么是BSDL文件？

BSDL，Boundary Scan Description Language的缩写，即边界扫描描述语言，属于VHDL的一个子集，内容符合VHDL的语法标准，用于描述JTAG在指定设备中的实现方式，只要设备符合JTAG标准，那么它必须具有对应的BSDL文件，

BSDL文件主要包括以下信息：

- 当前芯片所支持的最大TCK频率
- 定义了管脚的名称和序号
- 定义了电源、时钟、配置、IO管脚等等。每个管脚的类型，如VCC、GND、CLK，管脚的名称及序号
- 所有可用命令寄存器
- 所有可用的数据寄存器，包括可能的预设值，例如：器件的IDCODE

BSDL目前有两种标准IEEE 1149.1和IEEE 1149.6。IEEE 1149.6在IEEE 1149.1标准的基础上丰富了一些内容，它可以兼容IEEE 1149.1。

### 2. BSDL文件的获取

#### 方式1：BSDL Library

```c
https://www.bsdl.info/
```

这个网站几乎包括所有支持JTAG芯片的BSDL文件，超过100家半导体公司的上万款芯片，包括MCU、DSP、PowerPC、CPLD、FPGA等，现在还在持续更新中。

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230813/00.jpg)

支持通过芯片型号或IDCODE搜索对应的BSDL文件，可以在线进行预览，非常方便

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230813/01.jpg)

#### 方式2：各芯片的官方网站

在各大芯片厂商的官方网站一般会提供BSDL文件，下面以Xilinx、Altera、Microsemi、ST意法半导体为例，介绍如何获取BSDL文件。

##### Xilinx FPGA BSDL文件获取

Xilinx CPLD/FPGA BSDL文件一般位于开发环境ISE或Vivado安装路径下：

ISE 14.7对应路径为，例如Artix-7系列XC7A100T的BSDL文件位于：

```c
Xilinx\14.7\ISE_DS\ISE\artix7\data
```

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230813/02.jpg)

Vivado 2018.3对应路径如下：

```c
Vivado\Vivado\2018.3\ids_lite\ISE\artix7\data
```

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230813/03.jpg)

除了开发环境的安装目录，Xilinx还在官方网站上提供有各系列FPGA的BSDL文件下载：

```C
https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/device-models/bsdl-models/artix-series-fpgas.html
```

##### Altera FPGA BSDL文件获取

由于我的电脑没装Quartus开发环境，所以不确定BSDL文件是否能在安装路径下找到，Altera官方网站也可以进行下载：

IEEE 1149.1 BSDL 文件下载

```C
https://www.intel.cn/content/www/cn/zh/support/programmable/support-resources/board-layout/bsd-11491.html
```
IEEE 1149.6 BSDL 文件下载
```C
https://www.intel.cn/content/www/cn/zh/support/programmable/support-resources/board-layout/bsd-11496.html
```

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230813/06.jpg)

##### Microsemi FPGA BSDL文件获取

Microchip（Microsemi）FPGA的BSDL模型下载地址：

```c
https://www.microsemi.com/product-directory/design-resources/1717-bsdl-models
```

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230813/08.jpg)

##### ST MCU BSDL文件获取

意法半导体MCU的BSDL文件可以到官方网站搜索BSDL，就会弹出对应系列的BSDL文件包。

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230813/05.jpg)

部分系列的BSDL文件下载地址：

```c
STM32F1:
https://www.st.com/content/ccc/resource/technical/ecad_models_and_symbols/bsdl_model/75/4a/50/d0/ad/aa/49/92/stm32f1_bsdl.zip/files/stm32f1_bsdl.zip/jcr:content/translations/en.stm32f1_bsdl.zip

STM32F2:
https://www.st.com/content/ccc/resource/technical/ecad_models_and_symbols/bsdl_model/e9/d6/86/75/13/99/46/c8/stm32f2_bsdl.zip/files/stm32f2_bsdl.zip/jcr:content/translations/en.stm32f2_bsdl.zip

STM32F17:
https://www.st.com/content/ccc/resource/technical/ecad_models_and_symbols/bsdl_model/ad/a6/69/0f/70/95/49/92/stm32f7_bsdl.zip/files/stm32f7_bsdl.zip/jcr:content/translations/en.stm32f7_bsdl.zip
```

### 3. BSDL文件示例

下面是Xilinx CPLD XC95144的BSDL文件的全部内容：

```vhdl
--
--   BSDL File created/edited by BCAD BSD Editor Version 3.1
--
--BSDE:Revision: $Header: /devl/xcs/repo/env/Jobs/iMPACT/data/xc9500/xc95144.bsd,v 1.2 2000/10/24 00:58:57 sanjays Exp $
--BSDE:Description: Xilinx 144 macrocell FastFLASH ISP CPLD

entity XC95144 is

generic (PHYSICAL_PIN_MAP : string := "DIE_BOND" );

port (
    PB00_00: inout bit;
    PB00_01: inout bit;
    PB00_02: inout bit;
    PB00_03: inout bit;
    PB00_04: inout bit;
    PB00_05: inout bit;
    PB00_06: inout bit;
    PB00_07: inout bit;
    PB00_08: inout bit;
    PB00_09: inout bit;
    PB00_10: inout bit;
    PB00_11: inout bit;
    PB00_12: inout bit;
    PB00_13: inout bit;
    PB00_14: inout bit;
    PB00_15: inout bit;
    PB00_16: inout bit;
    PB01_00: inout bit;
    PB01_01: inout bit;
    PB01_02: inout bit;
    PB01_03: inout bit;
    PB01_04: inout bit;
    PB01_05: inout bit;
    PB01_06: inout bit;
    PB01_07: inout bit;
    PB01_08: inout bit;
    PB01_09: inout bit;
    PB01_10: inout bit;
    PB01_11: inout bit;
    PB01_12: inout bit;
    PB01_13: inout bit;
    PB01_14: inout bit;
    PB01_15: inout bit;
    PB01_16: inout bit;
    PB02_00: inout bit;
    PB02_01: inout bit;
    PB02_02: inout bit;
    PB02_03: inout bit;
    PB02_04: inout bit;
    PB02_05: inout bit;
    PB02_06: inout bit;
    PB02_07: inout bit;
    PB02_08: inout bit;
    PB02_09: inout bit;
    PB02_10: inout bit;
    PB02_11: inout bit;
    PB02_12: inout bit;
    PB02_13: inout bit;
    PB02_14: inout bit;
    PB02_15: inout bit;
    PB02_16: inout bit;
    PB03_00: inout bit;
    PB03_01: inout bit;
    PB03_02: inout bit;
    PB03_03: inout bit;
    PB03_04: inout bit;
    PB03_05: inout bit;
    PB03_06: inout bit;
    PB03_07: inout bit;
    PB03_08: inout bit;
    PB03_09: inout bit;
    PB03_10: inout bit;
    PB03_11: inout bit;
    PB03_12: inout bit;
    PB03_13: inout bit;
    PB03_14: inout bit;
    PB03_15: inout bit;
    PB03_16: inout bit;
    PB04_00: inout bit;
    PB04_01: inout bit;
    PB04_02: inout bit;
    PB04_03: inout bit;
    PB04_04: inout bit;
    PB04_05: inout bit;
    PB04_06: inout bit;
    PB04_07: inout bit;
    PB04_08: inout bit;
    PB04_09: inout bit;
    PB04_10: inout bit;
    PB04_11: inout bit;
    PB04_12: inout bit;
    PB04_13: inout bit;
    PB04_14: inout bit;
    PB04_15: inout bit;
    PB04_16: inout bit;
    PB05_01: inout bit;
    PB05_02: inout bit;
    PB05_03: inout bit;
    PB05_04: inout bit;
    PB05_05: inout bit;
    PB05_06: inout bit;
    PB05_07: inout bit;
    PB05_08: inout bit;
    PB05_09: inout bit;
    PB05_10: inout bit;
    PB05_11: inout bit;
    PB05_12: inout bit;
    PB05_13: inout bit;
    PB05_14: inout bit;
    PB05_15: inout bit;
    PB05_16: inout bit;
    PB06_01: inout bit;
    PB06_02: inout bit;
    PB06_03: inout bit;
    PB06_04: inout bit;
    PB06_05: inout bit;
    PB06_06: inout bit;
    PB06_07: inout bit;
    PB06_08: inout bit;
    PB06_09: inout bit;
    PB06_10: inout bit;
    PB06_11: inout bit;
    PB06_12: inout bit;
    PB06_13: inout bit;
    PB06_14: inout bit;
    PB06_15: inout bit;
    PB06_16: inout bit;
    PB07_01: inout bit;
    PB07_02: inout bit;
    PB07_03: inout bit;
    PB07_04: inout bit;
    PB07_05: inout bit;
    PB07_06: inout bit;
    PB07_07: inout bit;
    PB07_08: inout bit;
    PB07_09: inout bit;
    PB07_10: inout bit;
    PB07_11: inout bit;
    PB07_12: inout bit;
    PB07_13: inout bit;
    PB07_14: inout bit;
    PB07_15: inout bit;
    PB07_16: inout bit;
    TCK: in bit;
    TDI: in bit;
    TDO: out bit;
    TMS: in bit;
    VCCINT_1: linkage bit;
    VCCINT_2: linkage bit;
    VCCINT_3: linkage bit;
    VCCINT_VPP: linkage bit;
    VCCIO_1: linkage bit;
    VCCIO_2: linkage bit;
    VCCIO_3: linkage bit;
    VCCIO_4: linkage bit;
    VCCIO_5: linkage bit;
    VCCIO_6: linkage bit;
    VSSINT_1: linkage bit;
    VSSINT_2: linkage bit;
    VSSINT_3: linkage bit;
    VSSINT_4: linkage bit;
    VSSIO_1: linkage bit;
    VSSIO_2: linkage bit;
    VSSIO_3: linkage bit;
    VSSIO_4: linkage bit;
    VSSIO_5: linkage bit;
    VSSIO_6: linkage bit;
    VSSIO_7: linkage bit;
    VSSIO_8: linkage bit;
    VSSIO_9: linkage bit
);

use STD_1149_1_1990.all;

attribute PIN_MAP of XC95144 : entity is PHYSICAL_PIN_MAP;

constant DIE_BOND: PIN_MAP_STRING:=
    "PB00_00:PAD25," &
    "PB00_01:PAD18," &
    "PB00_02:PAD19," &
    "PB00_03:PAD27," &
    "PB00_04:PAD21," &
    "PB00_05:PAD22," &
    "PB00_06:PAD32," &
    "PB00_07:PAD23," &
    "PB00_08:PAD24," &
    "PB00_09:PAD34," &
    "PB00_10:PAD26," &
    "PB00_11:PAD28," &
    "PB00_12:PAD38," &
    "PB00_13:PAD29," &
    "PB00_14:PAD30," &
    "PB00_15:PAD39," &
    "PB00_16:PAD33," &
    "PB01_00:PAD158," &
    "PB01_01:PAD159," &
    "PB01_02:PAD3," &
    "PB01_03:PAD5," &
    "PB01_04:PAD2," &
    "PB01_05:PAD4," &
    "PB01_06:PAD7," &
    "PB01_07:PAD6," &
    "PB01_08:PAD8," &
    "PB01_09:PAD9," &
    "PB01_10:PAD11," &
    "PB01_11:PAD12," &
    "PB01_12:PAD14," &
    "PB01_13:PAD13," &
    "PB01_14:PAD15," &
    "PB01_15:PAD16," &
    "PB01_16:PAD17," &
    "PB02_00:PAD43," &
    "PB02_01:PAD35," &
    "PB02_02:PAD45," &
    "PB02_03:PAD48," &
    "PB02_04:PAD36," &
    "PB02_05:PAD37," &
    "PB02_06:PAD50," &
    "PB02_07:PAD42," &
    "PB02_08:PAD44," &
    "PB02_09:PAD52," &
    "PB02_10:PAD47," &
    "PB02_11:PAD49," &
    "PB02_12:PAD53," &
    "PB02_13:PAD54," &
    "PB02_14:PAD56," &
    "PB02_15:PAD55," &
    "PB02_16:PAD57," &
    "PB03_00:PAD132," &
    "PB03_01:PAD140," &
    "PB03_02:PAD147," &
    "PB03_03:PAD149," &
    "PB03_04:PAD142," &
    "PB03_05:PAD143," &
    "PB03_06:PAD150," &
    "PB03_07:PAD144," &
    "PB03_08:PAD145," &
    "PB03_09:PAD151," &
    "PB03_10:PAD146," &
    "PB03_11:PAD148," &
    "PB03_12:PAD153," &
    "PB03_13:PAD152," &
    "PB03_14:PAD154," &
    "PB03_15:PAD155," &
    "PB03_16:PAD156," &
    "PB04_00:PAD65," &
    "PB04_01:PAD58," &
    "PB04_02:PAD66," &
    "PB04_03:PAD67," &
    "PB04_04:PAD59," &
    "PB04_05:PAD60," &
    "PB04_06:PAD74," &
    "PB04_07:PAD62," &
    "PB04_08:PAD63," &
    "PB04_09:PAD76," &
    "PB04_10:PAD64," &
    "PB04_11:PAD68," &
    "PB04_12:PAD78," &
    "PB04_13:PAD69," &
    "PB04_14:PAD72," &
    "PB04_15:PAD83," &
    "PB04_16:PAD77," &
    "PB05_01:PAD117," &
    "PB05_02:PAD119," &
    "PB05_03:PAD123," &
    "PB05_04:PAD122," &
    "PB05_05:PAD124," &
    "PB05_06:PAD125," &
    "PB05_07:PAD126," &
    "PB05_08:PAD129," &
    "PB05_09:PAD128," &
    "PB05_10:PAD133," &
    "PB05_11:PAD134," &
    "PB05_12:PAD130," &
    "PB05_13:PAD135," &
    "PB05_14:PAD138," &
    "PB05_15:PAD131," &
    "PB05_16:PAD139," &
    "PB06_01:PAD79," &
    "PB06_02:PAD84," &
    "PB06_03:PAD85," &
    "PB06_04:PAD82," &
    "PB06_05:PAD86," &
    "PB06_06:PAD87," &
    "PB06_07:PAD88," &
    "PB06_08:PAD90," &
    "PB06_09:PAD89," &
    "PB06_10:PAD92," &
    "PB06_11:PAD95," &
    "PB06_12:PAD91," &
    "PB06_13:PAD96," &
    "PB06_14:PAD97," &
    "PB06_15:PAD93," &
    "PB06_16:PAD98," &
    "PB07_01:PAD101," &
    "PB07_02:PAD105," &
    "PB07_03:PAD107," &
    "PB07_04:PAD102," &
    "PB07_05:PAD103," &
    "PB07_06:PAD109," &
    "PB07_07:PAD104," &
    "PB07_08:PAD106," &
    "PB07_09:PAD112," &
    "PB07_10:PAD108," &
    "PB07_11:PAD111," &
    "PB07_12:PAD114," &
    "PB07_13:PAD113," &
    "PB07_14:PAD115," &
    "PB07_15:PAD118," &
    "PB07_16:PAD116," &
    "TCK:PAD75," &
    "TDI:PAD71," &
    "TDO:PAD136," &
    "TMS:PAD73," &
    "VCCINT_1:PAD46," &
    "VCCINT_2:PAD94," &
    "VCCINT_3:PAD157," &
    "VCCINT_VPP:PAD10," &
    "VCCIO_1:PAD1," &
    "VCCIO_2:PAD41," &
    "VCCIO_3:PAD61," &
    "VCCIO_4:PAD81," &
    "VCCIO_5:PAD121," &
    "VCCIO_6:PAD141," &
    "VSSINT_1:PAD31," &
    "VSSINT_2:PAD70," &
    "VSSINT_3:PAD100," &
    "VSSINT_4:PAD127," &
    "VSSIO_1:PAD20," &
    "VSSIO_2:PAD40," &
    "VSSIO_3:PAD51," &
    "VSSIO_4:PAD80," &
    "VSSIO_5:PAD99," &
    "VSSIO_6:PAD110," &
    "VSSIO_7:PAD120," &
    "VSSIO_8:PAD137," &
    "VSSIO_9:PAD160";

attribute TAP_SCAN_IN    of TDI : signal is true;
attribute TAP_SCAN_OUT   of TDO : signal is true;
attribute TAP_SCAN_MODE  of TMS : signal is true;
attribute TAP_SCAN_CLOCK of TCK : signal is (1.00e+07, BOTH);
attribute INSTRUCTION_LENGTH of XC95144 : entity is 8;

attribute INSTRUCTION_OPCODE of XC95144 : entity is
    "BYPASS ( 11111111)," &
    "CONLD ( 11110000)," &
    "EXTEST ( 00000000)," &
    "FERASE ( 11101100)," &
    "FBULK ( 11101101)," &
    "FPGM ( 11101010)," &
    "FPGMI ( 11101011)," &
    "FVFY ( 11101110)," &
    "FVFYI ( 11101111)," &
    "HIGHZ ( 11111100)," &
    "IDCODE ( 11111110)," &
    "INTEST ( 00000010)," &
    "ISCEN ( 11101000)," &
    "SAMPLE ( 00000001)," &
    "USERCODE ( 11111101)" ;

attribute INSTRUCTION_CAPTURE of XC95144 : entity is "000XXX01";

attribute INSTRUCTION_DISABLE of XC95144 : entity is "HIGHZ";

attribute IDCODE_REGISTER of XC95144 : entity is
    "0010" & "1001010100001000" & "00001001001" & "1";

attribute USERCODE_REGISTER of XC95144 : entity is
    "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX";

attribute REGISTER_ACCESS of XC95144 : entity is
    "BYPASS ( CONLD, HIGHZ )," &
    "ISCENABLE[12] ( ISCEN)," &
    "ISCONFIGURATION[27] ( FERASE, FBULK, FPGM, FVFY)," &
    "ISCDATA[10] ( FPGMI, FVFYI)";

attribute BOUNDARY_CELLS of XC95144 : entity is
        " BC_1";

attribute BOUNDARY_LENGTH of XC95144 : entity is 432;

attribute BOUNDARY_REGISTER of XC95144 : entity is
    "   0 (BC_1, *, internal, X)," &
    "   1 (BC_1, *, internal, X)," &
    "   2 (BC_1, *, internal, X)," &
    "   3 (BC_1, *, controlr, 0)," &
    "   4 (BC_1, PB07_16, output3, X, 3, 0, Z)," &
    "   5 (BC_1, PB07_16, input, X)," &
    "   6 (BC_1, *, controlr, 0)," &
    "   7 (BC_1, PB07_15, output3, X, 6, 0, Z)," &
    "   8 (BC_1, PB07_15, input, X)," &
    "   9 (BC_1, *, controlr, 0)," &
    "  10 (BC_1, PB07_14, output3, X, 9, 0, Z)," &
    "  11 (BC_1, PB07_14, input, X)," &
    "  12 (BC_1, *, controlr, 0)," &
    "  13 (BC_1, PB07_13, output3, X, 12, 0, Z)," &
    "  14 (BC_1, PB07_13, input, X)," &
    "  15 (BC_1, *, controlr, 0)," &
    "  16 (BC_1, PB07_12, output3, X, 15, 0, Z)," &
    "  17 (BC_1, PB07_12, input, X)," &
    "  18 (BC_1, *, controlr, 0)," &
    "  19 (BC_1, PB07_11, output3, X, 18, 0, Z)," &
    "  20 (BC_1, PB07_11, input, X)," &
    "  21 (BC_1, *, controlr, 0)," &
    "  22 (BC_1, PB07_10, output3, X, 21, 0, Z)," &
    "  23 (BC_1, PB07_10, input, X)," &
    "  24 (BC_1, *, controlr, 0)," &
    "  25 (BC_1, PB07_09, output3, X, 24, 0, Z)," &
    "  26 (BC_1, PB07_09, input, X)," &
    "  27 (BC_1, *, controlr, 0)," &
    "  28 (BC_1, PB07_08, output3, X, 27, 0, Z)," &
    "  29 (BC_1, PB07_08, input, X)," &
    "  30 (BC_1, *, controlr, 0)," &
    "  31 (BC_1, PB07_07, output3, X, 30, 0, Z)," &
    "  32 (BC_1, PB07_07, input, X)," &
    "  33 (BC_1, *, controlr, 0)," &
    "  34 (BC_1, PB07_06, output3, X, 33, 0, Z)," &
    "  35 (BC_1, PB07_06, input, X)," &
    "  36 (BC_1, *, controlr, 0)," &
    "  37 (BC_1, PB07_05, output3, X, 36, 0, Z)," &
    "  38 (BC_1, PB07_05, input, X)," &
    "  39 (BC_1, *, controlr, 0)," &
    "  40 (BC_1, PB07_04, output3, X, 39, 0, Z)," &
    "  41 (BC_1, PB07_04, input, X)," &
    "  42 (BC_1, *, controlr, 0)," &
    "  43 (BC_1, PB07_03, output3, X, 42, 0, Z)," &
    "  44 (BC_1, PB07_03, input, X)," &
    "  45 (BC_1, *, controlr, 0)," &
    "  46 (BC_1, PB07_02, output3, X, 45, 0, Z)," &
    "  47 (BC_1, PB07_02, input, X)," &
    "  48 (BC_1, *, controlr, 0)," &
    "  49 (BC_1, PB07_01, output3, X, 48, 0, Z)," &
    "  50 (BC_1, PB07_01, input, X)," &
    "  51 (BC_1, *, internal, X)," &
    "  52 (BC_1, *, internal, X)," &
    "  53 (BC_1, *, internal, X)," &
    "  54 (BC_1, *, internal, X)," &
    "  55 (BC_1, *, internal, X)," &
    "  56 (BC_1, *, internal, X)," &
    "  57 (BC_1, *, controlr, 0)," &
    "  58 (BC_1, PB06_16, output3, X, 57, 0, Z)," &
    "  59 (BC_1, PB06_16, input, X)," &
    "  60 (BC_1, *, controlr, 0)," &
    "  61 (BC_1, PB06_15, output3, X, 60, 0, Z)," &
    "  62 (BC_1, PB06_15, input, X)," &
    "  63 (BC_1, *, controlr, 0)," &
    "  64 (BC_1, PB06_14, output3, X, 63, 0, Z)," &
    "  65 (BC_1, PB06_14, input, X)," &
    "  66 (BC_1, *, controlr, 0)," &
    "  67 (BC_1, PB06_13, output3, X, 66, 0, Z)," &
    "  68 (BC_1, PB06_13, input, X)," &
    "  69 (BC_1, *, controlr, 0)," &
    "  70 (BC_1, PB06_12, output3, X, 69, 0, Z)," &
    "  71 (BC_1, PB06_12, input, X)," &
    "  72 (BC_1, *, controlr, 0)," &
    "  73 (BC_1, PB06_11, output3, X, 72, 0, Z)," &
    "  74 (BC_1, PB06_11, input, X)," &
    "  75 (BC_1, *, controlr, 0)," &
    "  76 (BC_1, PB06_10, output3, X, 75, 0, Z)," &
    "  77 (BC_1, PB06_10, input, X)," &
    "  78 (BC_1, *, controlr, 0)," &
    "  79 (BC_1, PB06_09, output3, X, 78, 0, Z)," &
    "  80 (BC_1, PB06_09, input, X)," &
    "  81 (BC_1, *, controlr, 0)," &
    "  82 (BC_1, PB06_08, output3, X, 81, 0, Z)," &
    "  83 (BC_1, PB06_08, input, X)," &
    "  84 (BC_1, *, controlr, 0)," &
    "  85 (BC_1, PB06_07, output3, X, 84, 0, Z)," &
    "  86 (BC_1, PB06_07, input, X)," &
    "  87 (BC_1, *, controlr, 0)," &
    "  88 (BC_1, PB06_06, output3, X, 87, 0, Z)," &
    "  89 (BC_1, PB06_06, input, X)," &
    "  90 (BC_1, *, controlr, 0)," &
    "  91 (BC_1, PB06_05, output3, X, 90, 0, Z)," &
    "  92 (BC_1, PB06_05, input, X)," &
    "  93 (BC_1, *, controlr, 0)," &
    "  94 (BC_1, PB06_04, output3, X, 93, 0, Z)," &
    "  95 (BC_1, PB06_04, input, X)," &
    "  96 (BC_1, *, controlr, 0)," &
    "  97 (BC_1, PB06_03, output3, X, 96, 0, Z)," &
    "  98 (BC_1, PB06_03, input, X)," &
    "  99 (BC_1, *, controlr, 0)," &
    " 100 (BC_1, PB06_02, output3, X, 99, 0, Z)," &
    " 101 (BC_1, PB06_02, input, X)," &
    " 102 (BC_1, *, controlr, 0)," &
    " 103 (BC_1, PB06_01, output3, X, 102, 0, Z)," &
    " 104 (BC_1, PB06_01, input, X)," &
    " 105 (BC_1, *, internal, X)," &
    " 106 (BC_1, *, internal, X)," &
    " 107 (BC_1, *, internal, X)," &
    " 108 (BC_1, *, internal, X)," &
    " 109 (BC_1, *, internal, X)," &
    " 110 (BC_1, *, internal, X)," &
    " 111 (BC_1, *, controlr, 0)," &
    " 112 (BC_1, PB05_16, output3, X, 111, 0, Z)," &
    " 113 (BC_1, PB05_16, input, X)," &
    " 114 (BC_1, *, controlr, 0)," &
    " 115 (BC_1, PB05_15, output3, X, 114, 0, Z)," &
    " 116 (BC_1, PB05_15, input, X)," &
    " 117 (BC_1, *, controlr, 0)," &
    " 118 (BC_1, PB05_14, output3, X, 117, 0, Z)," &
    " 119 (BC_1, PB05_14, input, X)," &
    " 120 (BC_1, *, controlr, 0)," &
    " 121 (BC_1, PB05_13, output3, X, 120, 0, Z)," &
    " 122 (BC_1, PB05_13, input, X)," &
    " 123 (BC_1, *, controlr, 0)," &
    " 124 (BC_1, PB05_12, output3, X, 123, 0, Z)," &
    " 125 (BC_1, PB05_12, input, X)," &
    " 126 (BC_1, *, controlr, 0)," &
    " 127 (BC_1, PB05_11, output3, X, 126, 0, Z)," &
    " 128 (BC_1, PB05_11, input, X)," &
    " 129 (BC_1, *, controlr, 0)," &
    " 130 (BC_1, PB05_10, output3, X, 129, 0, Z)," &
    " 131 (BC_1, PB05_10, input, X)," &
    " 132 (BC_1, *, controlr, 0)," &
    " 133 (BC_1, PB05_09, output3, X, 132, 0, Z)," &
    " 134 (BC_1, PB05_09, input, X)," &
    " 135 (BC_1, *, controlr, 0)," &
    " 136 (BC_1, PB05_08, output3, X, 135, 0, Z)," &
    " 137 (BC_1, PB05_08, input, X)," &
    " 138 (BC_1, *, controlr, 0)," &
    " 139 (BC_1, PB05_07, output3, X, 138, 0, Z)," &
    " 140 (BC_1, PB05_07, input, X)," &
    " 141 (BC_1, *, controlr, 0)," &
    " 142 (BC_1, PB05_06, output3, X, 141, 0, Z)," &
    " 143 (BC_1, PB05_06, input, X)," &
    " 144 (BC_1, *, controlr, 0)," &
    " 145 (BC_1, PB05_05, output3, X, 144, 0, Z)," &
    " 146 (BC_1, PB05_05, input, X)," &
    " 147 (BC_1, *, controlr, 0)," &
    " 148 (BC_1, PB05_04, output3, X, 147, 0, Z)," &
    " 149 (BC_1, PB05_04, input, X)," &
    " 150 (BC_1, *, controlr, 0)," &
    " 151 (BC_1, PB05_03, output3, X, 150, 0, Z)," &
    " 152 (BC_1, PB05_03, input, X)," &
    " 153 (BC_1, *, controlr, 0)," &
    " 154 (BC_1, PB05_02, output3, X, 153, 0, Z)," &
    " 155 (BC_1, PB05_02, input, X)," &
    " 156 (BC_1, *, controlr, 0)," &
    " 157 (BC_1, PB05_01, output3, X, 156, 0, Z)," &
    " 158 (BC_1, PB05_01, input, X)," &
    " 159 (BC_1, *, internal, X)," &
    " 160 (BC_1, *, internal, X)," &
    " 161 (BC_1, *, internal, X)," &
    " 162 (BC_1, *, internal, X)," &
    " 163 (BC_1, *, internal, X)," &
    " 164 (BC_1, *, internal, X)," &
    " 165 (BC_1, *, controlr, 0)," &
    " 166 (BC_1, PB04_16, output3, X, 165, 0, Z)," &
    " 167 (BC_1, PB04_16, input, X)," &
    " 168 (BC_1, *, controlr, 0)," &
    " 169 (BC_1, PB04_15, output3, X, 168, 0, Z)," &
    " 170 (BC_1, PB04_15, input, X)," &
    " 171 (BC_1, *, controlr, 0)," &
    " 172 (BC_1, PB04_14, output3, X, 171, 0, Z)," &
    " 173 (BC_1, PB04_14, input, X)," &
    " 174 (BC_1, *, controlr, 0)," &
    " 175 (BC_1, PB04_13, output3, X, 174, 0, Z)," &
    " 176 (BC_1, PB04_13, input, X)," &
    " 177 (BC_1, *, controlr, 0)," &
    " 178 (BC_1, PB04_12, output3, X, 177, 0, Z)," &
    " 179 (BC_1, PB04_12, input, X)," &
    " 180 (BC_1, *, controlr, 0)," &
    " 181 (BC_1, PB04_11, output3, X, 180, 0, Z)," &
    " 182 (BC_1, PB04_11, input, X)," &
    " 183 (BC_1, *, controlr, 0)," &
    " 184 (BC_1, PB04_10, output3, X, 183, 0, Z)," &
    " 185 (BC_1, PB04_10, input, X)," &
    " 186 (BC_1, *, controlr, 0)," &
    " 187 (BC_1, PB04_09, output3, X, 186, 0, Z)," &
    " 188 (BC_1, PB04_09, input, X)," &
    " 189 (BC_1, *, controlr, 0)," &
    " 190 (BC_1, PB04_08, output3, X, 189, 0, Z)," &
    " 191 (BC_1, PB04_08, input, X)," &
    " 192 (BC_1, *, controlr, 0)," &
    " 193 (BC_1, PB04_07, output3, X, 192, 0, Z)," &
    " 194 (BC_1, PB04_07, input, X)," &
    " 195 (BC_1, *, controlr, 0)," &
    " 196 (BC_1, PB04_06, output3, X, 195, 0, Z)," &
    " 197 (BC_1, PB04_06, input, X)," &
    " 198 (BC_1, *, controlr, 0)," &
    " 199 (BC_1, PB04_05, output3, X, 198, 0, Z)," &
    " 200 (BC_1, PB04_05, input, X)," &
    " 201 (BC_1, *, controlr, 0)," &
    " 202 (BC_1, PB04_04, output3, X, 201, 0, Z)," &
    " 203 (BC_1, PB04_04, input, X)," &
    " 204 (BC_1, *, controlr, 0)," &
    " 205 (BC_1, PB04_03, output3, X, 204, 0, Z)," &
    " 206 (BC_1, PB04_03, input, X)," &
    " 207 (BC_1, *, controlr, 0)," &
    " 208 (BC_1, PB04_02, output3, X, 207, 0, Z)," &
    " 209 (BC_1, PB04_02, input, X)," &
    " 210 (BC_1, *, controlr, 0)," &
    " 211 (BC_1, PB04_01, output3, X, 210, 0, Z)," &
    " 212 (BC_1, PB04_01, input, X)," &
    " 213 (BC_1, *, controlr, 0)," &
    " 214 (BC_1, PB04_00, output3, X, 213, 0, Z)," &
    " 215 (BC_1, PB04_00, input, X)," &
    " 216 (BC_1, *, internal, X)," &
    " 217 (BC_1, *, internal, X)," &
    " 218 (BC_1, *, internal, X)," &
    " 219 (BC_1, *, controlr, 0)," &
    " 220 (BC_1, PB03_16, output3, X, 219, 0, Z)," &
    " 221 (BC_1, PB03_16, input, X)," &
    " 222 (BC_1, *, controlr, 0)," &
    " 223 (BC_1, PB03_15, output3, X, 222, 0, Z)," &
    " 224 (BC_1, PB03_15, input, X)," &
    " 225 (BC_1, *, controlr, 0)," &
    " 226 (BC_1, PB03_14, output3, X, 225, 0, Z)," &
    " 227 (BC_1, PB03_14, input, X)," &
    " 228 (BC_1, *, controlr, 0)," &
    " 229 (BC_1, PB03_13, output3, X, 228, 0, Z)," &
    " 230 (BC_1, PB03_13, input, X)," &
    " 231 (BC_1, *, controlr, 0)," &
    " 232 (BC_1, PB03_12, output3, X, 231, 0, Z)," &
    " 233 (BC_1, PB03_12, input, X)," &
    " 234 (BC_1, *, controlr, 0)," &
    " 235 (BC_1, PB03_11, output3, X, 234, 0, Z)," &
    " 236 (BC_1, PB03_11, input, X)," &
    " 237 (BC_1, *, controlr, 0)," &
    " 238 (BC_1, PB03_10, output3, X, 237, 0, Z)," &
    " 239 (BC_1, PB03_10, input, X)," &
    " 240 (BC_1, *, controlr, 0)," &
    " 241 (BC_1, PB03_09, output3, X, 240, 0, Z)," &
    " 242 (BC_1, PB03_09, input, X)," &
    " 243 (BC_1, *, controlr, 0)," &
    " 244 (BC_1, PB03_08, output3, X, 243, 0, Z)," &
    " 245 (BC_1, PB03_08, input, X)," &
    " 246 (BC_1, *, controlr, 0)," &
    " 247 (BC_1, PB03_07, output3, X, 246, 0, Z)," &
    " 248 (BC_1, PB03_07, input, X)," &
    " 249 (BC_1, *, controlr, 0)," &
    " 250 (BC_1, PB03_06, output3, X, 249, 0, Z)," &
    " 251 (BC_1, PB03_06, input, X)," &
    " 252 (BC_1, *, controlr, 0)," &
    " 253 (BC_1, PB03_05, output3, X, 252, 0, Z)," &
    " 254 (BC_1, PB03_05, input, X)," &
    " 255 (BC_1, *, controlr, 0)," &
    " 256 (BC_1, PB03_04, output3, X, 255, 0, Z)," &
    " 257 (BC_1, PB03_04, input, X)," &
    " 258 (BC_1, *, controlr, 0)," &
    " 259 (BC_1, PB03_03, output3, X, 258, 0, Z)," &
    " 260 (BC_1, PB03_03, input, X)," &
    " 261 (BC_1, *, controlr, 0)," &
    " 262 (BC_1, PB03_02, output3, X, 261, 0, Z)," &
    " 263 (BC_1, PB03_02, input, X)," &
    " 264 (BC_1, *, controlr, 0)," &
    " 265 (BC_1, PB03_01, output3, X, 264, 0, Z)," &
    " 266 (BC_1, PB03_01, input, X)," &
    " 267 (BC_1, *, controlr, 0)," &
    " 268 (BC_1, PB03_00, output3, X, 267, 0, Z)," &
    " 269 (BC_1, PB03_00, input, X)," &
    " 270 (BC_1, *, internal, X)," &
    " 271 (BC_1, *, internal, X)," &
    " 272 (BC_1, *, internal, X)," &
    " 273 (BC_1, *, controlr, 0)," &
    " 274 (BC_1, PB02_16, output3, X, 273, 0, Z)," &
    " 275 (BC_1, PB02_16, input, X)," &
    " 276 (BC_1, *, controlr, 0)," &
    " 277 (BC_1, PB02_15, output3, X, 276, 0, Z)," &
    " 278 (BC_1, PB02_15, input, X)," &
    " 279 (BC_1, *, controlr, 0)," &
    " 280 (BC_1, PB02_14, output3, X, 279, 0, Z)," &
    " 281 (BC_1, PB02_14, input, X)," &
    " 282 (BC_1, *, controlr, 0)," &
    " 283 (BC_1, PB02_13, output3, X, 282, 0, Z)," &
    " 284 (BC_1, PB02_13, input, X)," &
    " 285 (BC_1, *, controlr, 0)," &
    " 286 (BC_1, PB02_12, output3, X, 285, 0, Z)," &
    " 287 (BC_1, PB02_12, input, X)," &
    " 288 (BC_1, *, controlr, 0)," &
    " 289 (BC_1, PB02_11, output3, X, 288, 0, Z)," &
    " 290 (BC_1, PB02_11, input, X)," &
    " 291 (BC_1, *, controlr, 0)," &
    " 292 (BC_1, PB02_10, output3, X, 291, 0, Z)," &
    " 293 (BC_1, PB02_10, input, X)," &
    " 294 (BC_1, *, controlr, 0)," &
    " 295 (BC_1, PB02_09, output3, X, 294, 0, Z)," &
    " 296 (BC_1, PB02_09, input, X)," &
    " 297 (BC_1, *, controlr, 0)," &
    " 298 (BC_1, PB02_08, output3, X, 297, 0, Z)," &
    " 299 (BC_1, PB02_08, input, X)," &
    " 300 (BC_1, *, controlr, 0)," &
    " 301 (BC_1, PB02_07, output3, X, 300, 0, Z)," &
    " 302 (BC_1, PB02_07, input, X)," &
    " 303 (BC_1, *, controlr, 0)," &
    " 304 (BC_1, PB02_06, output3, X, 303, 0, Z)," &
    " 305 (BC_1, PB02_06, input, X)," &
    " 306 (BC_1, *, controlr, 0)," &
    " 307 (BC_1, PB02_05, output3, X, 306, 0, Z)," &
    " 308 (BC_1, PB02_05, input, X)," &
    " 309 (BC_1, *, controlr, 0)," &
    " 310 (BC_1, PB02_04, output3, X, 309, 0, Z)," &
    " 311 (BC_1, PB02_04, input, X)," &
    " 312 (BC_1, *, controlr, 0)," &
    " 313 (BC_1, PB02_03, output3, X, 312, 0, Z)," &
    " 314 (BC_1, PB02_03, input, X)," &
    " 315 (BC_1, *, controlr, 0)," &
    " 316 (BC_1, PB02_02, output3, X, 315, 0, Z)," &
    " 317 (BC_1, PB02_02, input, X)," &
    " 318 (BC_1, *, controlr, 0)," &
    " 319 (BC_1, PB02_01, output3, X, 318, 0, Z)," &
    " 320 (BC_1, PB02_01, input, X)," &
    " 321 (BC_1, *, controlr, 0)," &
    " 322 (BC_1, PB02_00, output3, X, 321, 0, Z)," &
    " 323 (BC_1, PB02_00, input, X)," &
    " 324 (BC_1, *, internal, X)," &
    " 325 (BC_1, *, internal, X)," &
    " 326 (BC_1, *, internal, X)," &
    " 327 (BC_1, *, controlr, 0)," &
    " 328 (BC_1, PB01_16, output3, X, 327, 0, Z)," &
    " 329 (BC_1, PB01_16, input, X)," &
    " 330 (BC_1, *, controlr, 0)," &
    " 331 (BC_1, PB01_15, output3, X, 330, 0, Z)," &
    " 332 (BC_1, PB01_15, input, X)," &
    " 333 (BC_1, *, controlr, 0)," &
    " 334 (BC_1, PB01_14, output3, X, 333, 0, Z)," &
    " 335 (BC_1, PB01_14, input, X)," &
    " 336 (BC_1, *, controlr, 0)," &
    " 337 (BC_1, PB01_13, output3, X, 336, 0, Z)," &
    " 338 (BC_1, PB01_13, input, X)," &
    " 339 (BC_1, *, controlr, 0)," &
    " 340 (BC_1, PB01_12, output3, X, 339, 0, Z)," &
    " 341 (BC_1, PB01_12, input, X)," &
    " 342 (BC_1, *, controlr, 0)," &
    " 343 (BC_1, PB01_11, output3, X, 342, 0, Z)," &
    " 344 (BC_1, PB01_11, input, X)," &
    " 345 (BC_1, *, controlr, 0)," &
    " 346 (BC_1, PB01_10, output3, X, 345, 0, Z)," &
    " 347 (BC_1, PB01_10, input, X)," &
    " 348 (BC_1, *, controlr, 0)," &
    " 349 (BC_1, PB01_09, output3, X, 348, 0, Z)," &
    " 350 (BC_1, PB01_09, input, X)," &
    " 351 (BC_1, *, controlr, 0)," &
    " 352 (BC_1, PB01_08, output3, X, 351, 0, Z)," &
    " 353 (BC_1, PB01_08, input, X)," &
    " 354 (BC_1, *, controlr, 0)," &
    " 355 (BC_1, PB01_07, output3, X, 354, 0, Z)," &
    " 356 (BC_1, PB01_07, input, X)," &
    " 357 (BC_1, *, controlr, 0)," &
    " 358 (BC_1, PB01_06, output3, X, 357, 0, Z)," &
    " 359 (BC_1, PB01_06, input, X)," &
    " 360 (BC_1, *, controlr, 0)," &
    " 361 (BC_1, PB01_05, output3, X, 360, 0, Z)," &
    " 362 (BC_1, PB01_05, input, X)," &
    " 363 (BC_1, *, controlr, 0)," &
    " 364 (BC_1, PB01_04, output3, X, 363, 0, Z)," &
    " 365 (BC_1, PB01_04, input, X)," &
    " 366 (BC_1, *, controlr, 0)," &
    " 367 (BC_1, PB01_03, output3, X, 366, 0, Z)," &
    " 368 (BC_1, PB01_03, input, X)," &
    " 369 (BC_1, *, controlr, 0)," &
    " 370 (BC_1, PB01_02, output3, X, 369, 0, Z)," &
    " 371 (BC_1, PB01_02, input, X)," &
    " 372 (BC_1, *, controlr, 0)," &
    " 373 (BC_1, PB01_01, output3, X, 372, 0, Z)," &
    " 374 (BC_1, PB01_01, input, X)," &
    " 375 (BC_1, *, controlr, 0)," &
    " 376 (BC_1, PB01_00, output3, X, 375, 0, Z)," &
    " 377 (BC_1, PB01_00, input, X)," &
    " 378 (BC_1, *, internal, X)," &
    " 379 (BC_1, *, internal, X)," &
    " 380 (BC_1, *, internal, X)," &
    " 381 (BC_1, *, controlr, 0)," &
    " 382 (BC_1, PB00_16, output3, X, 381, 0, Z)," &
    " 383 (BC_1, PB00_16, input, X)," &
    " 384 (BC_1, *, controlr, 0)," &
    " 385 (BC_1, PB00_15, output3, X, 384, 0, Z)," &
    " 386 (BC_1, PB00_15, input, X)," &
    " 387 (BC_1, *, controlr, 0)," &
    " 388 (BC_1, PB00_14, output3, X, 387, 0, Z)," &
    " 389 (BC_1, PB00_14, input, X)," &
    " 390 (BC_1, *, controlr, 0)," &
    " 391 (BC_1, PB00_13, output3, X, 390, 0, Z)," &
    " 392 (BC_1, PB00_13, input, X)," &
    " 393 (BC_1, *, controlr, 0)," &
    " 394 (BC_1, PB00_12, output3, X, 393, 0, Z)," &
    " 395 (BC_1, PB00_12, input, X)," &
    " 396 (BC_1, *, controlr, 0)," &
    " 397 (BC_1, PB00_11, output3, X, 396, 0, Z)," &
    " 398 (BC_1, PB00_11, input, X)," &
    " 399 (BC_1, *, controlr, 0)," &
    " 400 (BC_1, PB00_10, output3, X, 399, 0, Z)," &
    " 401 (BC_1, PB00_10, input, X)," &
    " 402 (BC_1, *, controlr, 0)," &
    " 403 (BC_1, PB00_09, output3, X, 402, 0, Z)," &
    " 404 (BC_1, PB00_09, input, X)," &
    " 405 (BC_1, *, controlr, 0)," &
    " 406 (BC_1, PB00_08, output3, X, 405, 0, Z)," &
    " 407 (BC_1, PB00_08, input, X)," &
    " 408 (BC_1, *, controlr, 0)," &
    " 409 (BC_1, PB00_07, output3, X, 408, 0, Z)," &
    " 410 (BC_1, PB00_07, input, X)," &
    " 411 (BC_1, *, controlr, 0)," &
    " 412 (BC_1, PB00_06, output3, X, 411, 0, Z)," &
    " 413 (BC_1, PB00_06, input, X)," &
    " 414 (BC_1, *, controlr, 0)," &
    " 415 (BC_1, PB00_05, output3, X, 414, 0, Z)," &
    " 416 (BC_1, PB00_05, input, X)," &
    " 417 (BC_1, *, controlr, 0)," &
    " 418 (BC_1, PB00_04, output3, X, 417, 0, Z)," &
    " 419 (BC_1, PB00_04, input, X)," &
    " 420 (BC_1, *, controlr, 0)," &
    " 421 (BC_1, PB00_03, output3, X, 420, 0, Z)," &
    " 422 (BC_1, PB00_03, input, X)," &
    " 423 (BC_1, *, controlr, 0)," &
    " 424 (BC_1, PB00_02, output3, X, 423, 0, Z)," &
    " 425 (BC_1, PB00_02, input, X)," &
    " 426 (BC_1, *, controlr, 0)," &
    " 427 (BC_1, PB00_01, output3, X, 426, 0, Z)," &
    " 428 (BC_1, PB00_01, input, X)," &
    " 429 (BC_1, *, controlr, 0)," &
    " 430 (BC_1, PB00_00, output3, X, 429, 0, Z)," &
    " 431 (BC_1, PB00_00, input, X)";

end XC95144;
```

### 4. BSDL文件的应用

BSDL文件可以在一些边界扫描的软件中被使用，如XJTAG，TopJTAG等等，通过加载对应的BSDL文件可以实现对芯片外部所有管脚的读取和控制。具体使用方法，我会在后面的文章介绍。

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230813/04.jpg)

![](https://wcc-blog.oss-accelerate.aliyuncs.com/img/230813/07.jpg)