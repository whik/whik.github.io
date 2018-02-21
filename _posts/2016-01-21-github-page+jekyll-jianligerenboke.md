---
layout:     post
title:      github page + jekyll建立个人博客
subtitle:   专注于博客本身
date:       2016-01-23
author:     Shao Guoji
header-img: img/post-bg-github-page.jpg
catalog:    true
tag:
    - 博客
---

折腾了一两天，终于把自己的独立博客给建了起来，看了一下MarkDown的教程，尝试用Subline Text2写下这第一篇文章，记录一下建博的过程，给同样想建博的童鞋作为参考（虽然网上类似教程一抓一大把），也算给自己留个纪念。

*本文只是实现“博客建立起来能正常访问”的程度，仅记录重点步骤，至于一些操作细节以及之后的博客内容美化，自己探索吧~~~* 

---

<br/>

### github page & jekyll

github不用多说，程序猿的Facebook，博客实际上就是一个网站，众所周知，建网站需要域名、空间的，而github page充当的就是空间的作用，而且重点是它**完全免费！！！**

至于jekyll，是基于ruby的一个博客系统，可生成博客的静态网页，更是号称**只需几秒钟就能让网站跑起来**，不需要懂代码，一样能建站！（所以建博客没你想象的那么高大上不是么。。。）

**整体思路如下：**

1. 注册github账号，准备好github page
2. 用jekyll生成一堆的网站代码
3. 把jekyll生成的代码搬到github上

---


**你将需要：**

