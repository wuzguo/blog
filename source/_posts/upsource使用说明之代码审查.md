---
title: Upsource使用说明之代码审查实操
date: 2017-08-28 21:50:01 
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
	- /blog/images/201708_3/1.png
description: 代码审查工具Upsource使用说明系列文章之代码审查实操
---


## Upsource使用说明

### 一、添加项目

创建新项目，管理员账号登录Upsource中在选择Hub，创建一个需要进行代码审核的项目，输入项目名称、Key及描述信息。

![](/blog/images/201708_3/1.png)

添加完成后，可以为该项目添加团队成员，设置项目的基本信息等。

![](/blog/images/201708_3/2.png)

然后进入正题添加 Code Review 服务。选择Add Service 中配置源代码的VCS的类型及路径等信息，配置完成后Upsource便会连接配置的VCS链接获取代码提交记录。这里需要设置获取源代码的URL地址以及具有访问权限的账号。**Key为SSH的私钥。**

![](/blog/images/201708_3/3.png)

服务添加成功后便可点击 `upsource` 链接进行代码审核阶段。

![](/blog/images/201708_3/4.png)

具有代码审查权限的用户可以针对该项目的每一个提交操作创建一个Review。

![](/blog/images/201708_3/5.png)

这里有一个主意的地方就是 upsource 和版本管理系统（gitlab）各自有自己的账户体系，但是他们之间可以关联，只需要在账户设置中配置一下对应关系就好。

![](/blog/images/201708_3/6.png)


### 二、使用说明

1. 创建一个Review，创建Review时，可以为每个修改的文件添加行注释和段落注释，以及评论，同时为这个Review添加 Observer（**可以是作者本人，也可以说跟这个功能相关的人员，如某个接口修改了参数，需要把用到这个接口的人员都添加进来，这样避免修改了参数其他人不知道**）。

![](/blog/images/201708_3/7.png)

添加行级注释。

![](/blog/images/201708_3/8.png)

添加段落注释。

![](/blog/images/201708_3/9.png)

当Reviewer添加审查意见后，对应的开发人员和Observer都能在账户中看到对应的评价信息。**这个Review相关的所有人都可以对这个Review进行回应。也可以@某个人。**

![](/blog/images/201708_3/11.png)

这样就可以精确到某一行代码进行充分讨论，最后得出修改意见。然后代码的作者修改代码，**达到审查者的要求后，审查者可以Accept这个 Review的修改结果，表示接受此次修订，此次审查被视为完成。**也可以 Raise Concern 让作者引起关注。审查者和代码作者都可以关闭 Review，**可以在项目属性设置中代码作者是否有关闭Review的权限**。

![](/blog/images/201708_3/12.png)

相关人员都可以对这个Review 贴上标签。可以标注这个问题是功能性的BUG还是代码的样式问题等。也可以自定义标签在全局项目中使用。

![](/blog/images/201708_3/13.png)

开发人员根据代码审查过程中的讨论结果进行代码修改，然后提交到`CVS`系统，为了跟踪代码审查的执行结果，相关人员可以将提交的`revision` （修订）加入到本次审查流程中。

![](/blog/images/201708_3/14.png)

![](/blog/images/201708_3/16.png)

评论支持`MarkDown`语法，可以添加各种表情包，添加表格等等。

![](/blog/images/201708_3/17.png)



Upsource还支持自定义工作流程中进行自动化，其中包括：

- 可以设置当有人修改项目中的核心文件时自动发起代码审查操作。
- 当有人修改的代码或提交消息指定了设置的问题ID时（issue ID）时，自动将当前修订（Revision）添加到对应的Review流程中。
- 自动分配审核参与者给手动或自动创建的Review。
- 当所有参与评审者接受更改时关闭审查。
- 在审查结束时将所有讨论表决为`Resolved` 状态。
- 当对以前审查的分支机构进行新的提交时，重新审核并恢复分支跟踪。

![](/blog/images/201708_3/63.png)

如图设置了一个当有人修改 AngularEchartsExamplesApplication.java 这个文件时自动发起一个审查流程。

![](/blog/images/201708_3/65.png)
![](/blog/images/201708_3/64.png)

