---
layout:     post
title:      ESP8266 + Airkiss从零开始实现微信配网（二）
subtitle:   微信公众号开发、JSSDK、JSAPI的使用
date:       2017-01-28 10:12:50 +0800
author:     Shao Guoji
header-img: img/post-bg-wechat-airkiss2.jpg
catalog:    true
tag:
    - ESP8266
    - 物联网
    - C#
    - 微信开发
---

系列文章目录

[ESP8266 + Airkiss从零开始实现微信配网（一） - 目的、背景及准备工作]({% post_url 2017-01-16-ESP8266-wechat-onekey-config-1 %})

[ESP8266 + Airkiss从零开始实现微信配网（二） - 微信公众号开发、JSSDK、JSAPI的使用]({% post_url 2017-01-28-ESP8266-wechat-onekey-config-2 %})

[ESP8266 + Airkiss从零开始实现微信配网（三） - 编写单片机程序与WiFi模块通讯]({% post_url 2017-02-10-ESP8266-wechat-onekey-config-3 %})

---

<br/>

![微信配网](http://odaps2f9v.bkt.clouddn.com/17-2-1/3288516-file_1485882048916_1052d.jpg)

### 内容简介

利用公众号作为入口，**微信配网原生Airkiss界面可通过JS-SDK由“微信内网页（即在手机微信APP内打开的网页）”调起。**本篇我们来了解一下“微信内网页”接口开发，用C#后台代码实现微信JS-SDK注入配置，以及用前端JS调用相关接口。

---

<br/>

### 微信接口开发

![微信开放平台](http://odaps2f9v.bkt.clouddn.com/17-1-29/46694068-file_1485664439239_1116c.png)

微信作为一款即时通讯软件，却又不只是一款即时通讯软件。随着时代的发展，微信已经成为一个平台、一种生活方式。无论是微信聊天、微信支付还是朋友圈，都已经渗透到了我们生活的方方面面。而微信的强大，还在于它给开发者提供了大量的接口（包括移动应用、网站和公众号开发），大大缩短了应用的开发周期，降低了开发难度，给开发者提供了更多的可能。

#### 什么是接口（API）？

> API（Application Programming Interface,应用程序编程接口）是一些预先定义的函数，目的是提供应用程序与开发人员基于某软件或硬件得以访问一组例程的能力，而又无需访问源码，或理解内部工作机制的细节。 

说简单点，接口就是别人做好的功能，我们直接用就行了，并基于这些基础功能往上逐步搭建起我们自己丰富多彩的应用。

#### Smart Config接口

回到主角Smart Config上，前面说到**Smart Config是通过WiFi模块（无线网卡）通过广播和抓包的方式实现的，要对数据帧进行操作**。而这些如此底层、与硬件打交道的事伟大的程序猿早就做好了，我们不需要关心具体实现细节（除非你想要成为网络大牛，而不是实现一键配网），可以直接拿来用。

#### 接口的提供方式

我所了解到的接口有HTTP和SDK两种，HTTP接口基于POST和GET请求，常用Json格式字符串进行交互，使用简单但功能有限。而通过导入外部SDK的方式可以更直接使用更强大的接口。例如这次要用的Airkiss接口——包括运行在ESP8266模块的**AirKiss2.0库**，以及运行在手机微信的**JS-SDK**，通过微信APP去操控手机无线网卡（在微信内置浏览器运行的JS代码自然就能直接调用微信APP的功能）。

这次一键配网使用到微信提供的接口主要有**微信公众平台接口、微信JS-SDK、JS-API**。

---

<br/>

### 微信开发的正确姿势

正所谓“工欲善其事必先利其器”，一些好用的工具可以让我们更愉快的打代码……

#### 1、使用ngrok实现域名内网映射

> ngrok 是一款用go 语言开发的开源软件，它是一个反向代理，通过在公共的端点和
本地运行的Web 服务器之间建立一个安全的通道。

![ngrok主页](http://odaps2f9v.bkt.clouddn.com/17-1-29/11153808-file_1485665247117_e1f2.png)

使用JS-SDK需要在微信公众号绑定JS接口安全域名，且**不支持IP地址及端口号**，所以我们需要一个域名来访问页面。难道又要花重金买服务器和域名？不不不，这样就太麻烦了。对，不是钱的问题，是麻烦。难道你每次改完代码都要重新部署一遍再来看结果？我们现在可是在开发调试啊……

如果能**把在本地启动调试的网页映射到外网的域名**上就好了，所以就有了[ngrok](https://ngrok.com/)这个神器，不但能在外网访问，而且还有域名！在开发时偶得QQ浏览器团队服务器版的ngrok，炒鸡好用，推荐大家使用。按照教程[[工具] 使用 Ngrok 来帮助你实现本机外网访问](https://laravel-china.org/topics/2639)先把ngrok下载、配置好。

下载ngrok.exe文件，并在同目录下创建 ```ngrok.conf``` 配置文件，并根据需要更改配置文件。

```
trust_host_root_certs: false
server_addr: proxy.qqbrowser.cc:4443
inspect_addr: 0.0.0.0:4040
tunnels:
  wechat:
    subdomain: shaoguoji
    proto:
      http: 127.0.0.1:44073
```

我的 ```ngrok.conf``` 文件中配置了一个名为“wechat”的映射，域名为“shaoguoji”，指向本地（127.0.0.1为本机地址）的44073端口。

保存配置文件，在cmd命令行窗口进入ngrok所在目录（我的在C:\ngrok），敲入 ```ngrok start wechat``` 命令运行ngrok开始域名映射，得到的域名 ```shaoguoji.proxy.qqbrowser.cc``` 就可以用来开发啦。

![运行ngrok](http://odaps2f9v.bkt.clouddn.com/17-1-29/36616965-file_1485661834001_5b91.png)

在浏览器输入地址 ```http://127.0.0.1:4040/http/in#``` 可以进入ngrok后台管理页面，查看请求报文。

![ngrok后台管理页面](http://odaps2f9v.bkt.clouddn.com/17-1-29/43246531-file_1485664891872_1b6b.png)

这还没完呢，等下还要再修改配置，将ngrok指向网站启动的具体端口才能进行外网访问。

> 注：不知道什么原因，这个版本的ngrok长时间运行会自动退出，每次都要手动再运行，郁闷……

#### 2、微信接口测试号申请

既然是公众号开发，好像……先得有一个公众号？

不需要注册审核，微信给开发者提供了接口测试号，手机扫一扫即可登录，就在[微信公众平台开发者文档](http://mp.weixin.qq.com/wiki/home/index.html)左侧目录“开始开发”下的[接口测试号申请](http://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login)中。登录后可以扫描“测试号二维码”关注此公众号。

![接口测试号申请](http://odaps2f9v.bkt.clouddn.com/17-1-29/76677737-file_1485666789874_1590d.png)

#### 3、使用微信Web开发者工具调试网页

调用JS-SDK的页面要在微信内打开才有效，由于每次用手机调试的话略显繁琐与不便，调试信息也不全面。于是微信官方提供了“Web开发者工具”，相当于PC端的一个微信客户端模拟器了（浏览器），同样也可以扫码登录。

![Web开发者工具](http://odaps2f9v.bkt.clouddn.com/17-1-29/98577021-file_1485695897750_6036.png)

#### 4、随时查阅微信官方开发者文档

我个人是很不善于看文档开发的，对文档也非常恐惧。但**请不要学我！**很多时候在网上找了一大堆文章或视频教程来看，最后才发现文档写着清清楚楚呢……所以，遇到任何问题务必先查阅文档。

* [微信公众平台开发者文档](http://mp.weixin.qq.com/wiki/home/index.html)
* [微信JSSDK说明文档](http://mp.weixin.qq.com/wiki/7/aaa137b55fb2e0456bf8dd9148dd613f.html)
* [AirKiss2.0开发文档](http://iot.weixin.qq.com/wiki/new/index.html?page=4-1-2)

---

<br/>

### 微信公众平台开发——自定义菜单

有点标题党了，其实微信公众号并不是主角，只是在后台验证一下JS安全域名、在公众号界面提供一个菜单作为网页入口仅此而已（“微信内网页”要求运行在微信APP内置浏览器中）。直接在微信中点url打开页面也是可以的，只不过公众号提供了一个固定可发布的、更易用的入口。我们只用到了公众号的菜单功能，**但并不意味着这就是公众号开发的全部，基于消息、素材和用户管理的接口开发才是公众号开发的精髓**，本文重点是**微信内网页开发**所涉及到的JS-SDK和与硬件相关的JSAPI的使用。

#### 利用“接口在线调试工具”自定义菜单

手机扫码关注关注公众号后发现空空如也，那先加个菜单试一下吧！因为菜单不需要经常更改，可以用微信官方提供的“接口在线调试工具”调用自定义菜单接口，为公众号添加菜单，免去自己写后台代码。

> 注：对于常规公众号用户，可直接在后台管理页面增加菜单，而测试号没有此功能。

同样在[微信公众平台开发者文档](http://mp.weixin.qq.com/wiki/home/index.html)左侧目录“开始开发”下的[接口在线调试](http://mp.weixin.qq.com/debug/)中打开接口调试页面，默认是获取token的接口，把公众号后台的 ```appID``` 和 ```appsecret``` 复制填到接口参数列表输入框，点击“检查问题”按钮获取token。

![后台appID](http://odaps2f9v.bkt.clouddn.com/17-1-30/58806520-file_1485747685265_2cb9.png)

![填接口参数](http://odaps2f9v.bkt.clouddn.com/17-1-30/8236984-file_1485749563305_45a1.png)

下面返回的就是调用结果了，包含一些相应信息，其中 ```access_token``` 后的一长串就是获取到的token了，选中复制下来。

![返回token](http://odaps2f9v.bkt.clouddn.com/17-1-30/30291318-file_1485749791734_94f5.png)

token是所有接口调用的基础，有了token就可以调用其他接口了。在“接口类型”下拉框中选择“自定义菜单”，填入刚刚获取到的token，并根据[自定义菜单创建接口文档](http://mp.weixin.qq.com/wiki/10/0234e39a2025342c17a7d23595c6b40a.html)照葫芦画瓢填入Json格式的参数体，点击“检查问题”按钮调用接口，若调用成功会返回“Request successful”的提示信息。

```json
{
    "button": [
        {
            "type": "view", 
            "name": "打开百度", 
            "url": "https://www.baidu.com/"
        }
    ]
}
```

![调用自定义菜单接口](http://odaps2f9v.bkt.clouddn.com/17-1-30/67100932-file_1485748971404_829d.png)

![创建菜单成功返回](http://odaps2f9v.bkt.clouddn.com/17-1-30/35108232-file_1485750167218_8b58.png)

手机扫码关注测试号，可以看到有了一个“打开百度”的菜单，点进去就是熟悉的百度页面，也就是说**我们知道如何在公众号菜单打开网页了**！

![公众号打开百度](http://odaps2f9v.bkt.clouddn.com/17-1-30/7686219-file_1485750996722_12b5c.jpg)

---

<br/>

### 微信网页开发——JS-SDK

> 微信JS-SDK是使用JSAPI的基础，必须了解新框架的基本用法，如wx.config函数和wx.ready函数，这是所有JSAPI使用的前提。

想要使用JSAPI实现一微信配网，先要跨过JS-SDK这座小山丘。来看看JS-SDK的文档说明：

> 微信JS-SDK是微信公众平台面向网页开发者提供的基于微信内的网页开发工具包。 
> 
通过使用微信JS-SDK，网页开发者可借助微信高效地使用拍照、选图、语音、位置等手机系统的能力，同时可以直接使用微信分享、扫一扫、卡券、支付等微信特有的能力，为微信用户提供更优质的网页体验。

#### 前端与后端的配合

既然JS-SDK是基于JS的接口，那理论上来说是纯前端的东西，用静态页面和JS应该就能胜任。但微信对接口安全性是有要求的，需要token和ticket认证，而这些操作如果都放到前端JS中岂不是将自己的凭证公之于众了？你也不想别人按个F12就轻轻松松伪造你的身份吧……**所以到这里思路很清楚了：分工合作，前端负责JS-SDK接口调用，后端提供接口调用所需的凭证参数。**

![服务器响应接口网页流程](http://odaps2f9v.bkt.clouddn.com/17-1-31/30840492-file_1485837366296_17c06.png)

可当初就算道理都懂，对于还在Web开发门外徘徊的我，也是手足无措地上网找别人的demo，但下载发现不是不会用、不适用就是用不了。无奈之下只好向实验室的大神求助，他二话不说扔给了我一个教程视频，看完之后重（zhao）获（ban）新（dai）生（ma）。如果你也打算用C#实现，可以看看这个视频：[微信内网页课程（提取码：r5fb）](http://pan.baidu.com/s/1jIjuAqy)

#### Visual Studio新建ASP.Net空网站

在VS（我用的是VS2015）中新建一个ASP.Net空网站，找到解决方案管理器项目名称，右键添加一个WebForm页面，命名为 ```Index.aspx``` ，往里面写个“Hello World”做测试用。

![新建空网站](http://odaps2f9v.bkt.clouddn.com/17-1-31/51342154-file_1485867790881_1458e.png)

![添加Index页面](http://odaps2f9v.bkt.clouddn.com/17-1-31/22173437-file_1485867896974_1a74.png)

![Hello World测试](http://odaps2f9v.bkt.clouddn.com/17-2-1/15523705-file_1485956712561_8d94.png)

按下Ctrl+F5非调试运行网页，可以看到我们的“Hello World”，注意到浏览器地址栏的地址主机为 ```localhost``` ，端口号为 ```44073``` ，这个端口号是不是很熟悉？没错，就是前面配置ngrok时域名指向的端口号！我是早就知道了，但现在你应该**根据VS启动的实际端口号回头修改ngrok配置文件**了。

> 注：VS每次的启动端口号可能还会变！不同版本的VS设置方法也有区别。只要记住两点：1、确保VS每次启动网站的端口号是固定的。2、配置ngrok指向这个端口号。

![Hello World显示](http://odaps2f9v.bkt.clouddn.com/17-2-1/69775607-file_1485880772269_ebfa.png)

重新配置好ngrok后，所以你以为只要把地址从 ```http://localhost:44073/Index.aspx``` 变成 ```http://shaoguoji.proxy.qqbrowser.cc/Index.aspx``` 就能成功从外网通过域名访问？哈哈哈，我当初也是这么认为的，也成功了！是的，在实验室用VS2010能直接映射，但自己电脑的VS2015就不行，会找不到网页。瞎弄一通也没整出个所以然，没想到幕后竟隐藏着**“localhost与127.0.0.1”的大坑**。

> 注：如果你直接把“localhost:端口号”替换成ngrok域名就能访问了，那你人品真好，不用折腾。当然，也少了一次学习机会。

#### 小插曲——对“localhost与127.0.0.1”的思考...

ngrok映射到的IP是 ```127.0.0.1:44073``` ，而启动后地址栏上的是 ```http://localhost:44073/Index.aspx``` ，尝试把地址换成 ```http://127.0.0.1:44073/Index.aspx``` ，**纳尼？居然不行！**这俩货难道不是一回事么？？？于是……我开始查（huai）找（yi）资（ren）料（sheng）。。。

一番查找过后，发现其实**“localhost”只是一个名字（域名），只有通过host文件被解析成“127.0.0.1”时两者才等价**，一位网友分析的很好：

> 有人会说是本地ip，曾有人说，用127.0.0.1比localhost好，可以减少一次解析。
> 
虽说效果看起来是一样的，都是本地IP，但实际上区别很大：Localhost的意思是本地服务器，而127.0.0.1是本机地址，他们的关系是通过操作系统中的hosts文件，将Localhost解析为127.0.0.1。而实际工作中，Localhost是不经过网卡传输的，所以，它不受网络防火墙和与网卡相关的种种限制；而127.0.0.1则要通过网卡传输数据，是必须依赖网卡的。这一点是它们最大的区别。      一般设置程序时，本地服务用Localhost是最好的，Localhost不会解析成IP，也不会占用网卡、网络资源。有时候用Localhost可以，但用127.0.0.1就不可以的情况就是在于此。

再仔细想想就懂了：原来VS自带的IIS Express服务器只是把网站部署到了 ```localhost``` 名称（只是**名义上的主机**）下，且内部没有映射到 ```127.0.0.1``` 这个IP上（Tomcat就有，突然感觉Java好好哦~），是两个不同的世界。但即使把ngrok指向了 ```localhost:44073``` ，实际上ngrok还是会解析成 ```127.0.0.1:44073``` 再指向它,然而这下面啥也没有……

#### 添加IP绑定，让IIS Express也支持“127.0.0.1”

解决办法也很简单，在“127.0.0.1:44073”下也部署一下就好了，也就是让IIS Express再绑定到这个IP和端口上，操作如下：

在系统托盘处找到IIS Express图标，右键选择“显示所有应用程序”，选中我们的网站，再点下面的“配置”的路径，就会打开一个配置文件。

![IIS托盘图标](http://odaps2f9v.bkt.clouddn.com/17-2-1/21134517-file_1485879165820_6933.png)

![显示所有应用程序](http://odaps2f9v.bkt.clouddn.com/17-2-1/82368050-file_1485879334907_12451.png)

利用查找工具搜索“localhost”关键字，找到 ```name``` 属性为我们网站名的 ```<site>``` 标签，复制增加一行 ```<binding>``` 标签，只把IP改为 ```127.0.0.1```，保存文件，退出IIS Express，Ctrl+F5重启 ```Index.aspx``` 页面。

![修改配置文件增加绑定](http://odaps2f9v.bkt.clouddn.com/17-2-1/73848966-file_1485879634369_8a09.png)

这时再右击IIS Express任务栏图标，应该能看到新增了地址为 ```127.0.0.1``` 的站点，打开浏览器，用本机地址的 ```http://127.0.0.1:44073/Index.aspx``` 和域名映射的 ```http://shaoguoji.proxy.qqbrowser.cc/Index.aspx``` 都能正常访问网页了，因为ngrok实现了内网穿透，所以用手机输入域名也是可以访问的哦，换句话说，**能上网的地方就能看到我们的“Hello World”**，赶快把链接分享给你的朋友们来欣赏吧！

![IIS显示增加的地址](http://odaps2f9v.bkt.clouddn.com/17-2-1/23324301-file_1485879953698_16a5f.png)

![本机IP与域名访问成功](http://odaps2f9v.bkt.clouddn.com/17-2-1/96153434-file_1485880245361_12721.png)

域名能够正常访问后，再次利用[接口在线调试](http://mp.weixin.qq.com/debug/)重新设置一下公众号的菜单标题和链接，让按钮点击后跳转到我们域名下的站点。Json参数如下：

```json
{
    "button": [
        {
            "type": "view", 
            "name": "微信配网测试", 
            "url": "http://shaoguoji.proxy.qqbrowser.cc/Index.aspx"
        }
    ]
}
```

#### JSSDK使用步骤一、二：绑定域名和引入JS文件

花了好大篇幅终于把环境搭起来了，那接下来就让我们打开[JSSDK说明文档](https://mp.weixin.qq.com/wiki/11/74ad127cc054f6b80759c40f77ec03db.html#JSSDK.E4.BD.BF.E7.94.A8.E6.AD.A5.E9.AA.A4)，一步步开始“微信内网页”的编写吧。

##### 步骤一：绑定域名

确保ngrok域名可用后，去到公众号后台的“JS接口安全域名”选项处点击“修改”，填入ngrok域名 ```shaoguoji.proxy.qqbrowser.cc``` 后点“提交”就行了。

![公号后台绑定域名](http://odaps2f9v.bkt.clouddn.com/17-2-1/13845651-file_1485955903736_712f.png)

##### 步骤二：引入JS文件

在我们页面的 ```<head>``` 标签内加上一行 ```<script>``` 代码即可。

```html
<script type="text/javascript" src="http://res.wx.qq.com/open/js/jweixin-1.0.0.js"></script>
```
![引入JS文件](http://odaps2f9v.bkt.clouddn.com/17-2-1/50869769-file_1485956615595_5405.png)

#### JSSDK使用步骤三：通过config接口注入权限验证配置

这一步绝对是JSSDK的核心、所有接口调用的基础步骤。“注入权限”是指通过 ```wx.config()``` 方法（定义在引入的JS文件中），把appId、签名和调用接口列表等信息向微信注册的过程。只有注入配置成功、通过了微信签名认证的页面才是合法的、可以调用JS-SDK接口的页面。

```javascript
wx.config({
    debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来
    appId: '', // 必填，公众号的唯一标识
    timestamp: , // 必填，生成签名的时间戳
    nonceStr: '', // 必填，生成签名的随机串
    signature: '',// 必填，签名，见附录1
    jsApiList: [] // 必填，需要使用的JS接口列表，所有JS接口列表见附录2
});
```

可以看到config()方法需要六个参数，其中的 ```debug``` 、 ```appId``` 、 ```jsApiList``` 是已知的，而 ```timestamp``` 、 ```nonceStr``` 和 ```signature``` 是后台要提供的，也正是要用C#后台代码实现的。来分别看看这三个参数如何用代码生成。

展开“解决方案管理器”下项目中的 ```Index.aspx``` 文件，打开 ```Index.aspx.cs``` 文件编写后台代码，不断添砖加瓦，实现每个方法。

**1、引入所需的命名空间：**

```c#
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Net;
using System.Text;
using System.Web.Script.Serialization;
```

**2、定义用到的数据类型：**

```c#
public partial class Index : System.Web.UI.Page
{
    public string appID = "公众号appID";
    public string appsecret = "公众号appsecret";

    public string timestamp; // 生成签名的时间戳
    public string nonceStr; // 生成签名的随机串
    public string signature; // 签名

    class TokenResultMessage // 封装调用access_token接口返回的数据
    {
        public string access_token;
        public string expires_in;
    }

    class JsapiResultMessage // 封装调用jsapi_ticket接口返回的数据
    {
        public string errcode;
        public string errmsg;
        public string ticket;
        public string expires_in;
    }

    protected void Page_Load(object sender, EventArgs e)
    {

    }
}
```

```appID``` 和 ```appsecret``` 填入测试号后台的“appID”和“appsecret”。 ```timestamp``` 、 ```nonceStr``` 和 ```signature``` 是要给前台提供的数据，没什么好说的。而两个内部类 ```TokenResultMessage``` 和 ```JsapiResultMessage``` 是对接口返回结果的封装，成员字段是和接口的返回参数（见文档）一一对应的。两个内部类用的很巧妙，可大大简化我们的工作，后面会再分析。

**3、增加两个工具方法：**

```c#

    /// <summary>
    /// Get请求封装
    /// </summary>
    /// <param name="url"></param>
    /// <returns></returns>
    public string GetWebUrl(string url)
    {
        WebClient client = new WebClient(); // 创建浏览器
        Stream stream = client.OpenRead(url); // 传入url地址
        return new StreamReader(stream).ReadToEnd(); // 得到响应字符串
    }

    /// <summary>
    /// sha1签名算法
    /// </summary>
    /// <param name="str"></param>
    /// <returns></returns>
    public string Sha1(string str)
    {
        var sha1 = System.Security.Cryptography.SHA1.Create();
        byte[] bytes = Encoding.UTF8.GetBytes(str);
        byte[] bytesArr = sha1.ComputeHash(bytes);
        StringBuilder sb = new StringBuilder();
        foreach (var item in bytesArr)
        {
            sb.AppendFormat("{0:x2}", item);
        }
        return sb.ToString();
    }
```

```GetWebUrl()``` 方法封装了HTTP协议GET方法的请求与响应结果的读取与返回，传入url即可使用。例如请求地址 ```https://www.baidu.com/``` 就会返回百度首页的html代码，而请求一个接口，则返回Json结果字符串，用于调用HTTP接口时结果的接收。

```Sha1()``` 实现了字符串的sha1签名，并把结果以十六进制字符的形式返回。

**4、生成签名的时间戳**

```c#
    /// <summary>
    /// 生成时间戳
    /// </summary>
    private void GetTimeStamp()
    {
        TimeSpan ts = DateTime.UtcNow - new DateTime(1970, 1, 1);
        timestamp =  Convert.ToInt64(ts.TotalSeconds).ToString();
    }
```

时间戳用于区分接口调用的不同时刻，一般它表示从1970
年1月1日00:00:00 到当前时间的秒数之和（Unix时间戳）。

**5、生成签名的随机串**

```c#
    /// <summary>
    /// 生成32位随机字符串
    /// </summary>
    private void GetRandomStr()
    {
        string strArr = "0123456789QWERTYUIOPASDFGHJKLZXCVBNMqwertyuiopasdfghjklzxcvbnm";
        Random rand = new Random();

        string randStr = "";
        for (int i = 0; i < 32; i++)
        {
            int index = rand.Next(strArr.Length);
            randStr += strArr.Substring(index, 1);
        }
        nonceStr = randStr;
    }
```

可以看到生成随机串使用类似于“抽奖”的方式——从数字和大小写字母中随机抽取32位组成字符串（文档中对接口随机串的要求是**“不长于32位。推荐使用大小写字母和数字”**）。

**6、生成签名**

签名是页面合法性验证的重要凭证，前面的时间戳、随机串都是用来生成签名的，文档对签名算法的说明如下：

> 签名生成规则如下：参与签名的字段包括noncestr（随机字符串）, 有效的jsapi_ticket, timestamp（时间戳）, url（当前网页的URL，不包含#及其后面部分） 。对所有待签名参数按照字段名的ASCII 码从小到大排序（字典序）后，使用URL键值对的格式（即key1=value1&key2=value2…）拼接成字符串string1。这里需要注意的是所有参数名均为小写字符。对string1作sha1加密，字段名和字段值都采用原始值，不进行URL 转义。

我们所要做的，就是把上面的一大段文字翻译成代码。

```c#
    /// <summary>
    /// 生成签名
    /// </summary>
    private void GetSignatur()
    {
        // 通过API获取access_token
        string tokenUrl = "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=" + appID + "&secret=" + appsecret;
        var tokenInfo = new JavaScriptSerializer().Deserialize<TokenResultMessage>(GetWebUrl(tokenUrl));

        // 通过API获取jsapi_ticket
        string jsapiUrl = "https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token=" + tokenInfo.access_token + "&type=jsapi";
        var jsapiInfo = new JavaScriptSerializer().Deserialize<JsapiResultMessage>(GetWebUrl(jsapiUrl));

        // 当前网页的URL，不包含#及其后面部分
        string nowUrl = "http://shaoguoji.proxy.qqbrowser.cc/Index.aspx";
        //string nowUrl = Request.Url.AbsoluteUri; // 部署服务器后应动态获取页面url

        // 用Dictionary集合存储各变量
        Dictionary<string, string> dic = new Dictionary<string, string>();
        dic["timestamp"] = timestamp;
        dic["noncestr"] = nonceStr;
        dic["url"] = nowUrl;
        dic["jsapi_ticket"] = jsapiInfo.ticket;

        // 对变量名字典排序
        string[] arrKey = new string[] { "timestamp", "noncestr", "url", "jsapi_ticket" };
        arrKey = arrKey.OrderBy(n => n).ToArray();

        // 拼接字符串
        string signatureStr = "";
        foreach (var item in arrKey)
        {
            signatureStr += item + "=" + dic[item] + "&";
        }
        signatureStr = signatureStr.Substring(0, signatureStr.Length - 1);

        // 对拼接串sh1签名，得到最终签名
        signature = Sha1(signatureStr);
    }
```

签名算法看似复杂，其实只是**把几个变量名和值按顺序拼接成字符串再进行sh1签名**而已。注意到通过接口获取 ```access_token``` 和 ```jsapi_ticket``` 处的两行代码：

```C#
var tokenInfo = new JavaScriptSerializer().Deserialize<TokenResultMessage>(GetWebUrl(tokenUrl));

var jsapiInfo = new JavaScriptSerializer().Deserialize<JsapiResultMessage>(GetWebUrl(jsapiUrl));
```

我们知道，调用这两个HTTP接口会返回结果Json字符串，包含许多的结果信息。而怎样拿到所需要的那一个值呢？还记得当初声明的两个内部类 ```TokenResultMessage``` 和 ```JsapiResultMessage``` 么，而这里就是通过调用 ```JavaScriptSerializer```类的 ```Deserialize<T>(String)``` 方法对Json字符串进行序列化，并转换为内部类对象（强大的泛型）。这样就把Json字符串转换为对象了，直接用 ```tokenInfo.access_token``` 和 ```jsapiInfo.ticket``` 的写法就能方便地使用所需的接口结果值。

**此外还有一点要特别注意：页面的url应该动态获取。文档中的[附录5-常见错误及解决方法](https://mp.weixin.qq.com/wiki/11/74ad127cc054f6b80759c40f77ec03db.html#.E9.99.84.E5.BD.955-.E5.B8.B8.E8.A7.81.E9.94.99.E8.AF.AF.E5.8F.8A.E8.A7.A3.E5.86.B3.E6.96.B9.E6.B3.95)也写得非常清楚：**

> 确保你获取用来签名的url是动态获取的，动态页面可参见实例代码中php的实现方式。如果是html的静态页面在前端通过ajax将url传到后台签名，前端需要用js获取当前页面除去'#'hash部分的链接（可用location.href.split('#')[0]获取,而且需要encodeURIComponent），因为页面一旦分享，微信客户端会在你的链接末尾加入其它参数，如果不是动态获取当前链接，将导致分享后的页面签名失败。

在调试环境下我写死了（否则获取到的就是 ```http://localhost:44073/Index.aspx``` ，与公众号后台绑定的域名不一致导致签名失败），**但把网站部署服务器后应该使用 ```Request.Url.AbsoluteUri``` 来动态获取。**

到这里后台代码的编写就完成了，那下面看看如何在页面JS中调用后台生成的数据。

**7、编写页面JS代码**

要在JS实现配置注入，把提供的 ```config()``` 方法模板复制到页面中新的 ```<script>``` 标签下 ```window.onload``` 事件中（文档加载完执行），填上对应的变量名（与后台代码对应）标签即可。接口列表填“获取‘分享到朋友圈’按钮点击状态及自定义分享内容接口”**onMenuShareTimeline**：

```javascript
<script type="text/javascript">
    window.onload = function () {
        wx.config({
            debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
            appId: '<%=appID %>', // 必填，公众号的唯一标识
            timestamp: '<%=timestamp %>', // 必填，生成签名的时间戳
            nonceStr: '<%=nonceStr %>', // 必填，生成签名的随机串
            signature: '<%=signature %>',// 必填，签名，见附录1
            jsApiList: ["onMenuShareTimeline"] // 必填，需要使用的JS接口列表，所有JS接口列表见附录2
        });
    };
</script>
```

**8、测试网页的配置注入**

后台和页面都写好后就来见证成果吧。Ctrl+F5运行网页，打开“微信web开发者工具”（或手机公众号菜单），输入地址  ```http://shaoguoji.proxy.qqbrowser.cc/Index.aspx``` 访问我们的页面，若提示 **{"errMsg":"config:ok"}**那就说明注入配置成功了！同时在“JS-SDK”和“权限列表”选项卡中可查看更详细的信息。

![ok](http://odaps2f9v.bkt.clouddn.com/17-2-6/97012600-file_1486383923195_1204b.png)

![查看获得权限](http://odaps2f9v.bkt.clouddn.com/17-2-6/68970626-file_1486383021140_4ef4.png)

#### JSSDK使用步骤四：通过ready接口处理成功验证

既然注入权限配置成功了，那就可以任性的调用接口了哈哈~~~就用刚刚在列表里写的**onMenuShareTimeline**接口做测试吧。由于所有的接口调用都要在注入配置之后，对于在页面加载时调用的接口，微信提供了 ```ready()``` 方法：

> config信息验证后会执行ready方法，所有接口调用都必须在config接口获得结果之后，config是一个客户端的异步操作，所以如果需要在页面加载时就调用相关接口，则须把相关接口放在ready函数中调用来确保正确执行。对于用户触发时才调用的接口，则可以直接调用，不需要放在ready函数中。

在 ```<script>``` 标签的加入ready方法，并在其中定义onMenuShareTimeline接口要实现的内容（详见文档）。这里没用实现分享功能，只借助通用参数实现用户操作的弹窗提示（处理不同按钮的点击事件）。

```javascript
wx.ready(function(){
    wx.onMenuShareTimeline({
        title: '', // 分享标题
        link: '', // 分享链接
        imgUrl: '', // 分享图标
        trigger: function (res) {
            alert('用户点击分享到朋友圈');
        },
        success: function (res) {
               alert('已分享到朋友圈');
        },
        cancel: function (res) {
            alert('已取消到朋友圈');
        },
        fail: function (res) {
            alert(JSON.stringify(res));
        }
    });
});
```

重新刷新页面，点击菜单按钮的分享到朋友圈，在触发不同按钮事件时如果弹出相应提示，那么就证明接口调用成功啦，理论上来说调用其他接口也是没问题的。

![分享到朋友圈](http://odaps2f9v.bkt.clouddn.com/17-2-6/50818878-file_1486388171035_ba03.png)

![确定、取消分享到朋友圈](http://odaps2f9v.bkt.clouddn.com/17-2-6/42715157-file_1486388446257_eb9c.png)

---

<br/>

### AirKiss应用——微信配网JSAPI

了解了JS-SDK的基本使用后，调用Airkiss的JSAPI那是分分钟的事……

> 在JS-SDK初始化的基础上，请按如下方法使用微信硬件JSAPI：  
a.需要在wx.config的方法的参数jsApiList数组中，传入需要额外使用的jsapi名称。(在使用任何jsapi的接口前，必须先调用wx.config方法)。  
b. 需要在config方法中传入一个beta字段，并复制为true，则会在注入wx.invoke方法来调用还未开放的jsapi方法。（页面加载时就调用jsapi，则必须放到wx.ready回调中）。

#### 修改config方法

在原来的 ```config``` 修改两处即可：

```javascript
window.onload = function () {
    wx.config({
        beta: true,
        debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
        appId: '<%=appID %>', // 必填，公众号的唯一标识
        timestamp: '<%=timestamp %>', // 必填，生成签名的时间戳
        nonceStr: '<%=nonceStr %>', // 必填，生成签名的随机串
        signature: '<%=signature %>',// 必填，签名，见附录1
        jsApiList: ["onMenuShareTimeline", "configWXDeviceWiFi"] // 必填，需要使用的JS接口列表，所有JS接口列表见附录2
    });
};
```

要用到的配网接口为 ```configWXDeviceWiFi``` ，官方说明如下：

|函数名|configWXDeviceWiFi|
|描叙|调起原生AirKiss界面，不需要先调用（只支持WIFI设备）openWXDeviceLib|
|参数|key（可选）：base64 编码的AirKiss的密钥|
|返回值|configWXDeviceWiFi:ok　//配置成功<br>configWXDeviceWiFi:fail //超时<br>configWXDeviceWiFi:cancel  //用户取消<br>//返回res.desc，取值如下：<br>wifi_not_connected //当res.err_msg为config_wx_device_wifi:cancel时<br>//其它情况为空|

#### 调用配网接口

**在 ```ready``` 方法中增加一行 ```wx.invoke('configWXDeviceWiFi');``` 即可！**。

由于Web开发者工具不支持JSAPI的模拟，只能用手机测试，效果如下。

![原生AirKiss界面](http://odaps2f9v.bkt.clouddn.com/17-2-10/45127446-file_1486696225215_16d46.jpg)

**弹出原生AirKiss界面！我们成功了！！**

> 注：要为设备配置的WiFi就是手机当前连接的WiFi。如果手机未连接WiFi会有相应提示——当然，这都是微信帮我们实现的。

#### 增加引导页

用户一脸懵逼的看着弹出的配网界面可不是什么好事，我们还应该加上操作提示，更何况这是要和硬件配合使用的功能。

去掉原来的**Hello World**，在页面中加入引导页html代码，包括图标、提示信息及配置按钮（*可根据个人喜好调整图标及样式，但要注意屏幕适配*），并把在页面加载就调用的 ```wx.invoke('configWXDeviceWiFi');``` 放到按钮的 ```onclick``` 事件内，实现**先显示引导页，用户点击按钮后再弹出配网界面**的效果。：

```html
<body>
    <form id="form1" runat="server">
    <div>
        <div align="center"><img style="margin-top:7rem;"; src="http://odaps2f9v.bkt.clouddn.com/17-2-10/85209391-file_1486697604972_fa49.jpg"/></div>
        <p style="text-align:center;font-size:50px;">请长按模块上的网络配置按钮</p>
        <p style="text-align:center;font-size:50px;">待红色LED点亮后继续</p>
       <div align="center"><button style="background-color:#343434;color:white;width:90%;height:10rem;font-size:5rem;"; onclick="wx.invoke('configWXDeviceWiFi')">继续配置</button></div>
    </div>
    </form>
</body>
```

好了之后效果应该是这样的（也可以扫码关注我的测试号查看DEMO）：

![引导页](http://odaps2f9v.bkt.clouddn.com/17-2-10/65885133-file_1486699128713_e09c.jpg)

> 注：确保各项功能正常后就可以通过设置 ```debug: false``` 把烦人的调试弹窗信息关掉了……

至此，微信配网软件部分就完成了，至于能不能用，要拿WiFi模块来试试才知道。

---

<br/>

### 成果检验——用sscom测试WiFi模块Airkiss功能

如何用写好的页面来配置WiFi模块上网呢？首先要知道，**WiFi模块并不是随时都能进行智能配置，而是要通过 ```AT+CWSTARTSMART``` 指令开启“SmartConfig”后才能接收来自手机的无线ssid和密码，完成配置后还必须退出“SmartConfig”**，应该说智能配置是个临时的操作。

按照以下步骤测试WiFi模块的智能配置功能：

1. 把WiFi模块通过串口与电脑连接（详见文章[ESP8266串口WiFi模块的基本使用]({% post_url 2017-01-15-ESP8266-usage %})）

2. 在sscom确保AT指令运行正常后，发送 ```AT+CWMODE=1``` 指令配置模块为sta模式。 

3. 发送 ```AT+CWSTARTSMART``` 开启模块的“SmartConfig”功能，返回OK，此时模块处于混杂模式下进行无线抓包。

4. 手机连接WiFi，打开刚刚写好的微信页面，弹出配网界面后输入WiFi密码后点击“连接”按钮。

5. 此时留意到sscom，配网过程中模块串口会打印配置进度等信息，连接成功后可通过 ```AT+CWJAP?``` 指令查看模块连接WiFi情况。

6. 发送 ``AT+CWSTOPSMART``` 停止 SmartConfig。

![SmartConfig串口信息](http://odaps2f9v.bkt.clouddn.com/17-3-5/61504864-file_1488687035189_133d.png)

![SmartConfig手机端](http://odaps2f9v.bkt.clouddn.com/17-3-5/41475700-file_1488687044519_aea.png)

如果看到串口打印出 ```Smartconfig connected WiFi``` 那就证明WiFi模块成功通过微信Airkiss连接到了我们指定的WiFi（手机连接的WiFi）。

注意：

1. 用户可向 Espressif 申请 SmartConfig 的详细介绍文档。

2. 仅支持在 ESP8266 单 station 模式下调用。

3. 消息 ```Smart get WiFi info``` 表示 Smart Config 成功获取到AP信息。之后 ESP8266 尝试连接AP，打印连接过程。

4. 消息 ```Smartconfig connected WiFi``` 表示成功连接到AP，此时可以调用“AT+CWSTOPSMART”停止 SmartConfig 再执行其他指令，注意，**在SmartConfig 过程中请勿执行其他指令**。

5. 从 AT_v1.0 开始， SmartConfig 可以自动获取协议类型，Airkiss或者 ESP-TOUCH。

**无论 SmartConfig 成功与否，都请用“AT+CWSTOPSMART”释放快连占用的内存**。

---

<br/>

### 小结

写了那么长，大部分是在讲如何调出微信配网界面。其实如果只是为了使用的话，你完全可以直接关注我的测试号，用我写的页面调出配网界面进行配网（如果你实在搞不定的话），因为这个页面是微信提供的，谁调出来的都一样。而WiFi模块也只是用到了几条AT指令，所以实际上不需要打什么代码也可以轻松实现微信配网。

![测试号二维码](http://odaps2f9v.bkt.clouddn.com/17-3-7/4773376-file_1488878394729_7e3b.png)

源码下载：[JS-SDK 密码anlg](http://pan.baidu.com/s/1kUW32GR)，（**运行前请把“公众号appID”和“公众号appsecret”替换成你自己的，否则报错**）

下一篇来看看如何用单片机模拟手动输入AT指令来控制WiFi模块。

---

<br/>
<br/>

>参考文章： 
> 
> * [微信公众平台开发者文档](http://mp.weixin.qq.com/wiki/home/index.html)
> * [微信JSSDK说明文档](http://mp.weixin.qq.com/wiki/7/aaa137b55fb2e0456bf8dd9148dd613f.html)
> * [AirKiss2.0开发文档](http://iot.weixin.qq.com/wiki/new/index.html?page=4-1-2)
> * [[工具] 使用 Ngrok 来帮助你实现本机外网访问](https://laravel-china.org/topics/2639)
> * [vs visual studio 让外网访问设置](http://www.cnblogs.com/vktun/p/6112372.html)
> * [笔者支招:区分Localhost与127.0.0.1之间的差别](http://jingyan.baidu.com/article/3aed632e4ff3be701180914d.html)
> * [如何在微信上开发使用Airkiss技术](http://jingyan.baidu.com/article/afd8f4de675fce34e286e923.html)
> * [ESP8266 AT指令集](http://espressif.com/sites/default/files/documentation/4a-esp8266_at_instruction_set_cn.pdf)


