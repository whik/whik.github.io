---
layout:     post
title:    详解串行通信协议及其FPGA实现
subtitle:	 通信协议
date:       2019-08-23 21:55:40 +0800
author:     Wang Chao
header-img: img/post-bg-c-interface-implementatios.jpg
catalog:    true
tag:
    - 硬件
    - Verilog
    - FPGA
    - STM32
    - 单片机
---


### 前言

好久没更新博客了，这篇文章写写停停，用了近一周的时间，终于写完了。本篇文章介绍，串口协议数据帧格式、串行通信的工作方式、电平标准、编码方式及Verilog实现串口发送一个字节数据和接收一个字节数据。

对于MCU串口的发送接收，可能就是1行代码就能实现串口的发送和接收：

#### STM32的串口接收和发送


	//STM32发送1个字节
	USART_SendData(USART1, 'A'); 
	while (USART_GetFlagStatus(DEBUG_USARTx, USART_FLAG_TXE) == RESET);		

	//STM32接收1个字节：
	uint8_t Res;
	while (USART_GetFlagStatus(USART1, USART_FLAG_RXNE) == RESET);
	Res = USART_ReceiveData(USART1);


#### 51单片机的发送和接收


	//51单片机发送1个字节
	SBUF = 'A;
	while(!TI);
	TI=0;


	//51单片机接收1个字节：
	char Res;
	if(RI)
	{
	   Res = SBUF;
	   RI = 0;
	}



更方便一点的，通过重写C库fput函数和fgetc函数，还可以实现printf直接**重定向到串口**，用来输出一些调试信息再方便不过了。

#### STM32实现输入输出重定向到串口发送接收


	//可重定向printf函数
	int fputc(int ch, FILE *f)
	{
			USART_SendData(DEBUG_USARTx, (uint8_t) ch);
			while (USART_GetFlagStatus(DEBUG_USARTx, USART_FLAG_TXE) == RESET);		
			return (ch);
	}
	//可重定向scanf函数
	int fgetc(FILE *f)
	{
			while (USART_GetFlagStatus(DEBUG_USARTx, USART_FLAG_RXNE) == RESET);
			return (int)USART_ReceiveData(DEBUG_USARTx);
	}


而MCU上的串口是半导体厂商预先设计好的，几乎是MCU的标配，高度集成，使用起来十分方便，但是串口的引脚基本上是固定的，不可以更改。对于硬件橡皮泥——FPGA来说，需要使用HDL从底层串口数据帧来实现，可以直接在任意一个引脚实现串口功能。为了用Verilog HDL实现标准的串口通讯协议，我们有必要先来详细了解一下串口通讯协议。

### 串口数据帧格式

#### 波特率

波特率，即比特率（Baud rate），即通信双方“沟通的语言”，通信双方要设置为一样的波特率才可以正常通信。表示每秒发送的二进制位数，即传输1位的时间是：1/波特率 秒，如，波特率9600bps，即每秒传输9600bit，那么每一位的时间为：1/9600 s = 104.1666us，常用的波特率有：4800/9600/115200/12800等等，也可以根据需要自定义波特率大小，如1M或者3M，但是有的PC或者USB-TTL模块不支持太高速度的波特率，常用的USB-TTL芯片有：CH340，CP2102，PL2103，FT232等，其中FT232HL芯片**最大支持12M的波特率**，当然价格也比其他芯片高一些。

#### 起始位和停止位

数据帧从起始位开始，到停止位结束。起始信号用逻辑0表示，而停止位是用逻辑1表示，一般有0.5位、1位、1.5位或2位停止位，常用的一般是1位停止位，只要通信双方约定一致即可。

#### 数据位

起始位之后，紧跟着的是数据位，低位（LSB）在前，高位（MSB）在后，一般有5位、6位、7位和8位数据位，常用的是8位数据位，因为一个字节正好是8位。

#### 校验位

