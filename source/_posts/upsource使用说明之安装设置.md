---
title: Upsource使用说明之安装配置
date: 2017-08-26 23:03:10 
author: wúzguó
avatar: /blog/images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wzguo
authorDesc: 一个自强不息，艰苦奋斗的「十八线码农」
categories: 工具
tags: 
	- Upsource
	- Code Review
	- 代码审查
	- Intellij IDEA
keywords: Upsource，Code Review，代码审查，Intellij IDEA
photos:
	- /blog/images/201708_2/1.png
description: 代码审查工具Upsource使用说明系列文章之安装配置
---


### 一、简介

Upsource是一款优秀的代码审查工具，UI简洁大方，功能强大，浏览代码格式美观，可以跟IDEA无缝集成完成代码审查工作，目前官方最新版本是2017.2.2197，已经被我破解，有需要的可以联系我。

### 二、安装

1. 下载合适版本的Upsource安装程序，将其解压到任意磁盘，进入Upsource根目录(下面统称upsource_home)，准备进行安装。

2. 以管理员身份运行打开cmd，切换到目录，执行命令

   ```shell
   upsource_home\bin\upsource.bat start
   ```


![](/blog/images/201708_2/1.png)

当出现 upsource is running 提示时说明启动完成。

![](/blog/images/201708_2/2.png)

3. 启动完成后会打开默认浏览器网址http://机器名:80/welcome 页面，注意在windows中默认端口为80，在以前的版本端口默认是8080。这是你会看到如下页面：

![](/blog/images/201708_2/3.png)

当看到以上界面时说明安全正常完成。



### 三、设置

1. 初始化

   点击 `Set up` 链接，这时我们可以修改访问域名和端口和管理员账号。

![](/blog/images/201708_2/4.png)

​	点击 `next` 按钮设置其他信息，最后点击 `finish` 按钮静候`upsource`的初始化操作，这里需要一点时间。

![](/blog/images/201708_2/5.png)

![](/blog/images/201708_2/12.png)

​	初始化完成后，用管理员账号登录便可进入欢迎界面。

![](/blog/images/201708_2/13.png)

2. 设置默认值

   初始化完成后可以设置`upsource` 的全局信息设置，如语言，字体，session超时时间等。

![](/blog/images/201708_2/15.png)

3. 添加用户，用户组，项目。

![](/blog/images/201708_2/16.png)

然后给用户设置角色。upsource有五种角色，分别是：`Code Viewer`、`Developer`、`Observer`、`Project Amin`、`System Admin`。可以根据用户在项目组中担任的角色赋予相应的角色。一个账号可以有多个角色。

![](/blog/images/201708_2/17.png)


