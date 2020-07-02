---
title: Shell 工具
date: 2020-06-30 17:51:00
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


### 1. cut

cut的工作就是"剪"，具体的说就是在文件中负责剪切数据用的。cut 命令从文件的每一行剪切字节、字符和字段并将这些字节、字符和字段输出。

#### 1.1 基本用法

cut [选项参数]  filename

说明：默认分隔符是制表符

#### 1.2 选项参数说明

| 选项参数 | 功能                         |
| -------- | ---------------------------- |
| -f       | 列号，提取第几列             |
| -d       | 分隔符，按照指定分隔符分割列 |
| -c       | 指定具体的字符               |

#### 1.3 案例实操

1. 数据准备

```shell
[root@docker-ecs data]# touch cut.txt

[root@docker-ecs data]# vim cut.txt

dong shen

guan zhen

wo wo

lai lai

le le
```

2. 切割 cut.txt 第一列

```shell
[root@docker-ecs data]# cut -d " " -f 1 cut.txt 
dong

guan

wo

lai

le
```

3. 切割cut.txt第二、三列

```shell
[root@docker-ecs data]# cut -d " " -f 2,3 cut.txt 

shen

zhen

wo

lai

le
```

4. 在 cut.txt 文件中切割出 guan

```shell
[root@docker-ecs data]# cat cut.txt | grep "guan" | cut -d " " -f 1
guan
```

5. 选取系统PATH变量值，第2个“：”开始后的所有路径：

```shell
[root@docker-ecs data]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/java/jdk1.8.0_192/bin:/usr/java/jdk1.8.0_192/jre/bin:/root/bin
 
[root@docker-ecs data]# echo $PATH | cut -d: -f 2-
/usr/local/bin:/usr/sbin:/usr/bin:/usr/java/jdk1.8.0_192/bin:/usr/java/jdk1.8.0_192/jre/bin:/root/bin
```



6. 切割 ifconfig 后打印的 IP地址

```shell
[root@docker-ecs data]# ifconfig eth0 | grep "inet" | cut -d: -f 2 | cut -d " " -f 1
192.168.1.102
```



### 2. sed

sed是一种流编辑器，它一次处理一行内容。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”，接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。文件内容并没有改变，除非你使用重定向存储输出。

#### 2.1 基本用法

sed  [选项参数]  'command' filename

#### 2.2 选项参数说明

| 选项参数 | 功能                                  |
| -------- | ------------------------------------- |
| -e       | 直接在指令列模式上进行sed的动作编辑。 |
| -i       | 直接编辑文件                          |

#### 2.3  命令功能描述

| 命令 | 功能描述                              |
| ---- | ------------------------------------- |
| a    | 新增，a的后面可以接字串，在下一行出现 |
| d    | 删除                                  |
| s    | 查找并替换                            |

#### 2.4 案例实操

1. 数据准备

```shell
[root@docker-ecs data]# touch sed.txt

[root@docker-ecs data]# vim sed.txt

dong shen

guan zhen

wo wo

lai lai

le le
```



2. 将“mei nv” 这个单词插入到 sed.txt 第二行下，打印。

```shell
[root@docker-ecs data]# sed '2a mei nv' sed.txt 

dong shen

guan zhen

mei nv

wo wo

lai lai

le le

[root@docker-ecs data]# cat sed.txt 

dong shen

guan zhen

wo wo

lai lai

le le
```

注意：文件并没有改变



3. 删除sed.txt文件所有包含wo的行

```shell
[root@docker-ecs data]# sed '/wo/d' sed.txt

dong shen

guan zhen


lai lai

le le
```



4. 将sed.txt文件中wo替换为ni

```shell
[root@docker-ecs data]# sed 's/wo/ni/g' sed.txt 

dong shen

guan zhen

ni ni

lai lai

le le
```

注意：‘g’ 表示global，全部替换



5. 将sed.txt文件中的第二行删除并将wo替换为ni

```shell
[root@docker-ecs data]# sed -e '2d' -e 's/wo/ni/g' sed.txt 
dong shen
guan zhen

ni ni

lai lai

le le
```



