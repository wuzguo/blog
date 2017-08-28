---
title: Upsource使用说明之在IDEA中集成Upsource
date: 2017-08-27 09:14:56 
author: wúzguó
avatar: /blog/images/avatar.png
authorLink: https://wzguo.github.io
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
	- /blog/images/201708_3/29.png
description: 代码审查工具Upsource使用说明系列文章之安装配置
---


### 一、简介
    Upsource是一款优秀的代码审查工具，UI简洁大方，功能强大，浏览代码格式美观，可以跟IDEA进行无缝集成，完成代码审查工作。

### 二、集成IDEA

1.  首先在IDEA的插件仓库中下载Upsource插件

![29](/blog/images/201708_3/29.png)

  在IDEA中打开这个页面的路径是`  file -> settings ->plugins ->browse repositories`，由于我本机上已经安装了该插件所以显示为 `update`，安装完成后重启IDEA插件生效。

2. 配置服务器和用户名
   插件安装完成后会在IDEA的右下角有个`Upsource`的图标，点击图标可以配置Upsource的服务器信息。

![39](/blog/images/201708_3/39.png)

   点击图标弹出配置服务器信息，将`Server URL` 配置为：`http://172.16.0.100:2222/`，然后点击 `Apply`，其他配置默认即可。

配置完成后点击`Upsource`的图标会登录界面，用自己的用户名/密码登录到服务器就好了。

![41](/blog/images/201708_3/41.png)

登录成功以后，就会在IDEA中显示跟你有关的`Review`信息，可以直接在`News Feed`中针对某个`Comment`进行回复。

![33](/blog/images/201708_3/33.png)

后续可以直接点击`Upsource`按钮切换用户名，切换项目等操作，当需要设置`Edit Path Mapping `时，需要设置该项目的本地代码路径以及远程路径。

![42](/blog/images/201708_3/42.png)

其中`Local root path `设置为现在激活的项目的代码的本地路径，` Upsource remote path `设置为 `/ `就好。

![43](/blog/images/201708_3/43.png)

后续代码审查人员针对你提交的代码有审核操作的时候，你将自动收到来自`Upsource`的提示信息。

![44](/blog/images/201708_3/44.png)

3. 设置邮件提醒

在浏览器中登录Upsource服务器` http://www.wuzguo.com/ `，在右上角选中`Upsource`然后再点击右上角的账号图标选择 `Notifications` 便可以设置。

![48](/blog/images/201708_3/48.png)

![49](/blog/images/201708_3/49.png)

在账户配置中设置开通邮件提醒，当有关于你的代码审查时会邮件通知你。

![46](/blog/images/201708_3/46.png)

然后你将收到类似的邮件。

![47](/blog/images/201708_3/47.png)
