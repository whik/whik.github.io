---
layout:     post
title:      配置SharpDevelop编译单个cs文件
subtitle:   文章搬运
date:       2016-04-06
author:     Shao Guoji
header-img: img/post-bg-sharpdevelop.jpg
catalog:    true
tag:
    - C#
---

> 这是我之前初学C#是在博客园2015-11-19的文章，现在搬到了自己博客

<br/>

### 废话

近日开始学习C#，看的是陈广老师的教程，视频中用的开发工具是SharpDevelop,工具没有编译单文件的功能，建立工程的话一大堆文件太麻烦，百度找了一下相关资料挺少的（或者是我找资料能力不行），然后发现了一篇好文章[如何编译单个cs文件](http://xloved.blog.163.com/blog/static/18571909420114854026152/)，尝试后效果确实不错，个人在此基础上进行了改进一点改进。

<br/>

---

### 第一步：创建bat批处理

用记事本建立一个consoleComplier.bat批处理文件，内容如下：

```bat

C:\Windows\Microsoft.NET\Framework\v4.0.30319\csc.exe /t:exe %1

%~n1

@pause

del %~n1.exe

```

**代码解释：**

　　第一行：到.NET所在目录下调用csc.exe编译器编译当前cs文件，路径参数由SD的"${ItemPath}"提供

　　第二行：执行编译后的exe文件

　　第三行：提示”按任意键继续“，等待用户输入

　　第四行：删除编译生成的exe文件，为下次重新编译新文件准备(为了不残留一堆难看的exe，也可以移至第二行之后，这样程序运行完就立即删除exe，避免手动关闭窗口时漏删)
。
<br/>

---

### 第二步：设置SharpDevelop

将Complier.bat随便找个地方保存（我是放在SharpDevelop的安装目录下），打开SharpDevelop，在菜单栏的**工具 -> 选项 -> 工具 -> 扩展工具**，按下图设置。

![set-sharpdevelop](http://odaps2f9v.bkt.clouddn.com/16-9-20/1411283.jpg)

<br/>

![choose bat file](http://odaps2f9v.bkt.clouddn.com/16-9-20/35224563.jpg)

<br/>

点击”添加“按钮添加一个工具，标题就叫“consoleComplier”或“Complie And Run”什么的自己起吧，然后注意的是命令一栏是刚刚Complier.bat所在的路径，**参数："${ItemPath}"（包括双引号），工作目录：${ItemDir}（不包括双引号）**，在点击”上移“按钮把我们的工具移到第一个，最后点击“确定”按钮保存设置，大功告成！

<br/>

---

### 第三步：使用

点击菜单栏的工具菜单，就可以看到我们添加的工具啦，点击后即可使用，由于我们之前已经把它移到了最上面，所以使用快捷键”**Alt+T+Enter**"就可以调用了，是不是很方便呢？

![use](http://odaps2f9v.bkt.clouddn.com/16-9-20/28025291.jpg)

<br/>

![reslut](http://odaps2f9v.bkt.clouddn.com/16-9-20/17450109.jpg)

<br/>

---

### 总结

在修改批处理的过程中发现自己的批处理知识十分匮乏，有时间的话（呵呵）一定要恶补一下。。。


<br/>
<br/>

> 参考文章
> 
> [如何编译单个cs文件](http://xloved.blog.163.com/blog/static/18571909420114854026152/)