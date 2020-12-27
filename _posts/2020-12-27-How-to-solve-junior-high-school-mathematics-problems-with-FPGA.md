---
layout:     post
title: 如何用FPGA解一道初中数学题
subtitle:	FPGA的那些骚操作
date:       2020-12-27 16:57:00 +0800
author:     Wang Chao
header-img: img/post-bg-c-interface-implementatios.jpg
catalog:    true
tag:
    - FPGA
---

前几天和同事聊天，他说他上初中的儿子做出了一道很难的数学题，想考考我们这些大学生看能不能做得出来？

题目很简单：

![数学题目](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/201227/2020-12-21_220652.jpg)

大家先尝试做一下？我没想出怎么算的，只是用排除法确定了a和b的范围，然后再逐个尝试。

```c
1.对4361进行开方计算，得到结果最大为66，则a，b的值均小于等于66。
2.对4361/2进行开方计算，则得到结果为46，则a,b两者，一个是1-46，一个是46-66之间的数。
3.由平方和4361末尾为1，再根据整数平方和的几种可能，计算出仅有0+1和5+6这两种可能，而且平方之后的个位数为0/1/5/6，这样就进一步缩小了范围，通过多次计算尝试可以得出结果。
```

不过我懒得算了，就简单写了个C语言程序，计算出了结果：

```c
#include <stdlib.h>
#include <stdio.h>
#include <math.h>

int main(void)
{
    int num;     
    int a, b, n;
    int result;
    int sqr;

    printf("please enter a number:");//4361
    scanf("%d", &num);
    printf("input num: %d\n", num);

    sqr = sqrt(num);
    for(a = 1; a <= sqr; a++)		//可以设置1-46
    {
        for(b = 1; b <= sqr; b++)	//可以设置46-66
        {
            result = pow(a, 2) + pow(b, 2);
            if(result == num)
            {
                printf("a = %2d, b = %2d, a + b = %d\n", a, b, a+b);
                n++;
            }
        }
    }
    if(n == 0)
        printf("There is no answer!\n");

    return 0;
}
```

其实可以设置，一个数的循环范围是：1-46，一个数的循环范围是46-66，这样会减少循环次数。

运行结果：

![运行结果](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/201227/2020-12-20_144929.jpg)

而且这种方式还适用于解的个数不唯一的情况，比如7605：

![运行结果](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/201227/2020-12-27_150351.jpg)

作为一个野生FPGA开发者，我在想能不能用FPGA的编程思想来实现呢？也就是如何用Verilog来实现两个循环的嵌套呢？抄起键盘就是干！

![抄起键盘就是干](https://img-bbs.csdn.net/upload/201707/17/1500286923_459579.jpg)

verilog源文件fpga_math.v：

```c
module fpga_math(
    //inputs
    input clk,
    input rst_n,
    
    //outputs
    output reg [13:0] a, b,
    output reg [14:0] result,
    output ok
);

parameter SUM = 4361;
parameter SQR = 67;       //sqrt(SUM);

reg [13:0] tmp_a;
reg [13:0] tmp_b;
reg flag;

assign ok = (tmp_a*tmp_a + tmp_b*tmp_b == SUM);

always @ (posedge clk)
begin
    if(!rst_n)
        tmp_b <= 0;
    else if(tmp_b == SQR)
        tmp_b <= 0;
    else if(tmp_a != SQR)
        tmp_b <= tmp_b + 1;
end

always @ (posedge clk)
begin
    if(!rst_n)
        flag <= 0;
    else if(tmp_b == SQR)
        flag <= 1;
    else 
        flag <= 0;
end

always @ (posedge clk)
begin
    if(!rst_n)
        tmp_a <= 0;
    else if((tmp_a != SQR) & flag)
        tmp_a <= tmp_a + 1;
end

always @ (posedge clk)
begin
    if(!rst_n)
    begin
        a <= 0;
        b <= 0;
        result <= 0;
    end
    else if(ok)
    begin
        a <= tmp_a;
        b <= tmp_b;
        result = tmp_a + tmp_b;
    end
end

endmodule
```

为了验证这个模块的正确性，我们需要对这个模块进行仿真，即给一个激励输入信号，看输出是否正确。

新建testbench文件fpga_math_tb.v：

```c
`timescale 1ns/100ps

module fpga_math_tb;

parameter SUM = 4361;
parameter SQR = 67;     //sqrt(4361)

parameter SYSCLK_PERIOD = 10;// 100MHZ

wire [13:0] a, b;
wire [14:0] result;

reg SYSCLK;
reg NSYSRESET;

initial
begin
    SYSCLK = 1'b0;
    NSYSRESET = 1'b0;

    #(SYSCLK_PERIOD * 10 )
        NSYSRESET = 1'b1;
    #(SYSCLK_PERIOD * (SQR*SQR+500) )
        $stop;
end

/*generate clock*/
always @(SYSCLK)
    #(SYSCLK_PERIOD / 2.0) SYSCLK <= !SYSCLK;       
    
/*instance module*/
fpga_math #(
    .SUM(SUM),
    .SQR(SQR)
)fpga_math_0(
    //inputs
    .clk(SYSCLK),
    .rst_n(NSYSRESET),

    //outputs
    .a(a),
    .b(b),
    .result(result),
    .ok(ok)
);

