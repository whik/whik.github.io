---
layout:     post
title:    全平台轻量开源verilog仿真工具iverilog+GTKWave使用教程
subtitle:	iverilog教程
date:       2019-12-03 22:22:40 +0800
author:     Wang Chao
header-img: img/bg4.jpg
catalog:    true
tag:
    - Verilog
    - FPGA
    - 硬件
---

### 前言

如果你只是想检查Verilog文件的语法是否有错误，然后进行一些基本的时序仿真，那么Icarus Verilog 就是一个不错的选择。相比于各大FPGA厂商的IDE几个G的大小，Icarus Verilog 显得极其小巧，最新版安装包大小仅有17MB，支持全平台：Windows+Linux+MacOS，并且源代码开源。本文将介绍如何使用Icarus Verilog来进行verilog文件的编译和仿真。

### 关于 Icarus Verilog 

Icarus Verilog是一个轻量、免费、开源的Verilog编译器，基于C++实现，开发者是 Stephen Williams ，遵循 [GNU GPL license](http://www.gnu.org/licenses/gpl.html) 许可证，安装文件中已经包含 GTKWave支持Verilog/VHDL文件的编译和仿真，命令行操作方式，类似gcc编译器，通过testbench文件可以生成对应的仿真波形数据文件，通过自带的GTKWave可以查看仿真波形图，支持将Verilog转换为VHDL文件。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/iverilog/build.gif)

### iverilog的安装

iverilog安装时，默认会把GTKWave一起安装，用于查看生成的波形图。

iverilog支持Windows、Linux和MacOS三大主流平台，截止2019年12月1日，最新版本v11-20190809下载： 

http://bleyer.org/icarus/iverilog-v11-20190809-x64_setup.exe

#### Windows下的安装

Windows下直接双击上面下载的安装文件即可，安装完成后安装目录如下：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/iverilog/%E5%AE%89%E8%A3%85%E7%9B%AE%E5%BD%95%E6%96%87%E4%BB%B6.jpg)

#### Linux下的安装

Linux下的安装，以Ubuntu 16.04为例，可以通过apt-get直接安装。

- 安装iverilog：`sudo apt-get install iverilog`
- 安装GTKWave：`sudo apt-get install gtkwave`

不能成功安装的，尝试更换镜像地址，我使用的是网易的开源镜像地址。

#### MacOS下的安装

Mac下的安装可以通过 macports 或者 homebrew 来安装，

通过 Macports 安装：

- 安装iverilog：`sudo ports -d -v install iverilog`
- 安装GTKWave：`sudo ports -d -v install gtkwave`

通过 homebrew 安装：

- 安装iverilog：` brew install icarus-verilog`
- 安装GTKWave：`brew install caskroom/cask/gtkwave`

#### 查看是否安装成功

安装成功后，可以通过命令窗口来查看命令所在的路径。

Windows环境可以通过where命令查看安装路径

```

where iverilog
where vvp
where gtkwave

```

Linux环境可以通过which命令查看安装路径

```

which iverilog
which vvp
which gtkwave

```

### 基本参数介绍

Icarus Verilog编译器主要包含3个工具：

- iverilog：用于编译verilog和vhdl文件，进行语法检查，生成可执行文件
- vvp：根据可执行文件，生成仿真波形文件
- gtkwave：用于打开仿真波形文件，图形化显示波形

在终端输入`iverilog`回车，可以看到常用参数使用方法的简单介绍：

```

$ iverilog
D:\iverilog\bin\iverilog.exe: no source files.

Usage: iverilog [-EiSuvV] [-B base] [-c cmdfile|-f cmdfile]
                [-g1995|-g2001|-g2005|-g2005-sv|-g2009|-g2012] [-g<feature>]
                [-D macro[=defn]] [-I includedir]
                [-M [mode=]depfile] [-m module]
                [-N file] [-o filename] [-p flag=value]
                [-s topmodule] [-t target] [-T min|typ|max]
                [-W class] [-y dir] [-Y suf] [-l file] source_file(s)

See the man page for details.

```

下面来详细介绍几个常用参数的使用方法。

#### 参数-o

这是比较常用的一个参数了，和GCC中-o的使用几乎一样，用于指定生成文件的名称。如果不指定，默认生成文件名为a.out。如：`iverilog -o test test.v `

#### 参数-y

用于指定包含文件夹，如果top.v中调用了其他的的.v模块，top.v直接编译会提示

```

led_demo_tb.v:38: error: Unknown module type: led_demo
2 error(s) during elaboration.
*** These modules were missing:
        led_demo referenced 1 times.
***

```

