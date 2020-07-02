---
title: Shell的流程控制
date: 2020-06-29 13:24:00
author: Zak
avatar: /blog/images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wzguo
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

### 1. if 判断

#### 1.1 基本语法

if [ 条件判断式 ];then 

 程序 

fi 

或者 

if [ 条件判断式 ] 

 then 

  程序 

elif [ 条件判断式 ]

​    then

​       程序

else

​    程序

fi

   

注意事项：

1. [ 条件判断式 ]，中括号和条件判断式之间必须有空格
2. if 后要有空格

#### 1.2 案例实操

（1）输入一个数字，如果是1，则输出hello world，如果是2，则输出world hello，如果是其它，什么也不输出。

```shell
[root@docker-ecs data]# touch if.sh

[root@docker-ecs data]# vim if.sh

#!/bin/bash

if [ $1 -eq "1" ]

then

echo "hello world"

elif [ $1 -eq "2" ]

then

echo "world hello"

fi

[root@docker-ecs data]# chmod 777 if.sh 

[root@docker-ecs data]# ./if.sh 2
world hello
[root@docker-ecs data]# ./if.sh 1
hello world
```



### 2. case 语句

#### 2.1 基本语法

case $变量名 in 

 "值1"） 

  如果变量的值等于值1，则执行程序1 

  ;;

 "值2"） 

  如果变量的值等于值2，则执行程序2 

  ;;

 …省略其他分支… 

 *） 

  如果变量的值都不是以上的值，则执行此程序 

  ;;

esac

注意事项：

1)    case行尾必须为单词“in”，每一个模式匹配必须以右括号“）”结束。

2)    双分号“**;;**”表示命令序列结束，相当于java中的break。

3)    最后的“*）”表示默认模式，相当于java中的default。

#### 2.2 案例实操

（1）输入一个数字，如果是1，则输出hellO，如果是2，则输出world，如果是其它，输出hello world。

```shell
[root@docker-ecs data]# touch case.sh

[root@docker-ecs data]# vim case.sh

#!/bin/bash


case $1 in

"1")

echo "hellO"

;;

"2")

echo "world"

;;

*)

echo "hello world"

;;

esac

[root@docker-ecs data]# chmod 777 case.sh

[root@docker-ecs data]# ./case.sh 1
hellO
[root@docker-ecs data]# ./case.sh 2
world
[root@docker-ecs data]# ./case.sh 3
hello world
```



### 3. for 循环

#### 3.1 基本语法1

​    for (( 初始值;循环控制条件;变量变化 )) 

 do 

  程序 

 done

#### 3.2 案例实操

（1）从1加到100

```shell
[root@docker-ecs data]# touch for1.sh

[root@docker-ecs data]# vim for1.sh

#!/bin/bash

s=0

for((i=0;i<=100;i++))

do

s=$[$s+$i]

done

echo $s

 
[root@docker-ecs data]# chmod 777 for1.sh 

[root@docker-ecs data]# ./for1.sh 

"5050"
```



#### 3.3 基本语法2

for 变量 in 值1 值2 值3… 

 do 

  程序 

 done

#### 3.4 案例实操

​    （1）打印所有输入参数

```shell
[root@docker-ecs data]# touch for2.sh

[root@docker-ecs data]# vim for2.sh

#!/bin/bash

#打印数字

for i in $*

do

echo "xiao love $i "

done

[root@docker-ecs data]# chmod 777 for2.sh 

[root@docker-ecs data]# bash for2.sh cls xz bd
xiao love cls
xiao love xz
xiao love bd
```



（2）比较$*和$@区别

（a）$*和$@都表示传递给函数或脚本的所有参数，不被双引号“”包含时，都以$1 $2 …$n的形式输出所有参数。

```shell
[root@docker-ecs data]# touch for.sh  
[root@docker-ecs data]# vim for.sh     
  
#!/bin/bash     
  
for i in $* 
do    

echo "xiao love $i " 

done     

for j in $@  
do        

echo "xiao love $j" 

done     

[root@docker-ecs data]# bash for.sh cls xz bd 
xiao love  cls  
xiao love  xz  
xiao love  bd   

xiao love  cls  
xiao love  xz  
xiao love  bd  
```



（b）当它们被双引号“”包含时，“$*”会将所有的参数作为一个整体，以“$1 $2 …$n”的形式输出所有参数；“$@”会将各个参数分开，以“$1” “$2”…”$n”的形式输出所有参数。

```shell
[root@docker-ecs data]# vim for.sh

#!/bin/bash 

for i in "$*"

# $*中的所有参数看成是一个整体，所以这个for循环只会循环一次 

do 

echo "xiao love $i"

done 

for j in "$@"

# $@中的每个参数都看成是独立的，所以“$@”中有几个参数，就会循环几次 

do 

echo "xiao love $j" 

done

[root@docker-ecs data]# chmod 777 for.sh

[root@docker-ecs data]# bash for.sh cls xz bd
xiao love cls xz bd

xiao love cls
xiao love xz
xiao love bd
```

### 4. while 循环

#### 4.1 基本语法

while [ 条件判断式 ] 

 do 

  程序

 done

#### 4.2 案例实操

从1加到100

```shell
[root@docker-ecs data]# touch while.sh

[root@docker-ecs data]# vim while.sh

#!/bin/bash

s=0
i=1

while [ $i -le 100 ]
do

s=$[$s+$i]
i=$[$i+1]

done

echo $s

[root@docker-ecs data]# chmod 777 while.sh 

[root@docker-ecs data]# ./while.sh 
5050
```



### 5. read读取控制台输入

#### 5.1 基本语法

read (选项) (参数)

选项：

-p：指定读取值时的提示符；

-t：指定读取值时等待的时间（秒）。

参数

变量：指定读取值的变量名

#### 5.2 案例实操

提示7秒内，读取控制台输入的名称

```shell
[root@docker-ecs data]# touch read.sh

[root@docker-ecs data]# vim read.sh

#!/bin/bash

read -t 7 -p "enter your name in 7 seconds " NAME

echo $NAME

[root@docker-ecs data]# ./read.sh 
enter your name in 7 seconds hello
hello
```

