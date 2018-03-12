---
title: Angular5踩坑记录之'undefined is not a function'的错误

date: 2018-03-05 16:38:06

author: wúzguó

avatar: /blog/images/avatar.png

authorLink: http://www.wuzguo.com

authorAbout: https://github.com/wzguo

authorDesc: 一个自强不息，艰苦奋斗的「十八线码农」

categories: 前端

tags:
	- Angular

	- Angular5

	- undefined

keywords: Angular，Angular5，undefined

photos:
	- /blog/images/201803/0.jpg

description: 解决'undefined is not a function'的错误
---

最近在使用Angular5+Ant Desigin开发公司后台管理系统，使用的Angular 5.2.3版本，当把Angular版本升级到5.2.7的时候之前运行的很好的项目突然不能运行了，在控制台包了一个`TypeError: undefined is not a function`的错误，如下所示：

![](/blog/images/201803/0.jpg)

苦思冥想不得其解遂百度，找到一个博客：https://segmentfault.com/q/1010000013355994 得到答案，是应为我的懒加载的配置有问题，之前配置的有问题可能是Angular版本升级以后对这块功能进行了升级加强导致不能使用了，解决方案就是检查项目的懒加载配置，我的解决方式如下：

![](/blog/images/201803/1.jpg)

在`app.module.ts`中去掉`AdminModule`的引用，因为`AdminModule`是懒加载的，不需要直接引用进来。

![](/blog/images/201803/2.jpg)

在`routes.module.ts`中去掉 `PagesModule`、`ChartsModule` 、`DashboardModule` 模块，因为这三个模块我也做了懒加载。

**所以解决方案就是：懒加载模块不要直接放在 imports中导入，否则会出现 `TypeError: undefined is not a function` 的错误。**