---
layout:     post
title:      【踩坑】发布ASP.NET网站到本地IIS和云服务器
subtitle:	记录对动态网页摸索的一个下午
date:       2016-08-10 16:21:43 +0800
author:     Shao Guoji
header-img: img/post-bg-IIS.jpg
catalog:    true
tag:
    - C#
---

*这不太算教程，丑话say in top，我一直以来对网络都是“一窍不通 + 恐惧”的，以下内容也是记录自己乱搞的（根据[发布 asp.net网站 到本地IIS-ASP.NET-第七城市](http://www.th7.cn/Program/net/201311/161046.shtml)和[图文解说Win7系统机器上发布C#+ASP.NET网站 - gavin710的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/gavin710/article/details/8681204)这两篇文章），甚至都不一定正确，大家擦亮狗眼、取其精华就好。*

### 疑问：如何在服务器上发布一个网站？

申请了某某云的1元服务器域名，然而并不知道有什么乱用。总所周知，服务器一般都是用来发布网站的，然后我就陷入了一个很大的疑问中：如何在服务器上发布一个网站？

想起之前接触的类似服务器的Github Page，但GIthub Page完全就是被托管的傻瓜式操作，只支持静态网页（网站内容写死）。但如果是服务器，如果是动态网页，那又该怎么办？

接下来我尽量回忆我的思路与探索历程，纯小白自嗨，不说大神，稍微会一点的人就别浪费时间看了~~~

<br/>

---

### 动态网站？

> 动态网站并不是指具有动画功能的网站，而是指网站内容可根据不同情况动态变更的网站，一般情况下动态网站通过数据库进行架构。

某度百科得：*“目前，用于动态网站开发的主要语言有4种：ASP、ASP .NET、PHP、JSP”*，好吧又是ASP.NET——熟悉的陌生人，又在一篇教程的“ASP的安装”部分看到：

> ##### 把自己的 Windows PC 作为 Web 服务器
> 
> * 如果您安装了 IIS 或 PWS，就可以把自己的 PC 配置为一台 Web 服务器。
> * IIS 或 PWS 可以把您的计算机转变为 Web 服务器。
> * 微软的 IIS 和 PWS 是免费的 Web 服务器组件。

重点是这段：

> ##### 测试您的安装
> 在您安装完 IIS 或 PWS 之后，按照下面的步骤测试是否安装成功：
> 
> 1. 在您的硬盘中查找名为 Inetpub 的文件夹
> 2. 打开 Inetpub 文件夹，找到名为 wwwroot 的文件夹
> 3. 在 wwwroot下创建一个新文件夹，比如 "MyWeb"
> 4. 使用文本编辑器编写几行 ASP 代码，将这个文件取名为 "test1.asp" 保存在 "MyWeb" 文件夹中
> 5. 确保您的 Web 服务器正在运行，使用下面的方法确认它的运行状态：进入控制面板，然后是管理工具，然后双击"IIS 管理器"图标。
> 6. 打开您的浏览器，在地址栏键入 "http://localhost/MyWeb/test1.asp"，就可以看到您的第一个 ASP 页面了。

## 纳尼！！就可以看到您的第一个 ASP 页面了！！！

**就冲着这句话，IIS装起！必须装！！！**

<br/>

---

### 安装IIS

IIS全称互联网信息服务（Internet Information Services），是一个处理动态网页的Web服务器（同类产品有Apache、Tomcat、Jboss、nginx等）。在控制面板安装好ISS后，兴奋copy了一段ASP代码做测试（显示系统当前时间和问候语）：

```html

<%@ language="javascript" %>
<!DOCTYPE html>
<html>
<body>
<%
var d=new Date()
var h=d.getHours()

Response.Write("<p>")
Response.Write(d)
Response.Write("</p>")
if (h<12)
   {
   Response.Write("Good Morning!")
   }
else
   {
   Response.Write("Good day!")
   }
%>
</body>
</html>

```

保存为“test1.asp”文件（一开始不懂，保存为了html文件……），按照教程放到了“C:\inetpub\wwwroot”，兴冲冲地打开浏览器，输入地址"http://localhost/MyWeb/test1.asp"，“啪”一声Enter键按下去，然后……

<br/>

---

### 第一个坑：

#### 由于扩展配置问题而无法提供您请求的页面。如果该页面是脚本,请添加处理程序。

![第一个坑](http://odaps2f9v.bkt.clouddn.com/16-9-20/26096167.jpg)

划词搜索引擎搜一下，找到解决方法：

[Win7下IIS由于扩展配置问题而无法提供请求的页_百度经验](http://jingyan.baidu.com/article/7c6fb4287ea53680652c9047.html)

原来是.ASP文件类型解析的问题，简直完美解决啊有木有。然后很自然地想到了拿.aspx文件试一下，首先我们要有一个aspx网站。

<br/>

---

### ASP.Net网站的建立

之前在实验室接触过不少aspx页面，自然想到用VS建一个项目什么的，但居然还有那么多建立选项，一时也搞不懂之间的区别。

![新建ASP.NET项目](http://odaps2f9v.bkt.clouddn.com/16-9-20/34350924.jpg)

![新建ASP.NET项目2](http://odaps2f9v.bkt.clouddn.com/16-9-20/36758737.jpg)

![新建ASP.NET网站](http://odaps2f9v.bkt.clouddn.com/16-9-20/35406102.jpg)

刚刚发现有一篇文章有谈到项目与网站的区别：[VS“新建网站”与“新建Asp.Net Web 应用程序”的区别 - LisenYang的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/lisenyang/article/details/45500023)

我建了一个ASP.NET Web应用程序的Empty项目，新建一个aspx文件把刚刚的asp的代码搞过来，问题又来了，项目有了，怎么用呢？

<br/>

---

### ASP.Net网站的发布

废话也不多说，又找到了干货：

[图文解说Win7系统机器上发布C#+ASP.NET网站 - gavin710的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/gavin710/article/details/8681204)

用的是VS自带的“发布”功能，我电脑的VS2015的发布选项又与教程老版本的VS不同，反正总的意思就是选一个目录输出项目：

![VS2015发布菜单](http://odaps2f9v.bkt.clouddn.com/16-9-20/84759743.jpg)

![VS2015发布步骤2](http://odaps2f9v.bkt.clouddn.com/16-9-20/14690168.jpg)

![VS2015发布步骤3](http://odaps2f9v.bkt.clouddn.com/16-9-20/31258952.jpg)

![VS2015发布步骤4](http://odaps2f9v.bkt.clouddn.com/16-9-20/57813280.jpg)

![VS2015发布步骤5](http://odaps2f9v.bkt.clouddn.com/16-9-20/54080766.jpg)

*更多发布方法详见文章[将Asp.Net网站发布到IIS的四种方法及注意事项 - Mishchael - 博客园](http://www.cnblogs.com/mishchael/archive/2010/12/05/1897131.html)* 

（也就是说了那么多，其实直接建一个aspx文件放进去也是可以的……）

把发布文件夹的内容复制到IIS的wwwroot目录下（上面直接发布到wwwroot目录了），修改浏览器地址为“http://localhost/test/WebForm1.aspx“,感觉和asp差不多啊，同理，应该也能处理.ASPX文件吧，呵呵，图样图森破……

<br/>

---

### 第二个坑：

#### 无法识别的属性“targetFramework”。请注意属性名称区分大小写

![错误图](http://odaps2f9v.bkt.clouddn.com/16-9-20/95054951.jpg)

.NET版本不对应导致的问题，解决方法：[无法识别的属性“targetFramework”。请注意属性名称区分大小写。错误解决办法 - muchlin的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/muchlin/article/details/6800863)

<br/>

---

### 第三个坑：

#### 处理程序“PageHandlerFactory-Integrated”在其模块列表中有一个错误模块“ManagedPipelineHandler”

![错误图](http://odaps2f9v.bkt.clouddn.com/16-9-20/43690179.jpg)

.Net Framework 4.0框架尚未在IIS中注册的问题，解决方法：[asp.net发布到IIS中出现错误：处理程序“PageHandlerFactory-Integrated”在其模块列表中有一个错误模块“ManagedPipelineHandler” - 马兆娟  廊坊师范学院信息技术提高班第八期 - 博客频道 - CSDN.NET](http://blog.csdn.net/mazhaojuan/article/details/7660657)

<br/>

---

### 终极大坑：

#### 刚发现的，网站二级目录问题

![二级目录报错](http://odaps2f9v.bkt.clouddn.com/16-9-20/94126596.jpg)

怎么说呢……就是网站项目（单个页面文件不存在这样的问题）必须放在网站文件夹根目录，否则报错打不开，比如上面的默认目录wwwroot，把VS的项目（包括几个文件和几个文件夹）直接放在wwwroot，就可以通过“http://localhost/WebForm2.aspx”访问，而新建一个文件夹“abc”再放项目（二级目录下），就不能用“http://localhost/abc/WebForm2.aspx”访问了，神奇。

**又试了一下，发现只要把项目bin目录放到网站根目录即可，看来是bin目录的问题，这货一定要在根目录，原因不明，有待深究……**

解决方法：

1、在IIS中把abc目录右键“转换为应用程序”

2、把abc目录添加虚拟目录，但仍必须将可执行文件（dll等)放置在父级站点的bin目录下（所以然并卵~~~）

关于应用程序与虚拟目录详见：[iis中虚拟目录、应用程序的区别 - 学习也休闲](http://www.studyofnet.com/news/1265.html)和[IIS7中的站点，应用程序和虚拟目录详解 - 柯柏文 - 博客园](http://www.cnblogs.com/lwhkdash/archive/2013/01/12/2857650.html)还有[如何：在 IIS 中创建和配置 ASP.NET 虚拟目录](https://msdn.microsoft.com/zh-cn/library/zwk103ab(VS.80).aspx)这几篇文章（反正我没怎么看懂）。

<br/>

---

### 补充:

#### 默认文档设置

IIS管理中有“默认文档”的设置，有什么用呢，简单来说就是将页面放在网站根目录、并设置为默认文档后，就不同在url中指明文件名了（类似Github Page的index.html）。举个栗子，把上面的WebForm2.aspx添加为默认文档，就可以直接用“http://localhost”代替“http://localhost/WebForm2.aspx”访问网页了。

#### 端口号

同一台机器上的不同网站目录是用端口号区分的，默认目录的端口号是80，刚好也是http协议的“御用端口”，所以可以省略不写（上面就是），但在IIS添加其他网站时会有端口号的选项，就不能再用80了（被占用），就要用其他未被占用的端口，这时url中端口号就不能省略了，如http://localhost:8081/WebForm1.aspx。

### 长篇大论然并卵？

好像写了那么多，乱七八糟的，也没什么太大的价值，反正我觉得只要把[发布 asp.net网站 到本地IIS-ASP.NET-第七城市](http://www.th7.cn/Program/net/201311/161046.shtml)和[图文解说Win7系统机器上发布C#+ASP.NET网站 - gavin710的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/gavin710/article/details/8681204)这两篇文章结合看，问题应该不大。但我当时真的是太手忙脚乱了，像个无头苍蝇，导致我现在试图回想当时的思路、步骤都很困难、很混乱。总之只想着记下来，以后要找的话也有门啊……

## 最重要的是，对网站有一定了解后我就可以大方去买服务器了哈哈哈哈……

<br/>

---

### Windows Server 2012 R2服务器配置

噔噔噔噔~~~服务器买到手，顺便贴一下服务器IIS的配置，纯搬运……

[原文：云服务器 - 安装配置IIS及PHP - 腾讯云产品文档](https://www.qcloud.com/doc/product/213/2755)

#### 1. 安装配置IIS

##### 1.1. Windows2012R2版本示例

1) 点击Windows云服务器左下角【开始(Start)】，选择【服务器管理器(Server Manager)】，打开服务器管理界面，如下图所示：

![开始start](http://odaps2f9v.bkt.clouddn.com/16-9-20/53546426.jpg)

2) 选择【添加角色和功能】，在弹出的添加角色和功能向导弹出框”开始之前“中点击【下一步】按钮，在”安装类型“中选择【基于角色或基于功能的安装】，点击【下一步】按钮。


![添加角色和功能](http://odaps2f9v.bkt.clouddn.com/16-9-20/36320425.jpg)

![开始之前](http://odaps2f9v.bkt.clouddn.com/16-9-20/93719987.jpg)

![安装类型](http://odaps2f9v.bkt.clouddn.com/16-9-20/25161941.jpg)

3) 窗口左侧选择”服务器角色“选项卡，勾选【Web服务器（IIS）】，在弹出框中点击【添加功能】按钮后点击【下一步】按钮。

![服务器角色](http://odaps2f9v.bkt.clouddn.com/16-9-20/6596839.jpg)

![添加角色和功能向导](http://odaps2f9v.bkt.clouddn.com/16-9-20/2197045.jpg)

4) 在”功能“选项卡中点击【下一步】按钮后，在”Web服务器角色（IIS）“选项卡也点击【下一步】。

![功能](http://odaps2f9v.bkt.clouddn.com/16-9-20/10395390.jpg)

![Web服务器角色](http://odaps2f9v.bkt.clouddn.com/16-9-20/18979655.jpg)

5) 在”角色服务“选项卡中勾选【CGI】选项，点击下一步。

![角色服务](http://odaps2f9v.bkt.clouddn.com/16-9-20/97784911.jpg)

6) 确认安装并等待安装完成。

![确认](http://odaps2f9v.bkt.clouddn.com/16-9-20/45291482.jpg)

7) 安装完成后在云服务器的浏览器中访问localhost验证是否安装成功，出现以下界面即为成功安装。

![验证安装](http://odaps2f9v.bkt.clouddn.com/16-9-20/18851877.jpg)

<br/>

<br/>


> 参考文章
> 
> [发布 asp.net网站 到本地IIS-ASP.NET-第七城市](http://www.th7.cn/Program/net/201311/161046.shtml)
> 
> [图文解说Win7系统机器上发布C#+ASP.NET网站 - gavin710的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/gavin710/article/details/8681204)
> 
> [在自己的 PC 上运行 ASP - 菜鸟教程](http://www.runoob.com/asp/asp-install.html)
> 
> [ASP 变量 - 菜鸟教程](http://www.runoob.com/asp/asp-variables.html)
> 
> [Win7下IIS由于扩展配置问题而无法提供请求的页_百度经验](http://jingyan.baidu.com/article/7c6fb4287ea53680652c9047.html)
> 
> [VS“新建网站”与“新建Asp.Net Web 应用程序”的区别 - LisenYang的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/lisenyang/article/details/45500023)
> 
> [将Asp.Net网站发布到IIS的四种方法及注意事项 - Mishchael - 博客园](http://www.cnblogs.com/mishchael/archive/2010/12/05/1897131.html)
> 
> [无法识别的属性“targetFramework”。请注意属性名称区分大小写。错误解决办法 - muchlin的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/muchlin/article/details/6800863)
> 
> [asp.net发布到IIS中出现错误：处理程序“PageHandlerFactory-Integrated”在其模块列表中有一个错误模块“ManagedPipelineHandler” - 马兆娟  廊坊师范学院信息技术提高班第八期 - 博客频道 - CSDN.NET](http://blog.csdn.net/mazhaojuan/article/details/7660657)
> 
> [iis中虚拟目录、应用程序的区别 - 学习也休闲](http://www.studyofnet.com/news/1265.html)
> 
> [IIS7中的站点，应用程序和虚拟目录详解 - 柯柏文 - 博客园](http://www.cnblogs.com/lwhkdash/archive/2013/01/12/2857650.html)
> 
> [如何：在 IIS 中创建和配置 ASP.NET 虚拟目录](https://msdn.microsoft.com/zh-cn/library/zwk103ab(VS.80).aspx)
> 
> [云服务器 - 安装配置IIS及PHP - 腾讯云产品文档](https://www.qcloud.com/doc/product/213/2755)