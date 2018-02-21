---
layout:     post
title:      Sublime Text tmpl插件快速生成md文件YAML头
subtitle:   让写博客更简单
date:       2016-02-20 11:32:22 +0800
author:     Shao Guoji
header-img: img/post-bg-jekyll-head.jpg
catalog:    true
tag:
    - 博客
---

<br/>

不得不说Sublime Text果然是一款强大的文本编辑神器，拥有大量插件。正是因为如此，我坚信一定有这样一个插件，能帮我自动生成jekyll文章中YAML头的信息。昨晚百度搜狗了一下（果然不会下意识用谷歌），果然被我在互联网的大海中捞到了tmpl这根针。（当然大神都是自己写程序生成的，我这个渣渣嘛。。。）

![Sublime Text](http://odaps2f9v.bkt.clouddn.com/2016-02-20-Sublime_Text_pakage_generate_md_file_YAML_head_information/Sublime%20Text.png)

<br/>


**以前写文章：新建一个md文件 --> 打开旧的文章 --> 复制粘贴头信息 --> 手工改日期…**

**现在写文章：ctrl+Alt+M --> 保存文件 --> 写写写**

<br/>

**Sublime Tmpl是新建文件的模板插件，允许用户自定义新建文件的类型与对应模板。**安装tmpl插件后进行简单配置，就可实现快捷键新建一个带YAML头的新文件（自动生成时间），并定位输入光标，只需保存文件即可开始写作。

<br/>

### 就简简单单的五步：
1. 安装Sublime Tmpl插件
2. 增加模板文件
3. 修改日期变量格式
4. 配置菜单选项及快捷键
5. 使用tmpl新建md文件

---

<br/>

### 第一步、安装Sublime Tmpl插件

Ctrl+Shift+P 打开Sublime Text控制台

> Package Control / Install Package, 搜索"Sublime Tmpl" 或 "tmpl", 安装

安装完成后，会看到“File”菜单多出了Sublime Tmpl的新建文件选项

![menu](http://odaps2f9v.bkt.clouddn.com/2016-02-20-Sublime_Text_pakage_generate_md_file_YAML_head_information/menu.png)

---

<br/>

### 第二步、增加模板文件

**默认模版路径: "Data\Packages\SublimeTmpl\templates" 目录.**

到模板目录随便找一个文件，弄一个副本，改名为md.tmpl，打开，清空原内容，写入以下内容并保存：

![tempalate](http://odaps2f9v.bkt.clouddn.com/2016-02-20-Sublime_Text_pakage_generate_md_file_YAML_head_information/tempalate.png)

应该都看出来了，“${date}”是日期变量，而“${1:输入文章标题}”是新建文件后输入光标的定位及提示信息，便于新建文件后标题的输入，其他文本内容原样显示。

---

<br/>

### 第三步、修改日期变量格式

模板中日期变量“${date}”的默认格式为"%Y-%m-%d %H:%M:%S"，**而jekyll YAML头信息要求时间后加上时区+0800（北京时间）**，所以我们还要修改一下${date}的配置文件。

**tmpl配置文件路径："Data\Packages\SublimeTmpl\SublimeTmpl.sublime-settings"**

![time setting](http://odaps2f9v.bkt.clouddn.com/2016-02-20-Sublime_Text_pakage_generate_md_file_YAML_head_information/time%20setting.png)

打开配置文件，找到"date_format"处，加上时区，改为"%Y-%m-%d %H:%M:%S +0800"，保存即可。

**（或者只保留日期去掉时间，但有时间一定要加时区！！！否则文章识别不了，亲身踩坑。。。）**

---

<br/>

### 第四步、配置菜单选项及快捷键

#### 1、配置菜单选项

通过菜单和快捷键都能使用我们自定义的模板来新建文件，但想要新的类型在菜单中显示，只有模板文件是不够的，还需要修改菜单配置文件。

**菜单配置文件路径："Data\Packages\SublimeTmpl\Main.sublime-menu"**

打开菜单配置文件，照葫芦画瓢地加入markdown格式的配置即可，注意逗号

![menu setting](http://odaps2f9v.bkt.clouddn.com/2016-02-20-Sublime_Text_pakage_generate_md_file_YAML_head_information/menu%20setting.png)

#### 2、配置快捷键

如果说菜单是可有可无的，那么快捷键就是必备的了。

**快捷键配置文件路径： "Data\Packages\SublimeTmpl\Default.sublime-keymap"**

打开快捷键配置文件，抄！

![keymap](http://odaps2f9v.bkt.clouddn.com/2016-02-20-Sublime_Text_pakage_generate_md_file_YAML_head_information/keymap.png)

---

<br/>

### 第五步、使用tmpl新建md文件

**方法一（菜单）：File --> New File(Sumlime Tmpl) --> markdown**

**方法二（快捷键）：ctrl+Alt+M**

两种方法，你喜欢咯，方便快捷呢~~~

<br/>
<br/>

> 参考文章：
> 
> * [Sublime Text 新建文件可选类型的模版插件: SublimeTmpl - tinyphp - 博客园](http://www.cnblogs.com/tinyphp/p/3594547.html)
> * [Sublime Text 新建文件的模版插件: SublimeTmpl - 专注web前端开发](http://www.fantxi.com/blog/archives/sublime-template-engine-sublimetmpl/)




