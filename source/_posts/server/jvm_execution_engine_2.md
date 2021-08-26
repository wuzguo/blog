---
title: JVM执行引擎（二）
date: 2017-06-23 20:00:25 
author: Zak
avatar: /blog/images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wuzguo
authorDesc: 一个追求进步的「十八线码农」
categories: 后台
tags: 
- JVM
- 虚拟机
- 执行引擎
keywords: JVM，Java虚拟机，执行引擎
photos:
- /blog/images/201706/1.png
description: Java虚拟机执行引擎的介绍
---

### 三、基于栈的字节码指令集执行过程

#### 运行时栈帧结构

栈帧是用于支持虚拟机进行方法调用和方法执行的数据结构，它是虚拟机运行时数据区中的虚拟机栈的栈元素。栈帧存储了方法的局部变量表，操作数栈、动态连接和方法返回地址等信息。**每一个方法从调用到开始至执行完成的过程都对应着一个栈帧在虚拟机里面从入栈到出栈的过程。每一个栈帧都包括了局部变量表、操作数栈、动态连接、方法返回地址和一些额外的附加信息。**

![](/blog/images/201706/2.png)

1. 局部变量表

 局部变量表（`Local Variable Table`）是一组变量值存储空间，用于存放方法参数和方法内部定义的局部变量。在Java程序编译为Class文件时，就在方法的Code属性的`max_locals`数据项中确定了该方法所需要分配的局部变量表的最大容量。局部变量表的容量以变量槽（`Variable Slot`）为最小单位，一个槽可以存放一个32位以内的数据类型（boolean、byte、char、short、int、float、reference或returnAddress类型）。

 **在执行方法时，虚拟机是使用局部变量表完成参数值到参数变量列表的传递过程的，如果执行的是实例方法（非static的方法），那么局部变量的表中第0位索引的Slot默认是用于传递方法所属对象实例的引用**，在方法中可以通过关键字“this”来访问到这个隐含的参数，其余参数安装参数表的顺序排列，占用从1开始的局部变量Slot，参数表分配完毕后，再根据方法体内部定义的变量顺序和作用域分配其余的Slot。

 ![](/blog/images/201706/3.png)
 ​				

 ​				**(上图调用的是实例方法，下图调用的是静态方法)**

 ![](/blog/images/201706/4.png)

2. 操作数栈

 操作数栈（`Operand Stack`）也叫操作栈，它是一个后入先出的栈（`Last In First Out LIFO`）。同局部变量表一样，操作数栈的最大深度也在编译的时候写入到了Codes属性的`max_stacks`数据项中，操作数栈的每一个元素可以是任意Java数据类型，包括long和double。32位数据类型所占的栈容量为1,64位数据类型所占的栈容量为2。在执行方法的时候，操作数栈的深度都不会超过在max_stacks数据项设定的最大值。

3. 动态连接

 每一个栈帧都包含一个指向运行时常量池中该栈帧所属的引用，持有这个引用是为了支持方法调用过程中的动态连接（`Dynamic Linking`）。Class文件的常量池中存在大量的符号引用，字节码中的方法调用指令就以常量池中指向方法的符号引用作为参数，这些符号引用一部分会在类加载阶段或者第一次使用的时候就转换为直接引用，这种转化称为静态解析，另一部分将在每一次运行期间转化为直接引用，这部分称为动态连接。

4. 方法返回地址

 当方法开始执行后，只有两种方式可以退出这个方法：

 - 执行引擎遇到任意一个方法返回的字节码指令，这个时候可能有返回值传递给上层调用者。是否有返回值和返回值类型将根据遇到何种方法返回指令来决定，这种退出的方式称为正常完成出口。
 - 在方法执行过程中遇到了异常，并且这种异常没有在方法体内得到处理就会导致方法退出，这退出方式称为异常完成出口。
 - 无论采用何种退出方式，在方法退出之后，都需要返回到方法被调用的位置，程序才能继续执行。方法退出的过程实际上就等同于吧当前栈帧出栈，因此退出时可能执行的操作有：**恢复上层方法的局部变量表和操作数栈，把返回值（如果有的话）压入调用者栈帧的操作数栈，调用PC计数器的值以指向方法调用指令后面的指令等。**

5. 附加信息

 虚拟机规范允许具体的虚拟机实现增加一个规范里没有的描述的信息到栈帧之中，例如与调试相关的信息，这部分完全取决于具体的虚拟机实现。


