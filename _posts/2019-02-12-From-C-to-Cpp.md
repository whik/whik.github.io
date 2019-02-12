---
layout:     post
title:    C++学习之从C到C++
subtitle:	 C语言和C++的不同
date:       2019-02-12 21:30:40 +0800
author:     Wang Chao
header-img: img/post-bg-c-language-review.jpg
catalog:    true
tag:
    - C语言
    - C++
---

### 头文件的包含

C++的头文件

包含头文件可以不加.h结尾，如iostream，一些常用的头文件在引用时可以不加.h后缀，并在开头增加c，如：

```C
	#include <cstdio>
	#include <cstring>
	#include <cstdlib>
```

### 强制类型转换

在C中的强制类型转换为：(int )3.5；

而在C++中的强制类型转换为int(3.5)，更加清晰直观。

### 默认参数

在 C++ 中，声明一个函数时，可以指定默认的输入参数值。当调用有默认参数值的函数时，可以不写出参数，这时就相当于以默认值作为参数调用该函数。

例如：

	void Function1(int x=20); 	 //函数的声明中，指明参数 x 的默认值是 20
	Function1(); 				 	//正确的调用语句，等效于 Function1(20);

不仅可以用常数，还可以用任何有定义的表达式作为参数的默认值。例如：

	int Max(int m, int n);
	int a, b;
	void Function2(int x, int y=Max(a,b), int z=a*b)
	{
	    //...
	}
	
	Function2(4);  //正确，等效于 Function2(4, Max(a,b), a*b);
	Function2(4, 9);  //正确，等效于 Function2(4,9 , a*b);
	Function2(4, 2, 3);  //正确
	Function2(4, ,3);  //错误！这样的写法不允许，省略的参数一定是最右边连续的几个

C++中增加函数参数的默认值，可以像上面的 Function1 那样写在声明函数的地方，也可以像 Function2 那样写在定义函数的地方，但是不能在两个地方都写。

函数默认参数所带来的好处是使程序的可扩充性更好，即当程序需要增加新功能时，改动可以尽可能少。

试想下面这种情况。一个即将编写完成的绘图程序，其中有一个画圆的函数 Circle，画出来的圆都是黑色的，这时希望增加画彩色圆的功能，于是就需要在 Circle 函数中增加一个 int 型的 color 参数，用来表示颜色。

但是原来的程序中可能大多数调用 Circle 函数的地方依然只是画个黑色的圆就可以了，只有少数几个地方需要改成画彩色的圆。此时，如果要找出所有调用 Circle 函数的语句并补上颜色参数，会十分烦琐。

而有了函数参数默认值的机制，则只需为 Circle 函数的新参数指定默认值 0（假定 0 代表黑色），然后找出少数几个调用 Circle 函数画彩色圆的地方，补上颜色参数即可。

实践中这种情况 是经常发生的。

### 引用

在 C++ 中可以定义“引用”。定义方式如下：
类型名 &引用名 = 同类型的某变量名;

此种写法就定义了一个某种类型的引用，并将其初始化为引用某个同类型的变量。“引用名”的命名规则和普通变量相同。例如：

	int n;
	int & r = n;

r 就是一个引用，也可以说 r 的类型是 int &。第二条语句使得 r 引用了变量 n，也可以说 r 成为了 n 的引用。

某个变量的引用和这个变量是一回事，相当于该变量的一个别名。


也可以用一个引用去初始化另一个引用，这样两个引用就引用同一个变量。不能用常量初始化引用，也不能用表达式初始化引用（除非该表达式的返回值是某个变量的引用）。

总之，引用只能引用变量。

类型为 T& 的引用和类型为 T 的变量是完全兼容的，可以互相赋值。

引用的示例程序如下：
	
	#include <iostream>
	using namespace std;
	int main()
	{
	    int n = 4;
	    int & r = n;     		 	 //r引用了n，从此r和n是一回事
	    r = 4;             			 //修改r就是修改n
	    cout << r << endl;  	//输出4
	    cout << n << endl;  	//输出4
	    n = 5;              			//修改n就是修改r
	    cout << r << endl;  	//输出 5
	    int & r2 = r;         		//r2和r引用同一个变量,就是n
	    cout << r2 << endl; 	//输出 5
	    return 0;
	}

- 注意：定义引用时一定要将其初始化，否则编译无法通过。通常会用某个变量去初始化引用，初始化后，它就一直引用该变量，不会再引用别的变量。


### 引用作为函数的返回值

函数的返回值可以是引用。例如下面的程序：

	#include <iostream>
	using namespace std;
	int n = 4;
	int & SetValue()
	{
	    return n;  //返回对n的引用
	}
	int main()
	{
	    SetValue() = 40;  //返回值是引用的函数调用表达式，可以作为左值使用
	    cout << n << endl;  //输出40
	    int & r = SetValue();
	    cout << r << endl;  //输出40
	    return 0;
	}

SetValue 函数的返回值是一个引用，是 int & 类型的。因此第 6 行使得其返回值成为变量 n 的引用。

第 10 行，SetValue 函数返回对 n 的引用，因此对 SetValue 函数的返回值进行赋值，就是对 n 进行赋值，结果就是使得 n 的值变为 40。

第 12 行，表达式 SetValue 函数的返回值是 n 的引用，因此可以用来初始化 r，其结果就 是 r 也成为 n 的引用。

### 总结

以上只是C和C++不同的几个点，其实还有很多不同的地方，如内联函数、函数重载、动态分配内存和字符串类型等等。C是面向过程的，而C++是面向对象的，面向对象的程序设计有以下4个特点：

- 抽象
- 封装
- 继承
- 多态

总之，面向对象的程序设计方法继承了结构化程序设计方法的优点，同时又比较有效地克服了结构化程序设计的弱点。

----

Jlink使用技巧系列文章：

- [Jlink使用技巧之合并烧写文件](http://www.wangchaochao.top/2019/01/17/Jlink-merge/)
- [Jlink使用技巧之烧写SPI Flash存储](http://www.wangchaochao.top/2019/01/12/Jlink-SPI-Flash/)
- [Jlink使用技巧之虚拟串口功能](http://www.wangchaochao.top/2019/01/09/Jlink-UART/)
- [Jlink使用技巧之读取STM32内部的程序](http://www.wangchaochao.top/2019/01/06/Jlink-ReadBack-Hex/)
- [Jlink使用技巧之J-Scope虚拟示波器功能](http://www.wangchaochao.top/2018/10/17/JScope/)
- [Jlink使用技巧之单独下载HEX文件到单片机](http://www.wangchaochao.top/2019/01/05/Jlink-Download-Hex/)

----

欢迎大家关注我的[个人博客](http://www.wangchaochao.top)

或微信扫码关注我的公众号







