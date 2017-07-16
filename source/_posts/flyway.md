---
title: Flyway的基本使用
date: 2016-05-26 09:48:25 
author: wúzguó
avatar: /blog/images/avatar.png
authorLink: https://wzguo.github.io
authorAbout: https://github.com/wzguo
authorDesc: 一个自强不息，艰苦奋斗的「十八线码农」
categories: 数据库
tags: 
	- Flyway
keywords: Flyway
photos:
	- /blog/images/201605/11.png
description: 应用、管理并跟踪数据库变更的数据库版本管理工具
---


1. Flyway 是独立于数据库的应用、管理并跟踪数据库变更的数据库版本管理工具。

2. Flyway 的项目主页是 [http://flywaydb.org/](http://flywaydb.org/)

3. Flyway配置成Maven插件，如下图：
![](/blog/images/201605/11.png)

4. 以下是我测试时配置的POM.xml
> 	    ....
> 	    <build>
> 		    <finalName>flyway</finalName>
> 		    <plugins>
> 			    <plugin>
> 				    <groupId>org.flywaydb</groupId>
> 				    <artifactId>flyway-maven-plugin</artifactId>
> 				    <version>${flywaydb.version}</version>
> 				    <dependencies>
> 					    <dependency>
> 						    <groupId>mysql</groupId>
> 						    <artifactId>mysql-connector-java</artifactId>
> 						    <version>${mysql.jdbc.version}</version>
> 					    </dependency>
> 				    </dependencies>
> 				    <configuration>
> 					    <driver>com.mysql.jdbc.Driver</driver>
> 					    <url>jdbc:mysql://localhost:3306/flyway?useUnicode=true&amp;characterEncoding=utf-8</url>
> 					    <user>root</user>
> 					    <password>root</password>
> 					    
> 					    <!-- 设置接受flyway进行版本管理的数据库，多个数据库以逗号分隔 -->
> 					    <schemas>flyway</schemas>
> 					    <!-- 设置存放flyway metadata数据的表名 -->
> 					    <table>schema_version</table>
> 					    <!-- 设置flyway扫描sql升级脚本、java升级脚本的目录路径或包路径 -->
> 					    <locations>
> 					    <location>db/migration</location>
> 					    </locations>
> 					    <!-- 设置sql脚本文件的编码 -->
> 					    <encoding>UTF-8</encoding>
> 				    </configuration>
> 			    </plugin>
> 		    </plugins>
> 	    </build>
> 		....

5. Flyway常用的命令含义
<table><tr><th>命令</th><th>含义</th></tr><tr><td>migrate</td><td>升级数据库</td></tr><tr><td>clean</td><td>删除所有配置的schemas的对象</td></tr><tr><td>info</td><td>打印所有升级的明细和状态信息</td></tr><tr><td>validate</td><td>验证指定路径下（ClassPath）升级配置的正确性</td></tr><tr><td>baseline</td><td>基于当前数据库版本，跳过所有早于当前版本的升级</td></tr><tr><td>repair</td><td>修复数据库中的所有表</td></tr></table>