找不到调用的模块，那么就需要指定调用模块所在文件夹的路径，支持相对路径和绝对路径。

如：`iverilog -y D:/test/demo led_demo_tb.v`

如果是同一目录下：`iverilog -y ./ led_demo_tb.v`，另外，iverilog还支持Xilinx、Altera、Lattice等FPGA厂商的仿真库，需要在编译时通过-y参数指定库文件的路径，详细的使用方法可以查看官方用户指南：

https://iverilog.fandom.com/wiki/User_Guide 

#### 参数-I

如果程序使用`include语句包含了头文件路径，可以通过-i参数指定文件路径，使用方法和-y参数一样。

如：`iverilog -I D:/test/demo led_demo_tb.v`

#### 参数-tvhdl 

iverilog还支持把verilog文件转换为VHDL文件，如` iverilog -tvhdl -o out_file.vhd in_file.v `

### Verilog的编译仿真实际应用

新建led_demo.v源文件，内容如下：

```

module led_demo(
	input clk,
	input rst_n,

	output reg led
);

reg [7:0] cnt;

always @ (posedge clk)
begin
	if(!rst_n)
		cnt <= 0;
	else if(cnt >= 10)
		cnt <= 0;
	else 
		cnt <= cnt + 1;
end

always @ (posedge clk)
begin
	if(!rst_n)
		led <= 0;
	else if(cnt == 10)
		led <= !led;
end

endmodule 

```

功能非常简单，每10个时钟周期，led翻转一次。

仿真testbench文件led_demo_tb.v，内容如下：

```

`timescale 1ns/100ps

module led_demo_tb;

parameter SYSCLK_PERIOD = 10;

reg SYSCLK;
reg NSYSRESET;

initial
begin
    SYSCLK = 1'b0;
    NSYSRESET = 1'b0;
end

/*iverilog */
initial
begin            
    $dumpfile("wave.vcd");        //生成的vcd文件名称
    $dumpvars(0, led_demo_tb);    //tb模块名称
end
/*iverilog */

initial
begin
    #(SYSCLK_PERIOD * 10 )
        NSYSRESET = 1'b1;
	#1000
		$stop;
end

always @(SYSCLK)
    #(SYSCLK_PERIOD / 2.0) SYSCLK <= !SYSCLK;

led_demo led_demo_ut0 (
    // Inputs
    .rst_n(NSYSRESET),
    .clk(SYSCLK),

    // Outputs
    .led( led)
);

endmodule

```

注意testbench文件中有几行iverilog编译器专用的语句，如果不加的话后面不能生成vcd文件。

```

initial
begin            
    $dumpfile("wave.vcd");        //生成的vcd文件名称
    $dumpvars(0, led_demo_tb);    //tb模块名称
end

```

#### 1.编译

通过`iverilog -o wave led_demo_tb.v led_demo.v`命令，对源文件和仿真文件，进行语法规则检查和编译。由于本示例比较简单，只有1个文件，如果调用了多个.v的模块，可以通过前面介绍的-y参数指定源文件的路径，否则编译报错。如果源文件都在同同一个目录，可以直接通过`./`绝对路径的方式来指定。

例如，led_demo_tb.v中调用了led_demo.v模块，就可以直接使用`iverilog -o wave -y ./ top.v top_tb.v`来进行编译。

如果编译成功，会在当前目录下生成名称为wave的文件。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/iverilog/build.gif)

#### 2.生成波形文件

使用`vvp -n wave -lxt2`命令生成vcd波形文件，运行之后，会在当前目录下生成.vcd文件。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/iverilog/vvp.gif)

如果没有生成，需要检查testbench文件中是否添加了如下几行：

```

initial
begin            
    $dumpfile("wave.vcd");        //生成的vcd文件名称
    $dumpvars(0, led_demo_tb);    //tb模块名称
end 

```

#### 3.打开波形文件

使用命令`gtkwave wave.vcd`，可以在图形化界面中查看仿真的波形图。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/iverilog/gtkwave.gif)

### Verilog转换为VHDL

虽然VHDL和Verilog都诞生于20世纪80年代，而且都属于硬件描述语言（HDL），但是二者的语法特性却不一样。Icarus Verilog 还有一个小功能就是支持把使用Verilog语言编写的.v文件转换为VHDL语言的.vhd文件。

如把led_demo.v文件转换为VHDL文件led_demo.vhd，使用命令`iverilog -tvhdl -o led_demo.vhd led_demo.v`。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/iverilog/vhdl.gif)

生成的VHDL文件内容如下：

