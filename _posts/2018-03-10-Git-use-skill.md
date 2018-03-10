---
layout:     post
title:      Git的使用
subtitle:	Git初步学习
date:       2018-03-10 19:56:40 +0800
author:     Wang Chao
header-img: img/Git-Logo.jpg
catalog:    true
tag:
    - 版本控制
    - Git
    - Github
---

##Git是什么？

Git是目前世界上最先进的分布式版本控制系统（没有之一）。

Git有什么特点？简单来说就是：高端大气上档次！

##那什么是版本控制系统？
如果你用Microsoft Word写过长篇大论，那你一定有这样的经历：

想删除一个段落，又怕将来想恢复找不回来怎么办？

有办法，先把当前文件“另存为……”一个新的Word文件，再接着改，改到一定程度，再“另存为……”一个新文件，这样一直改下去，最后你的Word文档变成了这样：


来自本地仓库的图片
<figure>
<a><img src="{{site.url}}/img/git2.jpg"></a>
</figure>


来自网络的图片

![图片](https://cdn.liaoxuefeng.com/cdn/files/attachments/0013848606651673ff1c83932d249118bf8fd5c58c15ca2000/0)


过了一周，你想找回被删除的文字，但是已经记不清删除前保存在哪个文件里了，只好一个一个文件去找，真麻烦。

看着一堆乱七八糟的文件，想保留最新的一个，然后把其他的删掉，又怕哪天会用上，还不敢删，真郁闷。

更要命的是，有些部分需要你的财务同事帮助填写，于是你把文件Copy到U盘里给她（也可能通过Email发送一份给她），然后，你继续修改Word文件。一天后，同事再把Word文件传给你，此时，你必须想想，发给她之后到你收到她的文件期间，你作了哪些改动，得把你的改动和她的部分合并，真困难。

于是你想，如果有一个软件，不但能自动帮我记录每次文件的改动，还可以让同事协作编辑，这样就不用自己管理一堆类似的文件了，也不需要把文件传来传去。如果想查看某次改动，只需要在软件里瞄一眼就可以，岂不是很方便？

这个软件用起来就应该像这个样子，能记录每次文件的改动：



| 版本 | 文件名 | 用户 | 说明 | 日期 | 
| ---- | ------ | ------- | ------ | ---- | 
| 1 | test.doc  | 张三 | 创建文件      | 2018年2月10日 21:12:52 | 
| 2 | test.doc  | 李四 | 增加项目要求  | 2018年3月1日 12:27:22 | 
| 3 | test.doc  | 王五 | 修改项目要求  | 2018年3月10日 20:29:15 | 

这样，你就结束了手动管理多个“版本”的史前时代，进入到版本控制的20世纪。


创建SSH KEY:
`ssh-keygen -t rsa -C "youremail@example.com"`

用户主目录`.ssh`目录，里面有`id_rsa`和`id_rsa.pub`两个文件`id_rsa`是私钥，不能泄露，`id_rsa.pub`是公钥


在Github上创建一个新仓库

克隆到本地:`git clone git@github.com:xxxx`


文件修改增加到暂存区`git add filename`

当前文件夹所有文件都增加到暂存区`git add .`

提交更改：`git commit -m "modified what"`

本地修改提交到远程仓库master分支（默认分支）`git push origin master`
或者
`git push -u origin master`
或者`git push -u`

查看git日志:`git log`

本地回退到某个版本
`git reset --hard log_id`

本地回退更新到远程仓库
`git push -f origin master`

更新你的本地仓库至最新改动，执行：
`git pull`



查看当前分支
`git branch`或者`git branch -a`

新建分支
`git branch debug`


创建一个叫做“feature_x”的分支，并切换过去：
`git checkout -b feature_x`

切换分支：
`git checkout master`

删除分支：
`git branch -d feature_x`

分支推送到远程仓库
`git push origin <branch>`


合并其他分支到当前分支（例如 master），执行：
`git merge <branch>`

增加标签，如1b2e1d63ff版本为1.0.0 :
`git tag 1.0.0 1b2e1d63ff`

假如你操作失误（当然，这最好永远不要发生），你可以使用如下命令替换掉本地改动：
`git checkout -- <filename>`
此命令会使用 HEAD 中的最新内容替换掉你的工作目录中的文件。已添加到暂存区的改动以及新文件都不会受到影响。


