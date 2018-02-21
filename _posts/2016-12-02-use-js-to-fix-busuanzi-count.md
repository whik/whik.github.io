---
layout:     post
title:      JQuery实现“不蒜子”初始化首次数据
subtitle:	曲线救国 搞定初值
date:       2016-12-02 12:13:43 +0800
author:     Shao Guoji
header-img: img/post-bg-busuanzi.jpg
catalog:    true
tag:
    - 博客
    - 前端
---

#### 哈！我的博客多了访客计数功能，就像这样：

![主页计数](http://odaps2f9v.bkt.clouddn.com/public/16-12-2/46031795.jpg)

#### 或者这样：

![文章页计数](http://odaps2f9v.bkt.clouddn.com/public/16-12-2/48140665.jpg)

---

<br/>

### 关于“不蒜子”

![“不蒜子”首页](http://odaps2f9v.bkt.clouddn.com/public/16-12-2/61907455.jpg)

简单来说，“不蒜子”是提供静态页面计数功能的一个第三方服务（欲了解详情可拜访其[主页](http://service.ibruce.info/)和[文档](http://ibruce.info/2015/04/04/busuanzi/)），用户只需要在静态的html页面中插入**一行脚本**和**一行标签**，即可实现计数功能，官方说明如下（**带下划线加粗字是重点，要考！**）:

> 静态网站建站现在有很多快速的技术和平台，但静态是优点也有缺点，由于是静态的，一些动态的内容如评论、计数等等模块就需要借助外来平台，评论有“多说”，计数有“不蒜”！

> “不蒜子”与百度统计谷歌分析等有区别：“不蒜子”可直接将访问次数显示在您在网页上（也可不显示）；**<u>对于已经上线一段时间的网站，“不蒜子”允许您初始化首次数据。。</u>**

> 普通用户只需两步走：一行脚本+一行标签，搞定一切。追求极致的用户可以进行任意DIY。

#### 实现原理

“不蒜子”的实现原理也很简单，经初步分析，在引入的JS(JavaScript)脚本中，会把当前页面url（或某种唯一标识）注册到其第三方服务器，服务器上保存着url与对应的计数值，点击页面后通过JS更新服务器上的计数值，并在页面初始化时在本地标签加载、显示计数值。 
 
 所以**虽然静态页面弱爆了，但JS很强大啊！**

---

<br/>

### 需求分析

访问量是一个网站的主要指标（不是必要，如写博客更重要的是思考和学习），网页计数功能也成了我们的一个选择，像百度统计、谷歌统计等后台统计工具非常强大，但要想直接在页面（特别是静态页面）展示数据，“不蒜子”或许是你装X的不二之选。

**但对于那些已经有一定访问量的老站点，访问量需要在原来的数据基础上计算，这就是所谓的“初始化首次数据”**。

可看到其实“不蒜子”其实有提供“初始化首次数据”的功能，在文档中也有写到：

> 5、我的网站已经运行一段时间了，想初始化访问次数怎么办？
请先注册登录，自行修改阅读次数。

太好了，于是兴奋地点击主页右上角的“登录”按钮……

![未开发注册](http://odaps2f9v.bkt.clouddn.com/public/16-12-2/97073688.jpg)

![坑爹呢](http://odaps2f9v.bkt.clouddn.com/public/16-12-2/9566094.jpg)

纳尼，暂不开放注册？？？好吧，看看文档下面的评论，应该有网友遇到同样问题，果不出我所料……

![文档评论](http://odaps2f9v.bkt.clouddn.com/public/16-12-2/86879053.jpg)

原来是已经被作者搁置了一年多的功能，求人不如求己，直接上不行，但其实我们可以**“曲线救国”**来实现这个“初始化首次数据”的功能。
 

  思路如下：**不直接显示计数值，每次显示之前先把拿到的计数值加上一个初始值，再进行显示**，而这个中间处理可通过JS/JQuery来完成。

---

<br/>

### 曲线救国——利用JS/JQuery纠正计数值

好，思路有了，那就让我们开！干！吧！

#### 1、两行代码引入计数功能

 按“不蒜子”官方教程提示，在需要显示计数的位置插入对应的两行html代码，即可实现显示网站访问量（**注意uv是一个IP记一次，而pv是每刷新一次记一次**）。

*插入以下两行代码*

```html
    <script async src="//dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js"></script>
    <span id="busuanzi_container_site_pv">本站总访问量<span id="busuanzi_value_site_pv"></span>次</span>
```
 



![插入两行代码](http://odaps2f9v.bkt.clouddn.com/public/16-12-2/76602649.jpg)

![计数值显示](http://odaps2f9v.bkt.clouddn.com/public/16-12-2/75420620.jpg)

就可以看到计数值了，不过等一下，为什么刚运行在本地服务器的网页会有这么多访问量？上面说过，“不蒜子”是通过页面url（或者主机名神马的）来标识一个计数值，而像localhost、index.html这样的名字早已经被像我们一样的广大程序猿在测试时用烂了，自然就累计了好多次……

不用担心，部署服务器后就显示正常了：

![服务器显示正常](http://odaps2f9v.bkt.clouddn.com/public/16-12-2/11346245.jpg)

#### 2、增加CSS实现加载完毕自动显示

计数值有了，那下面我们就把初始值加上去，然后再显示吧！可是，要在什么时候加呢？**由于“不蒜子”引入的JS文件是异步加载的，所以你不知道在什么时候这个计数值数据加载完毕（视网络环境而定），也就不知道什么时候执行加法**，额…………这是个问题。。。

在看文档时发现了突破口，不蒜子提供了一个**数据加载完成后再显示标签**的功能，说是为了防止出现“本站总访问量 次”这样的尴尬局面，允许用户**先把整个标签设置为不可见，待计数值加载完毕后自动显示**。这样就有一个判别条件了，有戏！于是我们按照文档说明给标签加上**style='display:none'**css属性，所以最终插入的两行代码就是这样的：

```html
    <!-- 引入不蒜子脚本和标签 -->
    <script async src="//dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js"></script>
    <span id="busuanzi_container_site_pv" style='display:none'>本站总访问量<span id="busuanzi_value_site_pv"></span>次</span>
```

#### 2、编写JQuery代码加上初值

解决了什么时候加初始值的问题，二话不说，上代码（这里用JQuery，JS的话好像有点不一样）！

*注意：1、记得引入JQuery文件 2、注意区分标签id中的pv和uv*

```javascript
<!-- 不蒜子计数初始值纠正 -->
    <script >
        $(document).ready(function() {
            var int = setInterval(fixCount, 50);  // 50ms周期检测函数
            var countOffset = 20000;  // 初始化首次数据
           function fixCount() {                   
             if ($("#busuanzi_container_site_pv").css("display") != "none")
               {
                  $("#busuanzi_value_site_pv").html(parseInt($("#busuanzi_value_site_pv").html()) + countOffset); // 加上初始数据 
                  clearInterval(int); // 停止检测
             }  
         }           
        });
    </script> 
```

![加入JS代码](http://odaps2f9v.bkt.clouddn.com/public/16-12-2/70053675.jpg)

#### 3、运行测试

部署网站到服务器，打开页面查看计数值。测试结果如下，计数值加上了设定的初始值：

![修正后的结果](http://odaps2f9v.bkt.clouddn.com/public/16-12-2/48267677.jpg)

#### 4、代码解释

$(document).ready方法在文档加载后执行，即让网页加载完毕后执行我们的JS代码，再用setInterval设置50毫秒周期性调用fixCount()方法。countOffset这个变量就是我们要加上的初始值了（可通过网站之前的统计工具如百度统计谷歌统计查看到），这里设为20000。在重头戏fixCount()方法内，先判断标签的display属性，当不为"none"时（异步加载数据完成）获得原始数值，加上自定义的额初始值后再写入标签，最后关掉Interval，大功告成！

**这样，当我们访问页面时，会不停的检测标签的状态，一旦计数值加载出来就加上初始值纠正，再显示，解决了“不蒜子”不能初始化首次数据的问题。**

---

<br/>

### 总结

之前学的JS/JQuery都给回视频了，写的时候被各种语法细节搞得头晕转向，现在已经有“前端恐惧症”了。
 
**只想说：让男(zha)生(zha)写前端真是要命啊 >\_<~~~**

<br/>
<br/>

>参考文章： 
> 
> * [不蒜子 - 极简网页计数器](http://service.ibruce.info/)
> * [不蒜子 - 不如](http://ibruce.info/2015/04/04/busuanzi/)