校验位一般用来判断接收的数据位有无错误，校验方法有：奇校验（odd）、偶校验（even）、0校验（space）、1校验（mark）及无校验（noparity）。奇校验要求有效数据和校验位中“1”的个数为奇数，比如一个8位长的有效数据为：01101001，此时共有4个“1”，为达到奇数个"1"的效果，校验位为“1”，让“1”的个数变成5个（奇数）。偶校验刚好相反，要求有效数据和校验位的“1”数量为偶数，则此时为达到偶校验效果，校验位为“0”。而0校验，即校验位总是为“0”，1校验校验位总是为“1”。奇偶校验逻辑相反，01校验逻辑相反。一般是奇偶校验或者是无校验位。

#### 奇偶校验的Verilog实现

在Verilog中奇偶校验的计算非常简单，根据奇偶校验的原理，偶校验为数据位各位异或，奇校验是偶校验取反，通过使用单目运算符的缩减功能，可以非常简单的计算奇偶校验位：

	input [7:0] data_in,       //需要发送的8位数据
	
	wire even_bit;  //偶校验位 = 各位异或
	wire odd_bit;   //奇校验位 = ~偶校验位
	
	assign even_bit = ^data_in; //一元约简运算符，等效于data_in[0] ^ data_in[1] ^ .....
	assign odd_bit = ~even_bit;	
	
	wire POLARITY_BIT = even_bit;  //偶校验

#### 关于波特率允许的误差

经过我的实际测试，波特率是有一定的容错范围的，例如，STM32配置成115200波特率，每10ms发送一个30字节的字符串，串口芯片用的CH340，上位机波特率设置成113000-121000也可以接收，无乱码，差不多正负2000的波特率，这容错范围也太大了，当然如果发送频率太快，数据量太大，误码率肯定会大大增加，所以还是建议通信双方使用同样的波特率以减少误差。

#### 串口数据的实际波形

使用串口上位机连接USB-TTL模块，发送一个字节数据：1位停止位+8位数据位+1位奇校验位+1位停止位，使用示波器的单次触发功能，可以在USB-TTL模块的TX引脚测得串口协议数据的实际波形，你知道这发送的是什么字符吗？

一个字符的实际波形

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/UART_Verilog/%E4%B8%B2%E5%8F%A3%E5%8F%91%E9%80%81%E5%AD%97%E7%AC%A66.JPG)

两个字符的实际波形

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/UART_Verilog/%E4%B8%B2%E5%8F%A3%E5%8F%91%E9%80%81%E5%AD%97%E7%AC%A66%E5%92%8C2.JPG)


### 单工、半双工、全双工、异步和同步的区别

在介绍串口的电平标准之前，先来了解一下串行通信的工作方式，即单工、半双工、全双工，异步和同步的区别。

#### 单工

单工，即数据传输只在一个方向上传输，只能你给我发送或者我给你发送，方向是固定的，不能实现双向通信，如：室外天线电视、调频广播等。

#### 半双工

半双工比单工先进一点，传输方向可以切换，允许数据在两个方向上传输，但是某个时刻，只允许数据在一个方向上传输，可以基本双向通信，如：对讲机，IIC通信。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/UART_Verilog/%E5%8D%95%E5%8F%8C%E5%B7%A5.gif)

#### 全双工

比半双工更先进的是全双工，允许数据同时在两个方向传输。发送和接收完全独立，在发送的同时可以接收信号，或者在接收的同时可以发送。它要求发送和接收设备都要有独立的发送和接收能力，如：电话通信，SPI通信，串口通信。

#### 同步和异步的区别

串行通信可以分为两种类型，一种叫同步通信，另一种叫异步通信。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/UART_Verilog/%E5%BC%82%E6%AD%A5%E4%BF%A1%E5%8F%B7.gif)

简单的说，就是同步通信需要时钟信号，而异步通信不需要时钟信号。

- 同步：发送方发出数据后，等接收方发回响应以后才发下一个数据包的通讯方式。  
- 异步：发送方发出数据后，不等接收方发回响应，接着发送下个数据包的通讯方式。

