---
title: Upsource使用说明之代码审查规则
date: 2017-08-27 14:45:23 
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
	- /blog/images/201708_3/51.png
description: 代码审查工具Upsource使用说明系列文章之代码审查规则及操作说明
---


### 一、代码审查的意义
   代码审查简单的描述就是让其他同事来检查你的代码，其目的在于找到开发初期所忽略的错误，从而提高代码的整体质量，降低风险，减少代码隐藏的一些问题。同时通过同事间的合作可以相互学习并取得进步。代码提交者很有可能从该工作中得到反馈，并意识到可能存在的问题和需要改进的部分；而审查者也可以通过阅读他人代码学到新的东西，并找出适用于自己的工作方案。

### 二、使用Upsource进行代码审查
#### 1. Review审查流程

![51](/blog/images/201708_3/51.png)

由有权限的同事针对某个或者多个代码提交操作发起一次代码审查流程，然后审查人员添加跟这次审查流程相关的`Reviewers`和`Watchers`，这里的 `Authors`、`Reviewers`、`Watchers `都是复数形式表示都有可能是多个人员。

![52](/blog/images/201708_3/52.png)

一次`Review`也可以添加多个`Revisions`(修订，可以理解为一次版本提交)。

![53](/blog/images/201708_3/53.png)

一般情况下将这次`Review`中关系关系很密切的人设置为`Reviewer`，然后这次`Review`需要知道的人设置为`Watcher`，假设某个`Review`是关于某个接口的参数修改，我们可以把接口提供方的调用方都设置为`Reviewer`，然后跟这个接口关系不够密切但是需要知晓的人设置为`Watcher`。各个参与人员的角色没有很明确的划分界限，可以根据实际情况来确定。
Review的参与人员都有权限回复别人提出的意见，`Authors`（代码作者）角色的人在这次`Review`是执行者，负责执行大家讨论的结果。

如果`Authors `在IDEA中完成经过讨论的修改方案后，提交代码的时候可以设置`Review`的ID，表示本次提交是为了解决某个`Reivew`中提出的问题。

![35](/blog/images/201708_3/35.png)

然后在upsource中就可以跟踪到某个相应review的解决结果了。

![36](/blog/images/201708_3/36.png)

当 `Authors `修改完成代码后， `Reviewers `可以 `Accept`修改结果或者`Raise concern `,当所有 `Reviewers `都  `Accept`修改结果的时候，这个Review可以正常关闭，默认情况下 Authors也可以关闭Review，但是可以设置不允许 `Authors `关闭。（系统管理员选择` Edit Project -> Advanced -> 取消 Allow authors to close reviews `的选项）

如果提交的时候忘记忘记勾选 `reivew with upsource` 选项了可以在提交完成后在`Version Control ``Tab`页面右击鼠标选择`upsource `然后设置到对应的`Review`即可。

![37](/blog/images/201708_3/37.png)

![](/blog/images/201708_3/38.png)

也可以直接在`Upsource`页面将对应的修订提交加到这个`Review`中来。

![54](/blog/images/201708_3/54.png)

添加进来后修订的时间线将发生变化。

![55](/blog/images/201708_3/55.png)

当然也可以`Remove`某个版本。

![56](/blog/images/201708_3/56.png)

也可以直接在IDEA中对代码添加评论，或者将评论添加到Review流程中。

![](/blog/images/201708_3/66.png)

或者按下键盘 `ctrl+alt+ 斜杠` （windows下）添加评论。

![](/blog/images/201708_3/67.png)

勾选 `Attach to review` 可以将评论添加到对应的Review中。

![](/blog/images/201708_3/68.png)

### 2. 审查分支

分支审查包括所选分支的所有当前提交修订以及所有将来的修订，将自动附加到审查。`创建方式如下选择某个项目` -> `Branches` -> `选择需要Review的分支` -> `Create branch review`. 其他流程跟上面的一样，不再赘述。

### 3. 评论而不创建Review
每当你在同事的代码中发现一些小问题时，想要给他一个提醒或者开始一个非正式的讨论，你可以留下评论而不创建`Review`。您可以发表评论，并回复他人留下的评论。

![57](/blog/images/201708_3/57.png)

### 4. 添加标签
你可以为任何评论添加标签来标注你认为有用的信息或者分类，也可以创建自己的标签在整个项目中使用。

![58](/blog/images/201708_3/58.png)

### 5. 评论
评论支持MarkDown语法，可以添加各种表情包，添加图表等等。

![59](/blog/images/201708_3/17.png)

### 6. 关于通知
`Upsource`的通知信息一般包含邮件或者`IDEA`中的插件推送，如果你觉得消息泛滥有些消息不需要提醒的时候，可以自定义消息通知。

邮件通知在`Upsource`的账户中设置，根据自己关注的添加适当的触发器。

![60](/blog/images/201708_3/60.png)

![61](/blog/images/201708_3/61.png)

IDEA的插件通知可以在IDEA中打开设置页面，在 `file`-> `settings`-> `搜索“upsource”` ->然后根据自己的实际关注的消息进行设置。

![62](/blog/images/201708_3/62.png)
