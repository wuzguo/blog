---
title: Angular5踩坑记录之编译出现'JavaScript heap out of memory'的错误

date: 2018-03-05 17:40:42

author: Zak

avatar: /blog/images/avatar.png

authorLink: http://www.wuzguo.com

authorAbout: https://github.com/wzguo

authorDesc: 一个自强不息，艰苦奋斗的「十八线码农」

categories: 前端

tags:
	- Angular

	- Angular5

	- heap out of memory

keywords: Angular，Angular5，heap out of memory

photos:
	- /blog/images/avatar.png

description: 解决'CALL_AND_RETRY_LAST Allocation failed - JavaScript heap out of memory'的错误
---

在`Angular5`项目中执行 `'ng build --prod'` 编译打包时出现以下错误：

![](/blog/images/201803/4.jpg)

解决方案：

 扩大node的heap空间，设置方式如下：

```js
node --max_old_space_size=8192 node_modules/@angular/cli/bin/ng build --prod
```

可以直接在项目的package.json中定义命令：

```
"prod": "node --max_old_space_size=8192 node_modules/@angular/cli/bin/ng build --prod --build-optimizer=true --vendor-chunk=true"
```

然后打包的时候执行 `npm run prod` 命令即可。