SPI和IIC为同步通信，UART为异步通信，而USART为同步&异步通信。

- USART：通用同步和异步收发器
- UART：通用异步收发器

即USART支持同步和异步收发，而UART只支持异步收发。

如STM32的串口工作在同步模式时，即智能卡模式时，就需要连接同步时钟引脚。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/UART_Verilog/%E5%90%8C%E6%AD%A5%E4%BF%A1%E5%8F%B7.gif)

### 常用的串行通信协议/电平标准

#### TTL电平

即普通MCU芯片输出的串口电平，如各MCU输出的串口信号就是TTL电平。低电平为0-GND，高电平为1-VCC，标准的数字电路逻辑。特点是速度快，延迟低，但是功耗大。基本上用于板内两个芯片之间短距离通信。

#### RS232

RS232是工业上常用的串口标准，无论是PLC的232接口，还是工控机上的串口，输出的串口电平都是232电平标准，232标准采用负逻辑电平，即-15~-3v为逻辑1，+3~+15为逻辑0，这里的电平是指RX和TX相对于GND的电压，可见无论在电压范围还是电压极性上都和TTL不同，显然这两种电平不能直接连接，需要使用MAX232类似的电平转换芯片，对两种电平进行互相转换，全双工，传输距离一般控制在20m以内，原因是RS-232属单端信号传送，存在共地噪声和不能抑制共模干扰等问题。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/UART_Verilog/%E4%B8%B2%E5%8F%A3DB9%E6%8E%A5%E5%A4%B4%E5%AE%9A%E4%B9%89.jpg)

#### RS485

在要求通信距离为几十米到上千米时，广泛采用RS-485 串行总线标准。RS-485采用平衡发送和差分接收，因此具有抑制共模干扰的能力。加上总线收发器具有高灵敏度，能检测低至200mV的电压，故传输信号能在千米以外得到恢复。 RS-485采用半双工工作方式，任何时候只能有一点处于发送状态，因此，发送电路须由使能信号加以控制。RS-485用于多点互连时非常方便，可以省掉许多信号线。应用RS-485 可以联网构成分布式系统，其允许最多并联32台驱动器和32台接收器。

#### RS422

RS-422和RS-485电路原理基本相同，都是以差分方式发送和接受，不需要数字地线。RS-422通过两对双绞线可以全双工工作收发互不影响，而RS485只能半双工工作，发收不能同时进行，但它只需要一对双绞线。RS422和RS485在19kpbs下能传输1200米。RS-422的电气性能与RS-485完全一样。主要的区别在于：RS-422有4根信号线：两根发送（Y、Z）、两根接收（A、B）。由于RS-422的收与发是分开的所以可以同时收和发（全双工）。

### 串行通信的编码方式
 
#### RZ编码

RZ编码也成为归零码，归零码的特性就是在一个周期内，用二进制传输数据位，在数据位脉冲结束后，需要维持一段时间的低电平。如图：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/UART_Verilog/%E5%BD%92%E9%9B%B6%E7%A0%81.jpg)


上图表示的是单极性归零码，即低电平表示0，正电平表示1。对于双极性归零码来说，则是高电平表示1，负电平表示0。如下图所示：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/UART_Verilog/%E5%8F%8C%E6%9E%81%E6%80%A7%E5%BD%92%E9%9B%B6%E7%A0%81.jpg)

#### NRZ编码

NRZ编码也成为不归零编码，也是我们最常见的一种编码，即正电平表示1，低电平表示0。它与RZ码的区别就是它不用归零，也就是说，一个周期可以全部用来传输数据，这样传输的带宽就可以完全利用。

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/UART_Verilog/%E4%B8%8D%E5%BD%92%E9%9B%B6%E7%A0%81.jpg)

#### NRZI编码

