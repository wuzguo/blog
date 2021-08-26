---
title: Git彩蛋

date: 2018-03-15 15:15:23

author: Zak

avatar: /blog/images/avatar.png

authorLink: http://www.wuzguo.com

authorAbout: https://github.com/wuzguo

authorDesc: 一个追求进步的「十八线码农」

categories: 工具

tags:
	- Git

	- GitHub

	- 彩蛋

keywords: Git，GitHub，彩蛋

photos:
	- /blog/images/201803/10.jpg

description: Git图形化展示日志的命令
---


最近学到一条非常流弊的Git命令，记录一下，先看下效果：

![](/blog/images/201803/10.jpg)

命令为：`git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative`

如果嫌命令太长可以给他设置个别名，方式如下：
`git config --global alias.lg "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative"`

下次再用的时候就只需要敲：`git lg` 即可。