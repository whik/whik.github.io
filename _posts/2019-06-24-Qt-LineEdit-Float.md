---
layout:     post
title:    QLineEdit限制数据类型——只能输入浮点型数
subtitle:	 Qt学习
date:       2019-06-24 17:55:40 +0800
author:     Wang Chao
header-img: img/qt.jpg
catalog:    true
tag:
    - Qt
    - C++
    - C语言
---

### 前言

最近做了一个小的上位机，要通过串口来下发几个时间参数，为了防止误输入，产生不必要的麻烦，我把输入范围限制在0-680的浮点型数据，支持小数点后2位。学习了一下QLineEdit类是如何限制输入类型的。本来是想写一个函数，在下发参数时，传QLineEdit的字符串参数进去，然后判断是否合法，如果不合法，则不下发参数，请用户修改后再确认。这么做也实现了，但是想Qt这么强大，应该会考虑到这一点的，所以找了个更简单，在输入的时候就限制数据的类型，不合法的根本输入不进去。

### 关于QLineEdit类

QlineEdit是一个单行文本输入框，支持撤销、重做、复制、粘贴、拖放等操作，echomode模式支持，即只写模式，可以输入密码等不可见的文本，官方介绍：[QLineEdit Class ](https://doc.qt.io/qt-5/qlineedit.html)

可以通过setValidator函数来限制数据类型，

setValidator函数的参数是QValidator，主要有3种：

- QIntValidator             //限制只能输入整数，限制范围
- QDoubleValidator     //限制只能输入浮点数，包括范围，小数点位数
- QRegExpValidator    //限制规则按指定的正则表达式

**Amazing！**QDoubleValidator不就是我想要的吗？但是经过实际测试发现，其中QDoubleValidator可以限制浮点型数据和输入的小数位数，但是并**不能限制输入范围**，也就是setRange，setBottom，setTop这些函数的设置并没有生效，这难道是Qt的一个Bug？我的Qt版本是5.8.0，Qt Creator版本是4.2.1，而QRegExpValidator的使用就很强大了，需要了解正则表达式的相关知识。下面来详细介绍一下这三种类的使用。

### QIntValidator Class

- **功能**

  限制QLineEdit只能输入int类型数据，即整型数据，包含正负整数和0

- **相关函数**

```c++

  //限制数据范围
  QIntValidator(int minimum, int maximum, QObject *parent = Q_NULLPTR)
  //获取最小值
  int bottom() 
  //设置最小值
  void setBottom(int)
  //设置数据范围
  void setRange(int bottom, int top)
  //设置最大值
  void setTop(int)
  //获取最大值
  int top() const

```

- **示例**


```c++

//整型限制范围100-999
lineEdit->setValidator(new QIntValidator(100, 999, this));       

//或者
QIntValidator* aIntValidator = new QIntValidator;
aIntValidator->setRange(100, 999);
ui->le_L1->setValidator(aIntValidator);

```

### QDoubleValidator Class

- **功能**

  限制QLineEdit只能输入浮点型数据，可以指定输入范围及小数点位数

- **相关函数**


```c++

//限制数据范围
QDoubleValidator(double bottom, double top, int decimals, QObject *parent = Q_NULLPTR)
//设置小数点位数
void setDecimals(int)
//获取设置的小数点位数
int decimals() 
//设置数字表示方式，标准计数法还是科学计数法
void setNotation(Notation)
//获取设置的计数方式
Notation notation() 
//设置最小值
void setBottom(double)
//获取设置的最小值
double bottom() 
//设置最大值
void setTop(double)
//获取设置的最大值
double top() 
//设置数据范围，默认无小数位
void setRange(double minimum, double maximum, int decimals = 0)

```

- **示例**


```c++

//限制范围0-680，小数点2位
lineEdit->setValidator(**new** QDoubleValidator(0,680,2,**this**));

```

*限制范围无效，这可能是Qt的一个Bug。*

### QRegExpValidator Class

- **功能**

  按照自定义的正则表达式规则，限制输入的范围。

- **相关函数**


```c++

//设置按正则表达式限制
QRegExpValidator(const QRegExp &rx, QObject *parent = Q_NULLPTR)
//获取设置的正则表达式
QRegExp &regExp() 
//设置正则表达式
void setRegExp(const QRegExp &rx)

```

- **示例**

```c++

//限制-180，180，并限定小数点后4位
QRegExp rx("^-?(180|1?[0-7]?\\d(\\.\\d{1,4})?)$");  
QRegExpValidator *pReg = new QRegExpValidator(rx, this);  
lineEdit->setValidator(pReg);  

```

### 关于正则表达式

> **正则表达式**，又称规则表达式**。**（英语：Regular Expression，在代码中常简写为regex、regexp或RE），计算机科学的一个概念。正则表达式通常被用来检索、替换那些符合某个模式(规则)的文本。许多程序设计语言都支持利用正则表达式进行字符串操作。例如，在[Perl](https://baike.baidu.com/item/Perl)中就内建了一个功能强大的正则表达式引擎。正则表达式这个概念最初是由[Unix](https://baike.baidu.com/item/Unix)中的工具软件（例如sed和[grep](https://baike.baidu.com/item/grep/5997841)）普及开的。正则表达式通常缩写成“regex”，[单数](https://baike.baidu.com/item/%E5%8D%95%E6%95%B0/1658633)有regexp、regex，[复数](https://baike.baidu.com/item/%E5%A4%8D%E6%95%B0/254365)有regexps、regexes、regexen。

关于正则表达式的详细介绍：[正则表达式30分钟入门教程](http://deerchao.net/tutorials/regex/regex.htm)


```c++

//正则表达式说明：
/*

^(-?[0]|-?[1-9][0-9]{0,5})(?:\.\d{1,4})?$|(^\t?$)
(^-?180$)|(^-?1[0-7]\d$)|(^-?[1-9]\d$)|(^-?[1-9]$)|^0$
^-?(180|1?[0-7]?\d(\.\d+)?)$
^-?(180|1?[0-7]?\d(\.\d{1,4})?)$
^-?(90|[1-8]?\d(\.\d{1,4})?)$

 式子中开头的^和结尾的$限定字符串的开始和结尾；
 "-?" 表示一个或0个负号，这里面的问号表示其前面的字符重复0次或1次；
 管道符“|”表示平行分组，比如后三个，表示180或其它形式；
 [1-9] 表示限定数字范围为1到9，其余类似，如果是有限几个值，还可以用枚举的方式，比如限定-255到255时，第一个数字2的限定，应该表达为[1,2]，这表示这个位置只允许是1或者2；
 "\d"是一个转义字符，表示匹配一位数字；
 “\.” 表示匹配小数点；
 "\d+"，这里面的+表示其前面的\d重复一次或多次；
 "\d{1,4}"，里面的{1,4}表示重复1到4次；

*/

```


### 关于QDoubleValidator的Bug解决

网上搜索一遍，确实是Qt的Bug，需要重写，下面是一个网友实现的MyDoubleValidator类。

- **定义MyDoubleValidator类**


```c++

class MyDoubleValidator : public QDoubleValidator
{
    Q_OBJECT
public:
    MyDoubleValidator(QObject *parent);
    ~MyDoubleValidator();
    virtual QValidator::State validate(QString &input, int &pos) const;
};

```
- **函数实现**


```c++

#include "MyDoubleValidator.h"

MyDoubleValidator::MyDoubleValidator(QObject *parent)
	: QDoubleValidator(parent)
{
}

MyDoubleValidator::~MyDoubleValidator()
{
}

QValidator:: State MyDoubleValidator::validate(QString & input, int & pos) const
{
	if (input.isEmpty())
	{
		return QValidator::Intermediate;
	}
	bool OK = false;
	double val = input.toDouble(&OK);

	if (!OK)
	{
		return QValidator::Invalid;
	}
	
	int dotPos = input.indexOf(".");
	if (dotPos > 0)
	{
		if (input.right(input.length() - dotPos - 1).length() > decimals())
		{
			return QValidator::Invalid;
		}
	}
	if(val<bottom()|| val>top())
		return QValidator::Invalid;
	return QValidator::Acceptable;
}

```

- **实际应用**


```c++

{
	MyDoubleValidator * dv = new  MyDoubleValidator(0);
	dv->setNotation(QDoubleValidator::StandardNotation);
	dv->setRange(2.0, 3.0, 2);
	ui.lineEdit->setValidator(dv); 
}

```



### 自定义函数的实现方式

一开始，我并不知道可以通过setValidator函数来实现数据类型限制，我直接实现了一个检测输入的QString类型数据是否是Float数据，并没有指定小数后的位数，返回值为1表示是Float类型数据，否则不是。

- **函数实现**


```c++

int Dialog::FloatCheck(QString float_str)
{
    QByteArray ba = float_str.toLatin1();//QString 转换为 char*
    const char *str = ba.data();

    int dotNum = 0;
    int dotIdx = 0;
    int Idx = 0;
    while(*str)
    {
        Idx++;
        if(*str == '.')
        {
            dotIdx = Idx;   //dot
            dotNum++;   	//dot个数统计
            if(dotNum > 1)  //小数点个数超过1
                return 0;
            else if((dotNum == 0 && dotIdx) || (dotNum == 1 && dotIdx == 1))    //无小数点
            {
                return 1;
            }
        }
        if(*str != '.')
        {
            if(*str < '0' || *str > '9')
                return 0;
        }
        str++;
    }
    return 1;
}

```

- **测试验证**

	
	/*
	
	输入：
	    char *str1 = "1.2345";
	    char *str2 = "a.2345";
	    char *str3 = "0.2345";
	    char *str4 = "1a2345";
	    char *str5 = "a2345";
	    char *str6 = "1.2.2345";
	    char *str7 = "3.234.";
	    char *str8 = "3.234.a";
	
	输出：
	
	str1 : 1.2345 - 1
	str2 : a.2345 - 0
	str3 : 0.2345 - 1
	str4 : 1a2345 - 0
	str5 : a2345 - 0
	str6 : 1.2.2345 - 0
	str7 : 3.234. - 0
	str8 : 3.234.a - 0
	
	*/

### 历史精选

- [Qt实现软件自动更新的一种简单方法](http://www.wangchaochao.top/2019/03/31/Qt-Update/)
- [Qt小项目之串口助手控制LED](http://www.wangchaochao.top/2019/03/03/Qt-UART-Ctrl-LED/)
- [真正的RISC-V开发板——VEGA织女星开发板开箱评测](http://www.wangchaochao.top/2019/06/22/VEGA-4/)
- [手把手教你制作Jlink-OB调试器（含原理图、PCB、外壳、固件）](http://www.wangchaochao.top/2019/05/10/Open-JlinkOB/)

--------

欢迎关注我的[个人博客](http://www.wangchaochao.top)：`www.wangchaochao.top`

或微信扫码关注我的公众号