NRZI编码的全称为反向不归零编码，这种编码方式集成了前两种编码的优点，即既能传输时钟信号，又能尽量不损失系统带宽。对于USB2.0通信的编码方式就是NRZI编码。其实NRZI编码方式非常的简单，即信号电平翻转表示0，信号电平不变表示1。例如想要表示00100010(B)，则信号波形如下图所示：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/UART_Verilog/%E5%8F%8D%E5%90%91%E4%B8%8D%E5%BD%92%E9%9B%B6%E7%A0%81.jpg)

例如有一段数据为：1111 1111 (B)要发送，则整个传输线上的电平状态是这样的：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/UART_Verilog/%E5%8F%8D%E5%90%91%E4%B8%8D%E5%BD%92%E9%9B%B6%E7%A0%81%E4%BC%A0%E8%BE%9311111.jpg)

#### Manchester编码

曼彻斯特编码，又称数字双向码、分相码或相位编码(PE)，是一种常用的的二元码线路编码方式。常用在以太网通信，列车总线控制，工业总线等领域。在曼彻斯特编码中，每一位的中间有一跳变，位中间的跳变既作时钟信号，又作数据信号；从高到低跳变表示“0”，从低到高跳变表示“1”。其中非常值得注意的是，在每一位的"中间"必有一跳变，根据此规则，可以得出曼彻斯特编码波形图的画法。例如：传输二进制信息0，若将0看作一位，我们以0为中心，在两边用虚线界定这一位的范围，然后在这一位的中间画出一个电平由高到低的跳变。后面的每一位以此类推即可画出整个波形图。举个图例吧，若要表示数据1001 1010(B)，则信号波形图如下图所示：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/UART_Verilog/%E6%9B%BC%E5%BD%BB%E6%96%AF%E7%89%B9%E7%BC%96%E7%A0%81.jpg)

曼彻斯特编码方式也如前面所说，虽然传输了时钟信号，但也损失了一部分的带宽，主要表现在相邻相同数据上。但对于高速数据来说，这种编码方式无疑是这几种编码方式中最优的，相比NRZI编码，曼彻斯特编码不存在长时间信号状态不变导致的时钟信号丢失的情况，所以在这种编码方式在以太网通信中是十分常用的。

### 串行和并行哪个速度快？

串口，即串行通信接口，与之对应的是并行接口。在实际时钟频率比较低的情况下，并口因为可以同时传输若干比特，速率确实比串口快。但是，随着技术的发展，时钟频率越来越高，当时钟频率提高到一定的程度时，并行接口因为有多条并行且紧密的导线，导线之间的相互干扰越来越严重。而串口因为导线少，线间干扰容易控制，况且加上差分信号的加持，抗干扰性能大大提升，因此可以通过不断提高时钟频率来提高传输速率，这就是为什么现在高速传输都采用串行方式的原因。例如常见的USB、SATA、PCIe、以太网等。

如果有人问关于串行传输与并行传输谁更好的问题，你也许会脱口而出：串行通信好！但是，串行传输之所以走红，是由于将单端信号传输转变为差分信号传输，并提升了控制器工作频率的原因，而“在相同频率下并行通信速度更高”这个基本道理是永远不会错的，通过增加位宽来提高数据传输率的并行策略仍将发挥重要作用。当然，前提是有更好的措施来解决并行传输过程中的种种问题。

### 标准串口协议的Verilog实现

及FPGA实现，基于Verilog实现标准串口协议发送8位数据：起始位 + 8位数据位 + 校验位 + 停止位 = 11位，每1位的时间是16个时钟周期，所以输入时钟应该为：波特率*16，带Busy忙信号输出。实现方法比较简单，数据帧的拼接、计数器计时钟周期，每16个时钟周期输出一位数据即可。