* 一个github账号（可到[github](https://github.com/)官网注册）
* github for windows客户端（用于同步代码）
* ruby安装包（实现gem命令下载安装jekyll）
* jekyll

OK废话不多说，开工！

---

<br/>

### **一、注册github账号、新建仓库**

登录[github](https://github.com/)官网，然后点击……额。。。这个你们会的，都不是小孩了，不废话了。。。

创建好账号后先去验证一下邮箱，接着进到你的主页（如下图），

![github主页](http://odaps2f9v.bkt.clouddn.com/16-9-16/17828822.jpg)

点击“+New repository”新建一个仓库（所谓“仓库”就是放代码的地方啦）

![new](http://odaps2f9v.bkt.clouddn.com/16-9-16/90566620.jpg)

注意仓库命名格式是**username.github.io**，其中的username是注册github时的用户名，只有这样命名github才能识别为github page（一个用户只能拥有一个github page）。再点下面的"Create repository"完成新建仓库。

![new Create](http://odaps2f9v.bkt.clouddn.com/16-9-16/6631739.jpg)

---

<br/>

### 二、使用github for windows同步代码

先下载安装[GitHub_2_11_0_5离线安装包以及文件下载链接](http://pan.baidu.com/s/1eQYZQQu)

打开github for windows并登陆github账号，输入github账号和邮箱配置一下（不然无法commit代码），点下面“Skip”完成设置。

![Log in](http://odaps2f9v.bkt.clouddn.com/16-9-16/1671089.jpg)

![Configure](http://odaps2f9v.bkt.clouddn.com/16-9-16/38902830.jpg)

接着在左上角找到一个“+”，点Clone，选择刚建好的“username.github.io”仓库，再点下面的“Clone username.github.io”，选择存放路径，把仓库下载同步到本地。

![clone](http://odaps2f9v.bkt.clouddn.com/16-9-16/91916253.jpg)

![save](http://odaps2f9v.bkt.clouddn.com/16-9-16/71798795.jpg)

本地打开刚刚设置的保存仓库的文件夹（即“Clone username.github.io”目录），往里面随便扔个html文件，并**命名为index.html**（文件名要为index，否则显示不出来）。回到github for windows界面，会发现有“Uncommitted changes”，点击“show”按钮显示细节

![Uncommitted change](http://odaps2f9v.bkt.clouddn.com/16-9-16/23480271.jpg)

输入summary(摘要)后点“Commit to master”提交代码，最后**点右上角中间的“Push/Sync”**同步代码到github。

![commit](http://odaps2f9v.bkt.clouddn.com/16-9-16/17199396.jpg)

![push](http://odaps2f9v.bkt.clouddn.com/16-9-16/93743950.jpg)

>如果你想潇洒一点，打开github for windows自带的Git Shell(其实就是powershell)，cd进入到仓库的”username.github.io“目录，然后依次输入以下三条命令同样可实现提交并同步代码（事实上我更喜欢这样做）。

```~ $ git add --all```

```~ $ git commit -m "Summary"```

```~ $ git push -u```


再回到仓库网页setting，刷新一下即可看到github page已经自动生成了，域名为“http://username.github.io",如果刚刚有放index.html的话就可以显示其内容了。

![github code](http://odaps2f9v.bkt.clouddn.com/16-9-16/14272084.jpg)

![github setting](http://odaps2f9v.bkt.clouddn.com/16-9-16/38085155.jpg)

![side](http://odaps2f9v.bkt.clouddn.com/16-9-16/16375060.jpg)

这样你已经有了自己的网站了，如果仅仅是想把一个html网页发布的话，看到这里已经够了，但对于博客来说，好戏才刚刚开始。

---

<br/>

### 三、jekyll的安装与使用

jekyll(中文名：杰克尔，读音：把”Michael Jackson“中间两个音节倒过来念)是基于ruby的一个博客系统，用户快速生成博客所需的静态页面。

我是用Git Shell命令行下gem命令来安装，需先安装[ruby](http://rubyinstaller.org/downloads)，注意安装时要选择**“Add Ruby executables to your PATH”**

![ruby](http://odaps2f9v.bkt.clouddn.com/16-9-16/88461043.jpg)

安装完成后，打开Git Shell，输入`gem install jekyll`命令安装jekyll。

![install jekyll](http://odaps2f9v.bkt.clouddn.com/16-9-16/691858.jpg)

失败了？很正常，由于网络环境原因，访问国外服务器十分坑爹，不过我们可以使用某宝的ruby gems镜像http://ruby.taobao.org/（我也被吓到了，某宝居然还有这功能~）

**更改gems源：**

```~ $ gem sources --remove https://rubygems.org/```

```~ $ gem sources -a https://ruby.taobao.org/```

```~ $ gem sources -l```

再次运行”gem install jekyll“安装命令即可。

![installed](http://odaps2f9v.bkt.clouddn.com/16-9-16/4316801.jpg)

安装好jekyll后，可以输入”jekyll -h“命令测试一下。

![test jekyll](http://odaps2f9v.bkt.clouddn.com/16-9-16/55707611.jpg)

然后就可以用以下命令新建博客：

```~ $ jekyll new myblog```

进入博客目录：

```~ $ cd myblog```

运行博客服务器（为了进行本地预览）：

```~ $ jekyll serve```

![jekyll](http://odaps2f9v.bkt.clouddn.com/16-9-16/29373777.jpg)

打开浏览器，输入[http://localhost:4000](http://localhost:4000)就能在本地预览博客主页了。

---

<br/>

### 四、同步博客到github

最后一步，把”myblog“目录下所有文件搬到仓库的”username.github.io“文件夹下，同步到github，就可以通过"http://username.github.io"访问博客了！

**开始你的博客美（zhuang）妙（bi）之旅吧！！！**

---

<br/>

### 五、问题及解决方法（20160919更新）

图片挂了之后，想重新把流程过一遍截图，发现“再也回不到从前”：

#### 问题1、改gems源时出现"SSL_connect returned=1 errno=0"

![SSL connect erro](http://odaps2f9v.bkt.clouddn.com/16-9-19/41216050.jpg)

#### 解决方法：下载”cacert.pem“文件，添加环境变量指向。  
详细步骤参考文章[ruby  SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: - leorowe的专栏
        - 博客频道 - CSDN.NET](http://blog.csdn.net/leorowe/article/details/41968349)

---

#### 问题2、“jekyll new”新建的博客本地预览失败，github page样式乱七八糟

![jekyll serve fail](http://odaps2f9v.bkt.clouddn.com/16-9-19/60313689.jpg)

![github page fail](http://odaps2f9v.bkt.clouddn.com/16-9-20/29681347.jpg)

#### 解决方法：换其他的jekyll模板

用“jekyll new”生成博客文件似乎有问题，那么就从[Jekyll Themes](http://jekyllthemes.org/)选一个喜欢的主题使用吧。

---

#### 问题3、“jekyll serve”报错“cannot load such file -- jekyll-paginate”

这是之前换模板时遇到的的一个问题，也贴一下吧。

#### 解决方法：执行“gem uninstall --all”和 “gem install github-pages”

详细步骤见Github我提的issue [run jekyll serve failed ''cannot load such file -- jekyll-paginate“ · Issue #62 · Huxpro/huxpro.github.io](https://github.com/Huxpro/huxpro.github.io/issues/62)

和StackOverflow问题[ruby - don't have jekyll-paginate or one of its dependencies installed - Stack Overflow](http://stackoverflow.com/questions/35401566/dont-have-jekyll-paginate-or-one-of-its-dependencies-installed)下“gligoran”的回答。

<br/>
<br/>

>参考文章： 
> 
> * [用jekyll和github Pages写博客](http://my.oschina.net/laichendong/blog/499224)
> * [jekyll中文官网](http://jekyll.bootcss.com/)













