---
title: Shell 的基本使用
date: 2020-06-29 12:14:00
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

### 1. 变量

#### 1.1 系统变量

##### 1.1.1 常用系统变量

​	$HOME、$PWD、$SHELL、$USER等

##### 1.1.2 案例实操

查看系统变量的值

```shell
[root@docker-ecs data]# echo $HOME
/root
```

显示当前Shell中所有变量：set

```shell
[root@docker-ecs data]# set
BASH=/bin/bash
BASH_ALIASES=()
BASH_ARGC=()
BASH_ARGV=()
......
```

#### 1.2 自定义变量

##### 1.2.1 基本语法

1. 定义变量：变量=值 
2. 撤销变量：unset 变量

3. 声明静态变量：readonly 变量，注意：不能unset

##### 1.2.2 变量定义规则

1. 变量名称可以由字母、数字和下划线组成，但是不能以数字开头，环境变量名建议大写。

2. 等号两侧不能有空格

3. 在bash中，变量默认类型都是字符串类型，无法直接进行数值运算。
4. 变量的值如果有空格，需要使用双引号或单引号括起来。

##### 1.2.3 案例实操

1. 定义变量A

```shell
[root@docker-ecs data]# A=5

[root@docker-ecs data]# echo $A
5
```

2. 给变量A重新赋值

```shell
[root@docker-ecs data]# A=8

[root@docker-ecs data]# echo $A
8
```

3. 撤销变量A

```shell
[root@docker-ecs data]# unset A

[root@docker-ecs data]# echo $A

```

4. 声明静态的变量B=2，不能unset

```shell
[root@docker-ecs data]# readonly B=2

[root@docker-ecs data]# echo $B
2

[root@docker-ecs data]# B=3
-bash: B: readonly variable

[root@docker-ecs data]# echo $B
2

[root@docker-ecs data]# unset B
-bash: unset: B: cannot unset: readonly variable

```

5. 在bash中，变量默认类型都是字符串类型，无法直接进行数值运算

```shell
[root@docker-ecs data]# C=1+2

[root@docker-ecs data]# echo $C

1+2
```

6. 变量的值如果有空格，需要使用双引号或单引号括起来

```shell
[root@docker-ecs data]# D=hello world
-bash: world: command not found

[root@docker-ecs data]# D="hello world"

[root@docker-ecs data]# echo $D
hello world
```

7. 可把变量提升为全局环境变量，可供其他Shell程序使用

export 变量名

```shell
[root@docker-ecs data]$ vim helloworld.sh 
```

在hello.sh文件中增加echo $B

```shell
#!/bin/bash

echo "hello world"
echo $B

[root@docker-ecs data]# ./hellO.sh 
Hello world
```

发现并没有打印输出变量B的值。

```shell
[root@docker-ecs data]# export B

[root@docker-ecs data]# ./hello.sh 
hello world
2
```

#### 1.3 特殊变量：$n

##### 1.3.1 基本语法

​    $n  （功能描述：n为数字，$0代表该脚本名称，$1-$9代表第一到第九个参数，十以上的参数，十以上的参数需要用大括号包含，如${10}）

##### 1.3.2 案例实操

输出该脚本文件名称、输入参数1和输入参数2的值

```shell
[root@docker-ecs data]# touch param.sh 

[root@docker-ecs data]# vim param.sh

#!/bin/bash

echo "$0 $1 $2"

[root@docker-ecs data]# chmod 777 param.sh

[root@docker-ecs data]# ./param.sh 1 2
./param.sh 1 2
```



#### 1.4 特殊变量：$#

##### 1.4.1 基本语法

​    $#  （功能描述：获取所有输入参数个数，常用于循环）。

##### 1.4.2 案例实操

获取输入参数的个数

```shell
[root@docker-ecs data]# vim param.sh

#!/bin/bash
echo "$0 $1  $2"
echo $#

[root@docker-ecs data]# chmod 777 param.sh

[root@docker-ecs data]# ./param.sh 1 2 3
param.sh 1 2 
3
```



#### 1.5 特殊变量：$*、$@