#### 串口发送1个字节实现



	/*
	串口协议发送：起始位 + 8位数据位 + 校验位 + 停止位 = 11位 * 16 = 176个时钟周期
	clk频率 = 波特率 * 16
	*/
	
	module uart_tx_8bit(
	
	//input
	input clk,                //UART时钟=16*波特率
	input rst_n,
	input [7:0] data_in,       //需要发送的数据
	input trig,                //上升沿发送数据
	
	//output
	output busy,                 //高电平忙:数据正在发送中
	output reg tx                //发送数据信号
	
	);
	
	reg[7:0] cnt;             //计数器
	reg trig_buf;
	reg trig_posedge_flag;
	// reg trig_negedge_flag;
	reg send;
	
	reg [10:0] data_in_buf;     //trig上升沿读取输入的字节，拼接数据帧
	
	wire odd_bit;   //奇校验位 = ~偶校验位
	wire even_bit;  //偶校验位 = 各位异或
	wire POLARITY_BIT = even_bit;  //偶校验
	// wire POLARITY_BIT = odd_bit;   //奇校验
	
	assign even_bit = ^data_in; //一元约简，= data_in[0] ^ data_in[1] ^ .....
	assign odd_bit = ~even_bit;
	assign busy = send;     //输出的忙信号
	
	//起始位+8位数据位+校验位+停止位 = 11位 * 16 = 176个时钟周期
	parameter CNT_MAX = 176;
	
	always @(posedge clk)
	begin
	    if(!rst_n)
	    begin
	        trig_buf <= 0;
	        trig_posedge_flag <= 0;
	        // trig_negedge_flag <= 0;
	    end
	    else 
	    begin
	        trig_buf <= trig;
	        trig_posedge_flag <= (~trig_buf) & trig; //在trig信号上升沿时产生1个时钟周期的高电平
	        // trig_negedge_flag <= trig_buf & (~trig); //在trig信号下降沿时产生1个时钟周期的高电平    
	    end
	end
	
	always @(posedge clk)
	begin
	    if(!rst_n)
	        send <= 0;
	    else if (trig_posedge_flag &  (~busy))  //当发送命令有效且线路为空闲时，启动新的数据发送进程
	        send <= 1;
	    else if(cnt == CNT_MAX)      //一帧资料发送结束
	        send <= 0;
	end
	
	always @ (posedge clk)
	begin
	    if(!rst_n)
	        data_in_buf <= 11'b0;
	    else if(trig_posedge_flag & (~busy))    //只读取一次数据，一帧数据发送过程中，改变输入无效
	        data_in_buf <= {1'b1, POLARITY_BIT, data_in[7:0], 1'b0};   //数据帧拼接
	end
	
	always @ (posedge clk)
	begin
	    if(!rst_n)
	        cnt <= 0;
	    else if(!send || cnt >= CNT_MAX)
	        cnt <= 0;
	    else if(send)
	        cnt <= cnt + 1;
	end
	
	always @(posedge clk)
	begin
	    if(!rst_n)
	        tx <= 1;
	    else if(send)  
	    begin
	        case(cnt)                 //1位占用16个时钟周期
	            0: tx <= data_in_buf[0];           //低位在前，高位在后
	            16: tx <= data_in_buf[1];    //bit0,占用第16~31个时钟
	            32: tx <= data_in_buf[2];    //bit1,占用第47~32个时钟
	            48: tx <= data_in_buf[3];    //bit2,占用第63~48个时钟
	            64: tx <= data_in_buf[4];    //bit3,占用第79~64个时钟
	            80: tx <= data_in_buf[5];    //bit4,占用第95~80个时钟
	            96: tx <= data_in_buf[6];    //bit5,占用第111~96个时钟
	            112: tx <= data_in_buf[7];   //bit6,占用第127~112个时钟
	            128: tx <= data_in_buf[8];   //bit7,占用第143~128个时钟
	            144: tx <= data_in_buf[9];   //发送奇偶校验位，占用第159~144个时钟
	            160: tx <= data_in_buf[10];  //发送停止位，占用第160~167个时钟            
	            CNT_MAX: tx <= 1;            //无空闲位
	            default:;
	        endcase
	    end
	    else if(!send) 
	        tx <= 1;
	end
	
	endmodule


