---
title: Shell 入门
date: 2020-06-29 09:25:00
author: Zak
avatar: /blog/images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wuzguo
authorDesc: 一个追求进步的「十八线码农」
categories: 操作系统
tags: 
	- shell
	- linux
	- bash
	- centos
keywords: shell，linux，bash，centos
photos:
	- /blog/images/202007/0.jpg
description: Shell 从入门到精通
---

### 1. 概述

Shell 是一个命令行解析器，它接受应用程序/用户命令，然后调用操作系统的内核。它还是一个功能强大的编程语言，具有易编写，易调试，灵活性强的特点。

![]((/images/202007/2.png)

### 2. 解析器

#### 2.1 Linux提供的Shell解析器

```shell
[root@docker-ecs /]# cat /etc/shells
/bin/sh
/bin/bash
/sbin/nologin
/bin/dash
[logviewer@ip-10-20-1-182 ~]# ll | grep bash
```

#### 2.2 bash和sh的关系

```shell
[root@docker-ecs bin]# ll | grep bash

-rwxr-xr-x. 1 root root 906568 Mar 22  2017 bash
lrwxrwxrwx. 1 root root      4 Jul  4  2018 sh -> bash
```

#### 2.3 Centos的默认解析器

```shell
[root@docker-ecs /]# echo $SHELL
/bin/bash
```

### 3. 第一个脚本

#### 3.1 脚本格式

​	脚本以#!/bin/bash开头，指定解析器

#### 3.2 第一个Shell脚本

```shell
[root@docker-ecs data]# touch hello.sh

[root@docker-ecs data]# vi hello.sh
```

在 hello.sh 中输入如下内容

```shell
#!/bin/bash

echo "hello, world !"
```

#### 3.3 脚本的常用执行方式

##### 3.3.1 采用bash或sh+脚本的相对路径或绝对路径（不用赋予脚本+x权限）

sh+脚本的相对路径

```shell
[root@docker-ecs data]# sh hello.sh

hello, world !
```

sh+脚本的绝对路径

```shell
[root@docker-ecs data]# sh /data/hello.sh 

hello, world !
```

bash+脚本的相对路径

```shell
[root@docker-ecs data]# bash hello.sh 

hello, world !
```

bash+脚本的绝对路径

```shell
[root@docker-ecs data]# bash /data/hello.sh 

hello, world !
```

##### 3.3.2 采用输入脚本的绝对路径或相对路径执行脚本（必须具有可执行权限+x）

首先要赋予 hello.sh 脚本的+x权限

```shell
[root@docker-ecs data]# chmod 777 hello.sh
```

执行脚本

相对路径

```shell
[root@docker-ecs data]# ./hello.sh 

hello, world !
```

绝对路径

```shell
[root@docker-ecs data]# /data/hello.sh 

hello, world !
```

注意：第一种执行方法，本质是bash解析器帮你执行脚本，所以脚本本身不需要执行权限。第二种执行方法，本质是脚本需要自己执行，所以需要执行权限。

#### 3.4 第二个Shell脚本（多命令处理）

需求：在 /data/ 目录下创建一个 cls.txt 并在文件中增加 " I love cls"。

案例实操：

```shell
[root@docker-ecs data]# touch batch.sh

[root@docker-ecs data]# vi batch.sh
```

在 batch.sh 中输入如下内容

```shell
#!/bin/bash

cd /data

touch cls.txt

echo "I love cls" >> cls.txt
```

执行脚本，查看结果

```shell
[root@docker-ecs data]# cat cls.txt 
i love cls
```