```

-- This VHDL was converted from Verilog using the
-- Icarus Verilog VHDL Code Generator 11.0 (devel) (s20150603-612-ga9388a89)

library ieee;
use ieee.std_logic_1164.all;
use ieee.numeric_std.all;

-- Generated from Verilog module led_demo (led_demo.v:1)
entity led_demo is
  port (
    clk : in std_logic;
    led : out std_logic;
    rst_n : in std_logic
  );
end entity;

-- Generated from Verilog module led_demo (led_demo.v:1)
architecture from_verilog of led_demo is
  signal led_Reg : std_logic;
  signal cnt : unsigned(7 downto 0);  -- Declared at led_demo.v:8
begin
  led <= led_Reg;

  -- Generated from always process in led_demo (led_demo.v:10)
  process (clk) is
  begin
    if rising_edge(clk) then
      if (not rst_n) = '1' then
        cnt <= X"00";
      else
        if Resize(cnt, 32) >= X"0000000a" then
          cnt <= X"00";
        else
          cnt <= cnt + X"01";
        end if;
      end if;
    end if;
  end process;

  -- Generated from always process in led_demo (led_demo.v:20)
  process (clk) is
  begin
    if rising_edge(clk) then
      if (not rst_n) = '1' then
        led_Reg <= '0';
      else
        if Resize(cnt, 32) = X"0000000a" then
          led_Reg <= not led_Reg;
        end if;
      end if;
    end if;
  end process;
end architecture;

```

### VHDL文件的编译和仿真

如果你还和编译Verilog一样，使用`iverilog led_dmeo.v`来编译VHDL文件的话，那么会提示有语法错误，这是正常的，因为Verilog和VHDL是不同的语法规则，不能使用Verilog的标准来检查VHDL文件的语法。需要添加`-g2012`参数来对VHDL文件进行编译，如`iverilog -g2012 led_demo.vhd`，和Verilog一样，同样也支持Testbech文件的编译和仿真，当然需要编写对应的VHDL Testbench文件。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/iverilog/vhdl_build.gif)

### 批处理文件一键执行

通过批处理文件，可以简化编译仿真的执行过程，直接一键执行编译和仿真。

新建文本文档，输入以下内容：

```

echo "开始编译"
iverilog -o wave led_demo.v led_demo_tb.v
echo "编译完成"
vvp -n wave -lxt2
echo "生成波形文件"
cp wave.vcd wave.lxt
echo "打开波形文件"
gtkwave wave.lxt

```

文件扩展名需要更改，Windows系统保存为.bat文件，Linux系统保存为.sh文件。Windows直接双击运行，Linux在终端执行。

### 总结

从20040706版本，到现在的最新版本20190809，作者还在继续更新，有兴趣的朋友可以研究一下源代码是如何实现语法规则检查的，或者可以尝试编译源码，获得最新的版本。当然，和FPGA厂商的IDE相比，功能还是非常有限，GTKWave界面也比较简陋，如不支持宽度测量等，主要是小巧+全平台支持，可以配合IDE来使用。这个工具还支持主流FPGA厂商的IP核仿真，如Xilinx和Lattice，详细的使用方法可以参考官方使用指南。

### 参考资料

文章部分内容参考自Icarus Verilog官方网站。

- iverilog官网： http://iverilog.icarus.com/ 
- iverilog下载：http://bleyer.org/icarus/ 
- iverilog用户指南： https://iverilog.fandom.com/wiki/User_Guide 
- Github开源地址： https://github.com/steveicarus/iverilog 
- GTKWave下载(iverilog已经包含)：[http://gtkwave.sourceforge.net/](http://gtkwave.sourceforge.net/)

### 推荐阅读

- [详解EMC测试国家标准GB/T 17626](http://www.wangchaochao.top/2019/11/24/Detailed-explanation-of-EMC-test-national-standards/)
- [电路板上的这些标志你都知道是什么含义吗？](http://www.wangchaochao.top/2019/11/17/Certification/)
- [Microsemi Libero使用技巧——FPGA全局网络的设置](http://www.wangchaochao.top/2019/11/30/Libero-Skill-6/)
- [Microsemi Libero系列教程（一）——Libero开发环境介绍、下载、安装与注册](http://www.wangchaochao.top/2019/05/23/Libero-1/)
- [Microsemi Libero系列教程（二）——新建点灯工程](http://www.wangchaochao.top/2019/09/29/Libero-2/)
- [详解串行通信协议及其FPGA实现](http://www.wangchaochao.top/2019/08/23/UART-Simple/)

----

- 我的个人博客：www.wangchaochao.top
- 我的公众号：mcu149

 