仿真波形

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/UART_Verilog/%E4%B8%B2%E5%8F%A3%E5%8F%91%E9%80%811%E4%B8%AA%E5%AD%97%E8%8A%82%E4%BB%BF%E7%9C%9F%E5%9B%BE.jpg)

#### 串口接收1个字节实现

串口接收部分的实现，涉及到串口数据的采样，对于MCU来说，不同单片机集成外设的处理方式有所不同，具体采样原理可以参考内核的Reference Manual。以传统51内核为例，按照所设置的波特率，每个位时间被分为16个时间片。UART接收器会在第7、8、9三个时间片进行采样，按照三取二的逻辑获得该位时间内的采样结果。其它一些类型的单片机则可能会更加严苛，例如有些工业单片机会五取三甚至七取五（设置成抗干扰模式时）。

本程序中采用的中间值采样，即取16个时钟周期中的中间位作为当前的采样值。


	//Verilog实现串口协议接收，带错误指示，校验错误和停止位错误
	
	/*
	16个时钟周期接收1位，中间采样
	*/
	module my_uart_rx(
	
	input clk,             //采样时钟
	input rst_n,
	input rx,              //UART数据输入
	output reg [7:0] dataout,        //接收数据输出
	output reg rx_ok,          //接收数据有效，高说明接收到一个字节
	output reg err_check,      //数据出错指示
	output reg err_frame     //帧出错指示
	
	);
	
	reg [7:0] cnt;
	reg [10:0] dataout_buf;
	
	reg rx_buf;
	reg rx_negedge_flag;
	reg receive;
	
	wire busy;
	wire odd_bit;   //奇校验位 = ~偶校验位
	wire even_bit;  //偶校验位 = 各位异或
	wire POLARITY_BIT;   //本地计算的奇偶校验
	// wire polarity_ok;
	// assign polarity_ok = (POLARITY_BIT == dataout_buf[9]) ? 1 : 0; //校验正确=1，否则=0
	
	assign busy = rx_ok;
	assign even_bit = ^dataout;     //一元约简，= data_in[0] ^ data_in[1] ^ .....
	assign odd_bit = ~even_bit;
	assign POLARITY_BIT = even_bit;  //偶校验
	// assign POLARITY_BIT = odd_bit;  //奇校验
	
	parameter CNT_MAX = 176;
	
	//rx信号下降沿标志位
	always @(posedge clk)   
	begin
	    if(!rst_n)
	    begin
	        rx_buf <= 0;
	        rx_negedge_flag <= 0;
	    end
	    else
	    begin
	        rx_buf <= rx;
	        rx_negedge_flag <= rx_buf & (~rx);
	    end
	end
	//在接收期间，保持高电平
	always @(posedge clk)
	begin
	    if(!rst_n)
	        receive <= 0;
	    else if (rx_negedge_flag && (~busy))  //检测到线路的下降沿并且原先线路为空闲，启动接收数据进程
	        receive <= 1;      //开始接收数据
	    else if(cnt == CNT_MAX)  //接收数据完成
	        receive <= 0;
	end
	//起始位+8位数据位+校验位+停止位 = 11位 * 16 = 176个时钟周期
	always @ (posedge clk)
	begin
	    if(!rst_n)
	        cnt <= 0;
	    else if(!receive || cnt >= CNT_MAX)
	        cnt <= 0;
	    else if(receive)
	        cnt <= cnt + 1;
	end
	//校验错误:奇偶校验不一致
	always @ (posedge clk)
	begin
	    if(!rst_n)
	        err_check <= 0;
	    else if(cnt == 152)
	    begin
	        // if(POLARITY_BIT == rx)
	        if(POLARITY_BIT != dataout_buf[9])      //奇偶校验正确
	            err_check <= 1;         //锁存
	        // else
	            // err_check <= 1;       
	    end
	end
	//帧错误:停止位不为1
	always @ (posedge clk)
	begin
	    if(!rst_n)
	        err_frame <= 0;
	    else if(cnt == CNT_MAX)
	    begin
	        if(dataout_buf[10] != 1)        //停止位
	            err_frame <= 1;
	        // else
	            // err_frame <= 1;      //如果没有接收到停止位，表示帧出错
	    end
	end
	
	always @ (posedge clk)
	begin
	    if(!rst_n)
	        dataout <= 11'h00;
	    else if(receive)
	    begin
	        // if(rx_ok)
	        if(cnt >= 137)
	            dataout <= dataout_buf[8:1];        //数据位:8-1位
	        // else if(!rx_ok)
	            // dataout <= 0;
	    end
	end
	
	always @ (posedge clk)
	begin
	    if(!rst_n)
	        rx_ok <= 0;
	    else if(receive)
	    begin
	        if(cnt >= 137)   //137-169
	            rx_ok <= 1;
	        else 
	            rx_ok <= 0;
	    end
	    else 
	        rx_ok <= 0;
	end
	
	
	//起始位+8位数据+奇偶校验位+停止位 = 11 * 16 = 176位
	
	always @(posedge clk)
	begin
	    if(!rst_n)
	        dataout_buf <= 8'h00;
	    else if(receive)
	    begin
	        case (cnt)      //中间采样
	            8'd8: dataout_buf[0] <= rx;         //起始位=0
	            8'd24: dataout_buf[1] <= rx;        //LSB低位在前
	            8'd40: dataout_buf[2] <= rx;
	            8'd56: dataout_buf[3] <= rx;
	            8'd72: dataout_buf[4] <= rx;
	            8'd88: dataout_buf[5] <= rx;
	            8'd104: dataout_buf[6] <= rx;
	            8'd120: dataout_buf[7] <= rx;
	            8'd136: dataout_buf[8] <= rx;       //MSB高位在后
	            8'd152: dataout_buf[9] <= rx;       //奇偶校验位
	            8'd168: dataout_buf[10] <= rx;      //停止位=1
	            default:;
	        endcase
	    end
	end

	endmodule
	

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/UART_Verilog/%E4%B8%B2%E5%8F%A3%E6%8E%A5%E6%94%B6%E4%BB%BF%E7%9C%9F%E6%B3%A2%E5%BD%A2.jpg)

