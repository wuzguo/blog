---
title: 用AXIS2发布WebService的方法
date: 2016-04-30 10:04:10 
author: wúzguó
avatar: /blog/images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wzguo
authorDesc: 一个自强不息，艰苦奋斗的「十八线码农」
categories: 后台
tags: 
	- WebService
	- Axis2
keywords:
photos:
	- /blog/images/201604/1.png
description: Axis2+tomcat7.0 实现webService 服务端发布与客户端的调用
---


##用AXIS2发布WebService的方法
Axis2+tomcat7.0 实现webService 服务端发布与客户端的调用.

第一步：首先要下载开发所需要的jar包

下载：axis2-1.7.1-war.zip

[http://www.apache.org/dyn/closer.lua/axis/axis2/java/core/1.7.1/axis2-1.7.1-war.zip](http://www.apache.org/dyn/closer.lua/axis/axis2/java/core/1.7.1/axis2-1.7.1-war.zip)

下载完后解压至tomcat安装目录下的webapps文件夹下，启动tomcat后，在webapps目录下会生成axis2文件夹。

访问http://localhost:8080/axis2/能看到以下页面表示axis2运行成功。</br>

![](/blog/images/201604/11.png)

1.在IntelliJ IDEA下新建Web Project，工程名：HelloWorld，如下：

![](/blog/images/201604/1.png)

2.新建类HelloService，然后创建写两个方法。

![](/blog/images/201604/2.png)

3.将axis2 WEB-INF目录下的conf、lib、modules、services的文件夹拷贝到HelloWorld的 WEB-INF目录下。

![](/blog/images/201604/3.png)

4.在HelloWorld中services的文件夹创建 HelloWorld/META-INF目录,在该目录下创建services.xml文件，配置信息如下：

![](/blog/images/201604/4.png)

5.在web.xml中妹子一个axis的servlet，拦截客户端请求。

![](/blog/images/201604/5.png)

6.特别注意：要将WEB-INF目录下的lib包引入到工程中，不然的话将不能运行。

![](/blog/images/201604/6.png)

7.配置WEB容器，这里使用Tomcat 7.0.64版本。

![](/blog/images/201604/8.png)

8.配置工程访问路径。

![](/blog/images/201604/9.png)

9.启动tomcat，然后用上图配置的路径在浏览器中访问我们在services.xml中配置的Service。

![](/blog/images/201604/10.png)

10.如果返回上图浏览器所示的XML文件信息，说明webservice服务就发布成功了。




---到此Axis2的WebService服务已成功发布



11.客户端调用
![](/blog/images/201604/12.png)



12.参考文章 [http://www.cnblogs.com/javawebsoa/archive/2013/05/19/3087234.html](http://www.cnblogs.com/javawebsoa/archive/2013/05/19/3087234.html)