### 3. awk

一个强大的文本分析工具，把文件逐行的读入，以空格为默认分隔符将每行切片，切开的部分再进行分析处理。

#### 3.1 基本用法

awk [选项参数] ' pattern1{action1} pattern2{action2}...' filename

pattern：表示AWK在数据中查找的内容，就是匹配模式

action：在找到匹配内容时所执行的一系列命令

#### 3.2 选项参数说明

| 选项参数 | 功能                 |
| -------- | -------------------- |
| -F       | 指定输入文件折分隔符 |
| -v       | 赋值一个用户定义变量 |

#### 3.3 案例实操

1. 数据准备

```shell
[root@docker-ecs data]# sudo cp /etc/passwd ./
```



2. 搜索passwd文件以root关键字开头的所有行，并输出该行的第7列。

```shell
[root@docker-ecs data]# awk -F: '/^root/{print $7}' passwd 

/bin/bash
```



3. 搜索passwd文件以root关键字开头的所有行，并输出该行的第1列和第7列，中间以“，”号分割。

```shell
[root@docker-ecs data]# awk -F: '/^root/{print $1","$7}' passwd 

root,/bin/bash
```



注意：只有匹配了pattern的行才会执行action

4. 只显示 /etc/passwd 的第一列和第七列，以逗号分割，且在所有行前面添加列名user，shell在最后一行添加"hello,/bin/word"。

```shell
[root@docker-ecs data]# awk -F : 'BEGIN{print "user, shell"} {print $1","$7} END{print "hello,/bin/word"}' passwd
user, shell
root,/bin/bash
bin,/sbin/nologin
......
hello，/bin/word
```



注意：BEGIN 在所有数据读取行之前执行；END 在所有数据执行之后执行。

5. 将passwd文件中的用户id增加数值1并输出

```shell
[root@docker-ecs data]# awk -v i=1 -F: '{print $3+i}' passwd
1
2
3
4
5
......
```

#### 3.4 awk的内置变量

| 变量     | 说明                                   |
| -------- | -------------------------------------- |
| FILENAME | 文件名                                 |
| NR       | 已读的记录数                           |
| NF       | 浏览记录的域的个数（切割后，列的个数） |

#### 3.5 案例实操

1. 统计 passwd 文件名，每行的行号，每行的列数

```shell
[root@docker-ecs data]# awk -F: '{print "filename:" FILENAME ", linenum:" NR ",columns:" NF}' passwd
filename:passwd, linenum:1,columns:7
filename:passwd, linenum:2,columns:7
filename:passwd, linenum:3,columns:7
filename:passwd, linenum:4,columns:7
......
```

2. 切割IP

```shell
[root@docker-ecs data]# ifconfig eth0 | grep "inet" | awk -F: '{print $2}' | awk -F " " '{print $1}' 

192.168.1.102
```

3. 查询sed.txt中空行所在的行号

```shell
[root@docker-ecs data]# awk '/^$/{print NR}' sed.txt 
2
4
6
8
```

### 4. sort

sort命令是在Linux里非常有用，它将文件进行排序，并将排序结果标准输出。

#### 4.1 基本语法

sort (选项) (参数)

| 选项 | 说明                     |
| ---- | ------------------------ |
| -n   | 依照数值的大小排序       |
| -r   | 以相反的顺序来排序       |
| -t   | 设置排序时所用的分隔字符 |
| -k   | 指定需要排序的列         |

参数：指定待排序的文件列表

#### 4.2 案例实操

1. 数据准备

```shell
[root@docker-ecs data]# touch sort.sh

[root@docker-ecs data]# vim sort.sh 

bb:40:5.4

bd:20:4.2

xz:50:2.3

cls:10:3.5

ss:30:1.6
```

2. 按照“：”分割后的第三列倒序排序。

```shell
[root@docker-ecs data]# sort -t : -nrk 3 sort.sh 
bb:40:5.4
bd:20:4.2
cls:10:3.5
xz:50:2.3
ss:30:1.6
```

