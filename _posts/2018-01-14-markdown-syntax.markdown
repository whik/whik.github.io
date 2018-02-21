---
layout:     post
title:      Markdown语法简介
subtitle:   Markdown
date:       2018-01-14
author:     WangChao
header-img: img/post-bg-markdown.jpg
catalog: true
tags:
    - 博客
    - Markdown
    - 程序语言
---

## 一、标题



## 二、列表
### 无序列表
* 无序列表1使用*
- 无序列表2使用-
+ 无序列表3使用+
* 效果一样

*   Lorem ipsum dolor sit amet, consectetuer adipiscing elit.
	Aliquam hendrerit mi posuere lectus. Vestibulum enim wisi,
	viverra nec, fringilla in, laoreet vitae, risus.
*   Donec sit amet nisl. Aliquam semper ipsum sit amet velit.
    Suspendisse id sem consectetuer libero luctus adipiscing.
### 有序列表

1. 列表1段落1


    列表2段落2，一个tab键，或者4个空格
2. 列表2
3. 列表3
	>列表内也可以使用引用语法
	>引用2
	>引用3
	>>引用嵌套
	>####标题语法
			void main()代码需要2个tab键，或者8个空格
			    return 0;
1986. What a great season.如何正确显示1986.呢？在.号之前加一个\
1986\.What a great season.


## 三、引用


### 段落引用

> 引用文字1
> 引用文字2
> 引用文字3

>>引用嵌套
>引用内也可以使用其他语法
># 如标题语法
>1.  有序列表
>2.  列表2
>3.  return shell_exec("echo $input | $markdown_script");
>>>引用嵌套


## 四、强调 ##

使用*一个星号*这样是一个斜体强调

使用**粗体**来强调内容2个星号

使用_下划线_同样是斜体强调		

使用__两个下划线__也是粗体强调


## 五、链接

### 如：这是[百度](http://www.baidu.com/ "Baidu")的网站.
### 这是[我的博客][4]链接.

或者像这样多次使用时，可以这样
I get 10 times more traffic from [Google][1] than from
[Yahoo][2] or [MSN][3].

[WangChao Blog][4]

也可以这样[百度的网站][baidu]

属性是选择性的，可以是字母，数字，不区分大小写

[1]: http://google.com  "Google"
[2]: http://search.yahoo.com "Yahoo Search"
[3]: http://search.msn.com/ "MSN Search"
[4]: http://whik.gitee.io/ "My Blog"
[baidu]: http://baidu.com "Baidu"

## 六、图片

图片的形式和链接的形式差不多
<br>
插入网络上图片

![图片1](http://img1.imgtn.bdimg.com/it/u=4131634322,487666839&fm=27&gp=0.jpg "图片1")


插入仓库内的图片

<figure>
<a><img src="{{site.url}}/img/post-bg-markdown.jpg"></a>
</figure>


## 七、代码

1个tab键即可

	#include "stdio.h"
	void main()
	{
		printf("Hello World");
	}


```c

int main()
{
    MPU6050_Angle data;
    
    MPU6050_Init();
    Usrat_1_Init(84,9600,0);    
    
    while (1)
    {
        MPU6050_Get_Angle(&data);  // 计算三轴倾角
        
        printf("X_Angle = %lf°  ", data.X_Angle);
        printf("Y_Angle = %lf°  ", data.Y_Angle);
        printf("Z_Angle = %lf°  ", data.Z_Angle);
        printf("\r\n");
        
        Delay(0xfffff);
    }
}
```