endmodule
```

ModelSim仿真波形：

![仿真波形](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/201227/2020-12-20_151511.jpg)

仿真工具除了使用各大FPGA厂商IDE带的ModelSim等，也可以使用小巧开源的全平台仿真工具：iverilog+gtkwave，使用方法可以参考：[全平台轻量开源verilog仿真工具iverilog+GTKWave使用教程](https://mp.weixin.qq.com/s/ltUZJEJj-9_DJi1U5FqWdg)

如果使用iverilog进行仿真，需要在TB文件中添加以下几行语句：

```verilog
/*iverilog */
initial
begin            
    $dumpfile("wave.vcd");        //生成的vcd文件名称
    $dumpvars(0, fpga_math_tb);   //tb模块名称
end
/*iverilog *
```

首先对Verilog源文件进行编译，检查是否有语法错误，这会在当前目录生成wave目标文件：

```c
iverilog -o wave *.v
```

然后通过`vvp`指令，产生仿真的`wave.vcd`波形文件：

```c
vvp -n wave -lxt2
```

使用`gtkwave`打开波形文件：

```c
gtkwave wave.vcd
```

当然以上命令也可以写成批处理文件：

```bash
echo "开始编译"
iverilog -o wave *.v
echo "编译完成"
echo "生成波形文件"
vvp -n wave -lxt2
echo "打开波形文件"
gtkwave wave.vcd
```

以文本方式存储为`build.bat`文件即可，双击即可自动完成编译、生成波形文件、打开波形文件操作。

仿真波形：

![仿真波形](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/201227/2020-12-20_150902.jpg)

可以看出，和使用ModelSim仿真是一样的结果。

#### 总结

从仿真波形图中，可以得到计算的结果，a+b的值为91，如果要在真实的FPGA芯片硬件上实现，还需要添加其他功能模块，把结果通过串口输出，或者在数码管等显示屏上进行显示，这里只是简单介绍使用FPGA计算方法的实现。作为纯数字电路的FPGA，实现平方根是比较复杂的，这里采用直接人为输入平方根结果的方式，而不是像C语言那样调用sqrt函数自动计算平方根。FPGA中不仅有触发器和查找表，而且还有乘法器、除法器等硬核IP，所以在涉及到乘除法、平方根运算时，不要直接使用*/等运算符，而是要使用FPGA自带的IP核，这样就不会占用大量的逻辑资源，像Xilinx的基于Cordic算法的Cordic IP核，不仅能实现平方根计算，而且还有sin/cos/tan/arctan等三角函数。

在B站上有老师基于质数解题的视频，有兴趣的朋友可以看看：www.bilibili.com/video/av330696751