#### 基于栈的解释器执行过程

   Java编译器输出的指令流，基本上是一种基于栈的指令集架构（`Instruction Set Architecture, ISA`）,指令流中的指令大部分都是零地址指令，它们依赖操作数栈进行工作。与之相对的另一套常用的指令集架构是基于寄存器的指令集，最典型的就是`x86`的二地址指令集（现在主流PC机中直接支持的指令集）。

   以下举例演示基于栈的解释器执行过程：

   ```java
   public class CallFunc {

       public static void main(String[] args) {
           int a = 10;
           int b = 20;
           int c = 30;
           int d = calc(a, b, c);
           System.out.println("d: " + d);
       }

       public static int calc(int a, int b, int c) {
           return a + b - c;
       }
   }
   ```

   通过 `javap -verbose CallFunc` 命令输出Class文件字节码指令：

   ```java
   ......

   public static void main(java.lang.String[]);
   descriptor: ([Ljava/lang/String;)V
   flags: ACC_PUBLIC, ACC_STATIC
   Code:
     stack=3, locals=5, args_size=1
        0: bipush        10
        2: istore_1
        3: bipush        20
        5: istore_2
        6: bipush        30
        8: istore_3
        9: iload_1
       10: iload_2
       11: iload_3
       12: invokestatic  #2                  // Method calc:(III)I
       15: istore        4
       17: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
       20: new           #4                  // class java/lang/StringBuilder
       23: dup							  // 复制栈顶数值，并且复制值进栈
       24: invokespecial #5                  // Method java/lang/StringBuilder."<init>":()V
       27: ldc           #6                  // String d:    // 将int、float或String型常量值从常量池中推送至栈顶
       29: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
       32: iload         4
       34: invokevirtual #8                  // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
       37: invokevirtual #9                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
       40: invokevirtual #10                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
       43: return
     LineNumberTable:
       line 24: 0
       line 25: 3
       line 26: 6
       line 27: 9
       line 28: 17
       line 29: 43
     LocalVariableTable:
       Start  Length  Slot  Name   Signature
           0      44     0  args   [Ljava/lang/String;
           3      41     1     a   I
           6      38     2     b   I
           9      35     3     c   I
          17      27     4     d   I
          
   public static int calc(int, int, int);
   descriptor: (III)I
   flags: ACC_PUBLIC, ACC_STATIC
   Code:
     stack=2, locals=3, args_size=3
        0: iload_0
        1: iload_1
        2: iadd
        3: iload_2
        4: isub
        5: ireturn
     LineNumberTable:
       line 32: 0
     LocalVariableTable:
       Start  Length  Slot  Name   Signature
           0       6     0     a   I
           0       6     1     b   I
           0       6     2     c   I
          
   ......
   ```

根据字节码可以看出，`main`方法需要深度为3的操作数栈（Stack=3）和5个Slot的局部变量空间（Locals=5）。下面使用图片来描述上面的字节码代码执行过程中的代码、操作数栈和局部变量表的变化情况。

1. 执行偏移地址为0的指令的情况

![](/blog/images/201706/5.png)

2. 上图展示了执行偏移地址为0的指令的情况，bipush指令的作用是将单字节的整型常量值（-128~127）推入操作数栈顶，后跟一个参数，指明推送的常量值，这里是10。

执行偏移地址为1的指令的情况

![](/blog/images/201706/6.png)

3. 上图则是执行偏移地址为1的指令，istore_1指令的作用是将操作数栈顶的整型值出栈并存放到第1个局部变量Slot中。后面四条指令（3、5、6、8）都是做同样的事情，也就是在对应代码中把变量a、b、c赋值为10、20、30。后面四条指令的图就不重复画了。
执行偏移地址为9的指令的情况

![](/blog/images/201706/7.png)

4. 执行完偏移地址为9、10、11的指令后

![](/blog/images/201706/8.png)

5. 执行偏移地址为12的指令 `invokestatic  #2` 其中 `#2  =  // com/github/wuzguo/CallFunc.calc:(III)I`，所以是调用static的`calc`方法。前面已经说过每次方法调用都会有新的栈帧产生，所以线程的栈帧结构如下：

![](/blog/images/201706/9.png)

6. 根据字节码可以看出，`calc`方法需要深度为2的操作数栈（Stack=2）和3个Slot的局部变量空间（Locals=3）。下面使用图片来描述上面的字节码代码执行过程中的代码、操作数栈和局部变量表的变化情况。

![](/blog/images/201706/10.png)

7. 执行偏移地址为0和1的指令将局部变量表对应index的值载入操作数栈中，然后执行偏移地址为2的指令，将相加的结果再次存入操作数栈中，操作数栈和局部变量表的变化情况如下：

![](/blog/images/201706/11.png)

8. 然后执行偏移地址为3、4的指令，将局部变量表中`index==2`的值载入操作数栈然后做减法操作，结束后栈帧的变化情况如下：

![](/blog/images/201706/12.png)

9. 接着执行 5: `ireturn`指令，退出当前栈帧，栈帧被弹出当前线程栈，然后接着执行main方法的偏移 地址为15的指令，将`calc`方法运算结果保存到main方法栈帧的`index==4`的局部变量表，`mian`方法栈帧的变化情况如下：

![](/blog/images/201706/13.png)

10. 然后按类似的方式执行`main`方法栈帧中偏移地址17至43的指令，将运算结果打印至控制台，然后退出程序。

### 四、基于寄存器的指令集执行过程

#### 寄存器

CPU中临时存数据的结构。一个IA32的CPU包含一组8个存储32位值得寄存器，这些寄存器用来存储整数数据和指针。在多数情况下前6个寄存器可以看成通用寄存器，对他们的使用没有限制。最后两个寄存器（%ebp和%esp）保存着指向程序栈中重要位置的指针。只有根据栈管理的标准惯例才能修改这两个寄存器的值。