##### 1.5.1 基本语法

​    $*  （功能描述：这个变量代表命令行中所有的参数，$*把所有的参数看成一个整体）

​    $@ （功能描述：这个变量也代表命令行中所有的参数，不过$@把每个参数区分对待）

##### 1.5.2 案例实操

（1）打印输入的所有参数

```shell
[root@docker-ecs data]# vim param.sh

#!/bin/bash

echo "$0 $1  $2"
echo $#
echo $*
echo $@

[root@docker-ecs data]# bash param.sh 1 2 3
param.sh 1  2
3
1 2 3
1 2 3
```


#### 1.6 特殊变量：$？

##### 1.6.1 基本语法

$？ （功能描述：最后一次执行的命令的返回状态。如果这个变量的值为 0，证明上一个命令正确执行；如果这个变量的值为非0（具体是哪个数，由命令自己来决定），则证明上一个命令执行不正确了。）

##### 1.6.2 案例实操

判断 hello.sh 脚本是否正确执行

```shell
[root@docker-ecs data]# ./hello.sh 
hello world

[root@docker-ecs data]# echo $?
0
```



### 2. 运算符

#### 2.1 基本语法

1. “$((运算式))”或“$[运算式]”

2. expr  + , - , \*, /, %  加，减，乘，除，取余

   注意：expr 运算符间要有空格

#### 2.2 案例实操：

1. 计算3+2的值

```shell
[root@docker-ecs data]# expr 1+2
1+2
[root@docker-ecs data]# expr 1 +2
expr: syntax error
[root@docker-ecs data]# expr 1+ 2
expr: syntax error
[root@docker-ecs data]# expr 1 + 2
3
```

2. 计算3-2的值

```shell
[root@docker-ecs data]# expr 3 - 2 
1
```

3. 计算（2+3）X4的值

expr 一步完成计算

```shell
[root@docker-ecs data]# expr `expr 2 + 3` \* 4
20
```

采用$[运算式]方式

```shell
[root@docker-ecs data]# S=$[(2+3)*4]

[root@docker-ecs data]# echo $S
20
[root@docker-ecs data]# expr S = $[(2+3)*5]
0
[root@docker-ecs data]# echo $S
20
```



### 3. 条件判断

#### 3.1 基本语法

[ condition ]（注意condition前后要有空格）

注意：条件非空即为true，[ hello ]返回true，[] 返回false。

#### 3.2 常用判断条件

1. 两个整数之间比较

    = 字符串比较

    -lt 小于（less than）         -le 小于等于（less equal）

    -eq 等于（equal）           -gt 大于（greater than）

    -ge 大于等于（greater equal）  -ne 不等于（Not equal）

2. 按照文件权限进行判断

    -r 有读的权限（read）       -w 有写的权限（write）

    -x 有执行的权限（execute）

3. 按照文件类型进行判断

    -f 文件存在并且是一个常规的文件（file）

    -e 文件存在（existence）    -d 文件存在并是一个目录（directory）

#### 3.3 案例实操

1. 23是否大于等于22

```shell
[root@docker-ecs data]# [ 23 -ge 22 ]

[root@docker-ecs data]# echo $?
0
```

2. hello.sh 是否具有写权限

```shell
[root@docker-ecs data]# [ -w hello.sh ]

[root@docker-ecs data]# echo $?
0
```

3. /data/cls.txt目录中的文件是否存在

```shell
[root@docker-ecs data]# [ -e /data/cls.txt ]
[root@docker-ecs data]# echo $?
0
[root@docker-ecs data]# [ -e /data/clsw.txt ]
[root@docker-ecs data]# echo $?
1
```

4. （多条件判断（&& 表示前一条命令执行成功时，才执行后一条命令，|| 表示上一条命令执行失败后，才执行下一条命令）

```shell
[root@docker-ecs data]# [ condition ] && echo OK || echo notok
OK
[root@docker-ecs data]# [ condition ] && [ ] || echo notok
notok
[root@docker-ecs data]# [ condition ] && [ ] && echo OK || echo notok
notok
```