### 代码工程下载

- Github工程地址：[https://github.com/whik/UART_Demo_Verilog](https://github.com/whik/UART_Demo_Verilog)
- Gitee工程地址：[https://gitee.com/whik/UART_Demo_Verilog](https://gitee.com/whik/UART_Demo_Verilog)

工程包含：

- my_uart_rx：串口接收1个字节示例程序
- uart_tx_8bit：串口发送1个字节示例程序
- uart_tx_demo：串口每隔500ms循环发送0-9字符

### 参考资料：

- [百度百科_TTL电平](https://baike.baidu.com/item/TTL%E7%94%B5%E5%B9%B3/5904345#1)
- [一文读懂RS-232与RS-422及RS-485三者之间的特性与区别](http://m.elecfans.com/article/663969.html)
- [工作中经常遇到的232、485、TTL信号](https://blog.csdn.net/fxltsbl007/article/details/86539901)

### 推荐阅读：

- [玄铁910是个啥？是芯片吗？](http://www.wangchaochao.top/2019/07/28/XuanTie-Core/)
- [Qt平台下使用QJson解析和构建JSON字符串](http://www.wangchaochao.top/2019/07/23/QJson-Demo/)
- [国产处理器的逆袭机会——RISC-V](http://www.wangchaochao.top/2019/04/27/ESBF/)
- [真正的RISC-V开发板——VEGA织女星开发板开箱评测](http://www.wangchaochao.top/2019/06/22/VEGA-4/)
- [【2019北京国际消费电子博览会】参观总结](http://www.wangchaochao.top/2019/06/30/Beijing-CEE/)
- [Qt实现软件自动更新的一种简单方法](http://www.wangchaochao.top/2019/03/31/Qt-Update/)

--------

我的博客：[www.wangchaochao.top](www.wangchaochao.top)

或微信扫码关注我的公众号