![](/blog/images/201706/14.png)

![](/blog/images/201706/15.png)

以下以简单演示的C语言函数调用过程来演示基于寄存器的指令执行过程

 ```c
#include <stdio.h>
int calc( int *xp, int *yp) 
{
	int x = *xp;
	int y = *yp;
	return x+y;
}

int main(int argc, char *argv[])
{
	int x = 10;
	int y = 20;
	int sum = calc(&x, &y);
	printf("sum is %d\n", sum);
	return sum;
}
 ```
通过GCC运行编译器 ` gcc -m32 -O0 -S hello.c` 产生汇编代码（`AT&T`）（**汇编代码非常接近于机器代码，并非机器代码**），如下：
 ```basic
	.file	"hello.c"
	.text
	.globl	_calc
	.def	_calc;	.scl	2;	.type	32;	.endef
_calc:
0	pushl	%ebp
1	movl	%esp, %ebp
2	subl	$16, %esp
3	movl	8(%ebp), %eax
4	movl	(%eax), %eax
5	movl	%eax, -4(%ebp)
6	movl	12(%ebp), %eax
7	movl	(%eax), %eax
8	movl	%eax, -8(%ebp)
9	movl	-4(%ebp), %edx
10	movl	-8(%ebp), %eax
11	addl	%edx, %eax
12	leave
13	ret
	.def	___main;	.scl	2;	.type	32;	.endef
	.section .rdata,"dr"
LC0:
	.ascii "sum is %d\12\0"
	.text
	.globl	_main
	.def	_main;	.scl	2;	.type	32;	.endef
_main:
0	pushl	%ebp
1	movl	%esp, %ebp
2	andl	$-16, %esp//按位&,内存地址对齐(0xfffffff0)
3	subl	$32, %esp//x86编程指导方针确定任何函数所占用栈空间为16字节的整数倍
4	call	___main//GNU标准库的子程序，作用是初始化gcc所需的内容
5	movl	$10, 24(%esp)
6	movl	$20, 20(%esp)
7	leal	20(%esp), %eax
8	movl	%eax, 4(%esp)
9	leal	24(%esp), %eax
10	movl	%eax, (%esp)
11	call	_calc
12	movl	%eax, 28(%esp)
13	movl	28(%esp), %eax
14	movl	%eax, 4(%esp)
15	movl	$LC0, (%esp)
16	call	_printf
17	movl	28(%esp), %eax
18	leave
19	ret
	.ident	"GCC: (GNU) 7.1.0"
	.def	_printf;	.scl	2;	.type	32;	.endef
 ```

#### 图解CPU执行过程

1. 初始状态假设`ebp`寄存器执行内存地址的512处，`esp`寄存器指向内存地址的544处。

![](/blog/images/201706/16.png)

2. 之所以将`ebp`压栈是为了保存上一个栈帧的起始位置，当执行完序号（0、1）的指令后，栈中的结构如下：

![](/blog/images/201706/17.png)

3. 然后序号（2、3、4）操作对其内存，并为当前栈帧分配32个字节的空间，操作结束后栈结构如下：

  ![](/blog/images/201706/18.png)

4. 序号为（5、6）的指令将立即数10和20压入寄存器`esp`指向的位置偏移24字节和20字节的地方，leal指令是指将寄存器的值加上立即数然后存放到对应的寄存器，如：

  `leal 20(%esp), %eax` 是将寄存器`esp`指向的内存地址加上20然后取值，然后存放在寄存器`eax`中，所以执行完序号（5、6、7、8、9、10）后栈的结构如下：

  ![](/blog/images/201706/19.png)

5. call指令的效果是将返回地址入栈（`push`），并跳转到被调用函数的起始处。返回地址是在程序中紧跟在call后面的那条指令的地址，这样当被调用函数返回时，执行会从此处继续，ret指令从栈中弹出（`pop`）地址，并跳转到这个位置。所以 call _calc 就是调用calc函数并将函数地址入栈，执行完成后结果如下：

  ![](/blog/images/201706/20.png)

6. 然后是执行calc函数序号为（0、1、2）指令将上一个函数（`mian`）的栈帧开始位置入栈并分配内存地址，操作完成后对应的栈帧结构如下：
  ![](/blog/images/201706/21.png)

7. 序号为（3、4、5、6、7、8）的操作就是通过栈帧中存放的内存地址，获取变量10和20。

  操作`movl 8(%ebp), %eax` 表示将`ebp`指向的内存地址加8然后取值（484）然后放入`eax`寄存器。

  操作`movl (%eax), %eax` 表示将`eax`指向的内存地址的值（20）然后放入`eax`寄存器。操作完成后结构如下：

  ![](/blog/images/201706/22.png)

8. 操作（9、10、11）将10和20相加，并将结果存放在`eax`中。然后执行`leave`（为返回操作准备栈）指令和ret（从调用函数返回）指令。然后返回到`main`方法序号为12的指令，此时`esp`寄存器地址还原指向464，结构如下：

  ![](/blog/images/201706/23.png)

9. 后续操作类似，不再详述。