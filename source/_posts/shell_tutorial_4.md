---
title: Shell 函数
date: 2020-06-30 10:47:00
author: Zak
avatar: /blog/images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wuzguo
authorDesc: 一个自强不息，艰苦奋斗的「十八线码农」
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


### 1. 系统函数

#### 1.1 basename基本语法

basename [string / pathname] [suffix]    （功能描述：basename命令会删掉所有的前缀包括最后一个（‘/’）字符，然后将字符串显示出来。

选项：

suffix为后缀，如果suffix被指定了，basename会将pathname或string中的suffix去掉。

#### 1.2 案例实操

（1）截取该 /data/hello.sh 路径的文件名称

```shell
[root@docker-ecs data]# basename ./hello.sh 
hello.sh
[root@docker-ecs data]# basename /data/hello.sh 
hello.sh
[root@docker-ecs data]# basename /data/hello.sh .sh
hello
[root@docker-ecs data]# basename /data/hello.sh .xx
hello.sh
```

#### 1.3 dirname基本语法

​    dirname 文件绝对路径    （功能描述：从给定的包含绝对路径的文件名中去除文件名（非目录的部分），然后返回剩下的路径（目录的部分））

#### 1.4 案例实操

（1）获取banzhang.txt文件的路径

```shell
[root@docker-ecs data]# dirname /data/hello.sh 
/data
```

### 2. 自定义函数

#### 2.1 基本语法

```shell
[ function ] funname[()]

{

Action;

[return int;]

}

funname
```

#### 2.2 经验技巧

​    （1）必须在调用函数地方之前，先声明函数，shell脚本是逐行运行。不会像其它语言一样先编译。

​    （2）函数返回值，只能通过$?系统变量获得，可以显示加：return返回，如果不加，将以最后一条命令运行结果，作为返回值。return后跟数值n(0-255)

3．案例实操

​    （1）计算两个输入参数的和

```shell
[root@docker-ecs data]# touch fun.sh

[root@docker-ecs data]# vim fun.sh

#!/bin/bash

function sum()

{

  s=0

  s=$[ $1 + $2 ]

  echo "$s"

}

 
read -p "Please input the number1: " n1;

read -p "Please input the number2: " n2;

sum $n1 $n2;

 
[root@docker-ecs data]# chmod 777 fun.sh

[root@docker-ecs data]# ./fun.sh 
Please input the number1: 2
Please input the number2: 5

7
```

