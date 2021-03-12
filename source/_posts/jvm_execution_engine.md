---
title: JVM执行引擎
date: 2017-06-23 20:00:25 
author: Zak
avatar: /blog/images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wuzguo
authorDesc: 一个自强不息，艰苦奋斗的「十八线码农」
categories: 操作系统
tags: 
	- JVM
	- 虚拟机
	- 执行引擎
keywords: JVM，Java虚拟机，执行引擎
photos:
	- /blog/images/201706/1.png
description: Java虚拟机执行引擎的介绍
---

#### 一、JVM的结构
   执行引擎是Java虚拟机（`JVM`）最核心的组成部分之一。执行引擎在Class文件的时候可能会有解释执行（通过解释器执行）和编译执行（通过及时编译器产生本地代码执行）两种选择，也可以二者兼备，甚至还可能会包含几个不同级别的编译器执行引擎，但是从外观上看所有的执行引擎都是一致的：**输入的是字节码，处理过程是字节码解析的等效过程，输出的是执行结果。**

![](/blog/images/201706/1.png)

`Java虚拟机 = 类加载器（classloader） + 执行引擎（execution engine） + 运行时数据区域（runtime data area）`

#### 二、机器级代码

**指令体系结构**（`Instruction set architecture, ISA`）是计算机系统对机器级程序的行为的抽象，它是指挥机器工作的指示和命令的集合，大多数ISA（如 `IA32`、`X86-64`）将程序的行为描述成**好像每条指令是按顺序执行的**，但是处理器的硬件远比描述的精细复杂，它们可以并发的执行许多指令，但是可以采取措施保证整体行为与ISA指定的顺序执行完全一致。执行程序的过程就是计算机的工作过程。指令通常包括两方面的内容：**操作码和操作数，操作码决定要完成的操作，操作数指参加运算的数据及其所在的单元地址。**在计算机中，操作要求和操作数地址都由二进制数码表示，分别称作操作码和地址码，**整条指令以二进制编码的形式存放在存储器中**。

**指令集是CPU(或虚拟机)支持的指令的集合，指令集种类和多少与具体的机型有关**：

1. CPU指令集

   存储在CPU内部，对CPU运算进行指导和优化的硬程序。拥有这些指令集，CPU就可以更高效地运行。Intel主要有`x86`，`EM64T`，`MMX`，`SSE`，`SSE2`，`SSE3`，`VMX`等指令集。AMD主要是`x86`，`x86-64`，`3D-Now!`指令集。

   - 指令集的分类：

     - 精简指令集，即RISC指令集`Reduced Instruction Set Computer`：

       这种指令集的特点是指令数目少，每条指令都采用标准字长、执行时间短、中央处理器的实现细节对于机器级程序是可见的。

     - 复杂指令集，即CISC指令集`Complex Instruction Set Computer`：

       在CISC微处理器中，程序的各条指令是按顺序串行执行的，每条指令中的各个操作也是按顺序串行执行的。顺序执行的优点是控制简单，但计算机各部分的利用率不高，执行速度慢。

     通俗的理解，RICS指令集是针对CISC指令集中的一些常用指令进行优化设计，放弃了一些复杂的指令，对于复杂的功能，需要通过组合指令来完成。自然，两者的使用场合不一样，对于复杂的系统，CISC更合适，否则，RICS更合适，且低功耗。

   - 常见指令（**后续说的指令都指IA32体系指令**）：

     **大多数指令有一个或多个操作数（Operand）,指示出执行一个操作中药引用的源数据值，以及放置结果的位置。**操作数可以被分为以下三种类型：

     - 立即数（immediate），也就是常数值。`ATT(AT&T)`格式的汇编代码中，立即数书写格式为：**‘$’ + 标准C表示法表示的整数。**

     - 寄存器（register），它表示某个寄存器的内容，对双字操作来说，可以是8个32位寄存器中的一个（如：`%eax`）,对字操作来说，可以是8个16位寄存器中的一个（如：`%ax`）,或者对字节操作来说，可以是8个单字节寄存器元素总的一个（如：`%al`）,**用E$a$来表示任意寄存器，用引用R[E$a$]来表示它的值。**

     - 存储器（memory）引用，它会根据计算出来的地址（通常称为有效地址）访问某个存储器位置，**可以把存储器看成一个很大的字节数组，用符号M$b$[Addr]表示对存储在存储器中从地址Addr开始的b个字节值得引用。**

       |  类型  | 格式                   | 操作数值                         | 名称         |
       | :--: | -------------------- | ---------------------------- | ---------- |
       | 立即数  | $*Imm*               | *Imm*                        | 立即数寻址      |
       | 寄存器  | E$a$                 | R[E$a$]                      | 寄存器寻址      |
       | 存储器  | *Imm*                | M[*Imm*]                     | 绝对寻址       |
       | 存储器  | (E$a$)               | M[R[E$a$]]                   | 间接寻址       |
       | 存储器  | *Imm*(E$b$)          | M[*Imm*+R[E$b$]]             | （基址+偏移量）寻址 |
       | 存储器  | (E$b$,E$i$)          | M[R[Eb]+R[E$i$]]             | 变址寻址       |
       | 存储器  | *Imm*(E$b$,E$i$)     | M[*Imm*+R[E$b$]+R[E$i$]]     | 变址寻址       |
       | 存储器  | (,E$i$,$s$)          | M[R[E$i$]*$s$]               | 比例变址寻址     |
       | 存储器  | *Imm*(,E$i$,$s$)     | M[*Imm*+R[E$i$]*$s$]         | 比例变址寻址     |
       | 存储器  | (E$b$,E$i$,$s$)      | M[R[E$b$]+R[E$i$]*$s$]       | 比例变址寻址     |
       | 存储器  | *Imm*(E$b$,E$i$,$s$) | M[*Imm*+R[E$b$]+R[E$i$]*$s$] | 比例变址寻      |
       |      |                      |                              |            |

     - 数据传送指令

       将数据从一个位置复制到另一个位置。操作数表示的通用性使得一条简单的数据传送指令能够完成在许多机器中要好几条指令才完成的操作。

       |     指令     | 效果                                       | 描述        |
       | :--------: | :--------------------------------------- | --------- |
       | MOV  S, D  | D  <-- S                                 | 传送        |
       |    movb    | 传送字节                                     |           |
       |    movw    | 传送字                                      |           |
       |    movl    | 传送双字                                     |           |
       | MOVS  S, D | D <--- 符号扩展（S）                           | 传送符号扩展的字节 |
       |   movsbw   | 将做了符号扩展的字节传送到字                           |           |
       |   movsbl   | 将做了符号扩展的字节传送到双字                          |           |
       |   movwl    | 将做了符号扩展的字传送到双字                           |           |
       | MOVZ S, D  | D <---零扩展（S）                             | 传送零扩展的字节  |
       |   movzbw   | 将做了零扩展的字节传送到字                            |           |
       |   movzbl   | 将做了零扩展的字节传送到双字                           |           |
       |   movzwl   | 将做了零扩展的字传送到双字                            |           |
       |  pushl  S  | R[%esp]  <-- R[%esp] - 4 ; M[R[%esp]] <-- S | 将双字压栈     |
       |  popl   D  | D <-- M[R[%esp]] ; R[%esp] <-- R[%esp+4] | 将双字出栈     |
       |            |                                          |           |

     - 算术和逻辑操作

       |     指令      | 效果               | 描述         |
       | :---------: | ---------------- | ---------- |
       | leal  S,  D | D <-- &S         | 加载有效地址     |
       |   INC   D   | D <-- D + 1      | 加 1        |
       |   DEC  D    | D <-- D - 1      | 减 1        |
       |   NEG  D    | D <-- -D         | 取负         |
       |   NOT  D    | D <-- ~D         | 取补         |
       |  ADD  S, D  | D <-- D + S      | 加          |
       |  SUB  S, D  | D <-- D - S      | 减          |
       | IMUL  S, D  | D <-- D * S      | 乘          |
       |  XOR  S, D  | D <-- D  ^ S     | 异或         |
       |  OR  S, D   | D <-- D  \| S    | 或          |
       |  AND  S, D  | D <-- D  & S     | 与          |
       |  SAL  k, D  | D <-- D  << k    | 左移         |
       |  SHL  k, D  | D <-- D  << k    | 左移（等同于SAL） |
       |  SAR  k, D  | D <-- D  >>$A$k  | 算术右移       |
       |  SHR  k, D  | D <-- D  >> $L$k | 逻辑右移       |
       |             |                  |            |
2. JVM指令集

   JVM的指令由一个字节长度的（**指令总数不超过256个**）、代表着某种特定操作含义的数字（`Opcode,操作码`）以及跟随其后的零至多个代表操作所需参数（`Operands,操作数`）而构成，由于Java虚拟机曹勇面向操作数栈而不是寄存器的架构，所有大多数的指令都不包含操作数，只有一个操作码：

   - 与数据类型相关的指令

     对于大部分与数据类型相关的字节码指令，它们的操作码助记符中大部分都有特殊的字符来表明专门为哪种类型服务：**i代表int类型的数据操作，l代表long,s代表short,b代表byte,c代表char,f代表float,d代表double，a代表reference。**

     | Opcode    | byte    | short   | int       | long    | float   | double  | char | reference |
     | --------- | ------- | ------- | --------- | ------- | ------- | ------- | ---- | --------- |
     | Tstore    |         |         | istore    | lstore  | fstore  | dstore  |      | astore    |
     | Tinc      |         |         | iinc      |         |         |         |      |           |
     | Taload    | baload  | saload  | iaload    | laload  | faload  | daload  |      | aaload    |
     | Tastore   | bastore | sastore | iastore   | lastore | fstore  | dastore |      | aastore   |
     | Tadd      |         |         | iadd      | ladd    | fadd    | dadd    |      |           |
     | Tsub      |         |         | isub      | lsub    | fsub    | dsub    |      |           |
     | Tmul      |         |         | imul      | lmul    | fmul    | dmul    |      |           |
     | Tdiv      |         |         | idiv      | ldiv    | fdiv    | ddiv    |      |           |
     | Trem      |         |         | irem      | lrem    | frem    | drem    |      |           |
     | Tneg      |         |         | ineg      | lneg    | fneg    | dneg    |      |           |
     | Tshl      |         |         | ishl      | lshl    |         |         |      |           |
     | Tshr      |         |         | ishr      | lshr    |         |         |      |           |
     | Tushr     |         |         | iushr     | lushr   |         |         |      |           |
     | Tand      |         |         | iand      | lor     |         |         |      |           |
     | Tor       |         |         | ior       |         |         |         |      |           |
     | Txor      |         |         | ixor      | lxor    |         |         |      |           |
     | i2T       | i2b     | i2s     |           | i2l     | i2f     | i2d     |      |           |
     | l2T       |         |         | l2i       |         | l2f     | l2d     |      |           |
     | f2T       |         |         | f2i       | f2l     |         | f2d     |      |           |
     | d2T       |         |         | d2i       | d2l     | d2f     |         |      |           |
     | Tcmp      |         |         |           | lcmp    |         |         |      |           |
     | Tcmpl     |         |         |           |         | fcmpl   | dcmpl   |      |           |
     | Tcmpg     |         |         |           |         | fcmpg   | dcmpg   |      |           |
     | if_TcmpOP |         |         | if_icmpOP |         |         |         |      | if_acmpOP |
     | Treturn   |         |         | ireturn   | lreturn | freturn | dreturn |      | areturn   |

   - 加载和存储指令

     加载和存储指令用于将数据在栈帧中的局部变量表和操作数栈之间来回传输，这类指令包括如下内容：

     - 将一个局部变量加载到操作栈：`iload`、`iload_<n>`、`lload`、`lload_<n>`、`fload`、`fload_<n>`、`dload`、`dload_<n>`、`aload`、`aload_<n>`。

     - 将一个数值从操作数栈存储到局部变量表：`istore`、`istore_<n>`、`lstore`、`lstore_<n>`、`fstore`、`fstore_<n>`、`dstore`、`dstore__<n>`、`astore`、`astore_<n>`。

     - 将一个常量加载到操作数栈：`bipush`、`sipush`、`idc`、`idc_w`、`idc2_w`、`aconst_null`、`iconst_ml`、`iconst_<i>`、`lconst_<l>`、

       `fconst_<f>`、`dconst_<d>`。

     - 扩充局部变量表的访问索引的指令：`wide`。

   - 运算指令：

     运算或算术指令用于对两个操作数栈上的值进行某种特定运算，并把结果重新存入到操作栈顶，大体上算术指令可以分为两种：对整形数据进行运算的指令与对浮点型数据进行运算的指令，无论是哪种算术指令，都使用Java虚拟机的数据类型，由于没有直接支持byte、short、char、和boolean类型的算术指令，对于这类数据的运算，应使用int类型的指令代替。

     - 加法指令：iadd、ladd、fadd、dadd。
     - 减法指令：isub、lsub、fsub、dsub。
     - 乘法指令：imul、lmul、fmul、dmul。
     - 除法指令：idiv、idiv、fdiv、ddiv。
     - 求入指令：irem、lrem、frem、drem。
     - 取反指令：ineg、lneg、fneg、dneg。
     - 位移指令：ishl、ishr、iushr、lshl、lshr、lushr。
     - 按位或指令：ior、lor。
     - 按位与指令：iand、land。
     - 按位异或指令：ixor、lxor。
     - 局部变量自增指令：iinc。
     - 比较指令：dcmpg、dcmpl、fcmpg、fcmpl、lcmp。

   - 类型转换指令：

     类型转换指令可以将两种不同的数值类型相互转换，这些转换操作将一般用于实现用户代码总的显式类型转换操作，或者用来处理字节码指令中数据类型相关指令无法与数据类型一一对应的问题。Java虚拟机直接支持（即转换时无需显式的转换指令）以下数值类型的宽化类型转换（即小范围类型向大范围类型的安全转换）。

     - int类型到long、float或者double类型。
     - long类型到float、double类型。
     - float类型到double类型。

     相对的，处理窄化类型转换时，必须显式地使用转换指令完成，这些指令包括：i2b、i2c、i2s、l2i、f2i、f2l、d2i、d2l和d2f。窄化类型转换可能导致转换结果产生不同的正负号，不同的数量级的情况，转换过程很可能导致数值精度的丢失。

   - 对象创建和访问指令

     虽然类实例和数组都是对象，但Java虚拟机对类实例和数组的创建与操作使用了不同的字节码指令。对象创建后就可以通过对象访问指令获取对象实例或者数组实例中的字段或者数组元素，这些指令如下：

     - 创建类实例的指令：new。
     - 创建数组的指令：newarray、anewarray、multianewarray。
     - 访问类字段（static字段）和实例字段（非static字段）的指令：getfield、putfiled、getstatic、putstatic。
     - 把一个数组元素加载到操作数栈的指令：baload、caload、saload、iaload、laload、faload、daload、aaload。
     - 将一个操作数栈的值存储到数组元素中的指令：bastore、castore、sastore、iastore、fastore、dastore、aastore。
     - 取数组长度的指令：arraylength。
     - 检查类实例类型的指令：instanceof、checkcast。

   - 操作数栈管理指令

     - 将操作数栈的栈顶一个或两个元素出栈：pop、pop2。
     - 复制栈顶一个或两个数值并将复制值或双份的复制值重新压入栈顶：`dup`、`dup2`、`dup_x1`、`dup2_x1`、`dup_x2`、`dup2_x2`。
     - 将栈最顶端的两个数值互换：swap。

   - 控制转移指令

     控制转移指令可以让Java虚拟机有条件或无条件地从指定的位置指令而不是控制转移指令的下一条指令继续执行程序，从概念上理解，可以认为控制转移指令就是在有条件或无条件的修改PC寄存器的值。

     - 条件分支：ifeq、iflt、ifle、ifne、ifgt、ifnull、ifnonnull、if_icmpeq、if_icmpne、if_icmplt、if_icmpgt、if_icmple、if_icmpge、if_acmpeq和if_acmpne。
     - 复合条件分支：tableswitch、lookupswitch。
     - 无条件分支：goto、goto_w、jsr、jsr_w、ret。

   - 方法调用和返回指令

     - invokevirtual指令用于调用对象的实例方法，根据对象的实例类型进行分派（虚方法分派），这也是Java语言中最常见的方法分派方式。
     - invokeinterface指令用于调用接口方法，他会在运行时搜索一个实现了这个接口方法的对象，找出合适的方法进行调用。
     - invokespecial指令用于调用一些需要特殊处理的方法，包括实例初始化方法，私有方法和父类方法。
     - invokestatic指令用于调用类方法（static方法）。
     - invokedynamic指令用于在运行时动态解析出调用点限定符所引用的方法并调用该方法。

   - 异常处理指令：

     在Java程序中显示抛出异常的操作（throw语句）都由athrow指令来实现，除了用throw语句显式抛出异常的情况之外，Java虚拟机规范还规定了许多运行时异常会在其他Java虚拟机指令监测到异常状况时自动抛出。

   - 同步指令

     Java虚拟机可以支持方法级的同步和方法内部一段指令序列的同步，这两种同步的结构都是使用管程（Monitor）来支持的。

#### 三、基于栈的字节码指令集执行过程

1. 运行时栈帧结构

   栈帧是用于支持虚拟机进行方法调用和方法执行的数据结构，它是虚拟机运行时数据区中的虚拟机栈的栈元素。栈帧存储了方法的局部变量表，操作数栈、动态连接和方法返回地址等信息。**每一个方法从调用到开始至执行完成的过程都对应着一个栈帧在虚拟机里面从入栈到出栈的过程。每一个栈帧都包括了局部变量表、操作数栈、动态连接、方法返回地址和一些额外的附加信息。**

   ![](/blog/images/201706/2.png)


   - 局部变量表

     局部变量表（`Local Variable Table`）是一组变量值存储空间，用于存放方法参数和方法内部定义的局部变量。在Java程序编译为Class文件时，就在方法的Code属性的`max_locals`数据项中确定了该方法所需要分配的局部变量表的最大容量。局部变量表的容量以变量槽（`Variable Slot`）为最小单位，一个槽可以存放一个32位以内的数据类型（boolean、byte、char、short、int、float、reference或returnAddress类型）。

     **在执行方法时，虚拟机是使用局部变量表完成参数值到参数变量列表的传递过程的，如果执行的是实例方法（非static的方法），那么局部变量的表中第0位索引的Slot默认是用于传递方法所属对象实例的引用**，在方法中可以通过关键字“this”来访问到这个隐含的参数，其余参数安装参数表的顺序排列，占用从1开始的局部变量Slot，参数表分配完毕后，再根据方法体内部定义的变量顺序和作用域分配其余的Slot。

     ![](/blog/images/201706/3.png)
     ​				

     ​				**(上图调用的是实例方法，下图调用的是静态方法)**

     ![](/blog/images/201706/4.png)

   - 操作数栈

     操作数栈（`Operand Stack`）也叫操作栈，它是一个后入先出的栈（`Last In First Out LIFO`）。同局部变量表一样，操作数栈的最大深度也在编译的时候写入到了Codes属性的`max_stacks`数据项中，操作数栈的每一个元素可以是任意Java数据类型，包括long和double。32位数据类型所占的栈容量为1,64位数据类型所占的栈容量为2。在执行方法的时候，操作数栈的深度都不会超过在max_stacks数据项设定的最大值。

   - 动态连接

     每一个栈帧都包含一个指向运行时常量池中该栈帧所属的引用，持有这个引用是为了支持方法调用过程中的动态连接（`Dynamic Linking`）。Class文件的常量池中存在大量的符号引用，字节码中的方法调用指令就以常量池中指向方法的符号引用作为参数，这些符号引用一部分会在类加载阶段或者第一次使用的时候就转换为直接引用，这种转化称为静态解析，另一部分将在每一次运行期间转化为直接引用，这部分称为动态连接。

   - 方法返回地址

     当方法开始执行后，只有两种方式可以退出这个方法：

     - 执行引擎遇到任意一个方法返回的字节码指令，这个时候可能有返回值传递给上层调用者。是否有返回值和返回值类型将根据遇到何种方法返回指令来决定，这种退出的方式称为正常完成出口。
     - 在方法执行过程中遇到了异常，并且这种异常没有在方法体内得到处理就会导致方法退出，这退出方式称为异常完成出口。
     - 无论采用何种退出方式，在方法退出之后，都需要返回到方法被调用的位置，程序才能继续执行。方法退出的过程实际上就等同于吧当前栈帧出栈，因此退出时可能执行的操作有：**恢复上层方法的局部变量表和操作数栈，把返回值（如果有的话）压入调用者栈帧的操作数栈，调用PC计数器的值以指向方法调用指令后面的指令等。**

   - 附加信息

     虚拟机规范允许具体的虚拟机实现增加一个规范里没有的描述的信息到栈帧之中，例如与调试相关的信息，这部分完全取决于具体的虚拟机实现。


2. 基于栈的解释器执行过程

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


- 根据字节码可以看出，`main`方法需要深度为3的操作数栈（Stack=3）和5个Slot的局部变量空间（Locals=5）。下面使用图片来描述上面的字节码代码执行过程中的代码、操作数栈和局部变量表的变化情况。

  - 执行偏移地址为0的指令的情况

    ![](/blog/images/201706/5.png)

  - 上图展示了执行偏移地址为0的指令的情况，bipush指令的作用是将单字节的整型常量值（-128~127）推入操作数栈顶，后跟一个参数，指明推送的常量值，这里是10。

    执行偏移地址为1的指令的情况

    ![](/blog/images/201706/6.png)

  - 上图则是执行偏移地址为1的指令，istore_1指令的作用是将操作数栈顶的整型值出栈并存放到第1个局部变量Slot中。后面四条指令（3、5、6、8）都是做同样的事情，也就是在对应代码中把变量a、b、c赋值为10、20、30。后面四条指令的图就不重复画了。
    执行偏移地址为9的指令的情况

    ![](/blog/images/201706/7.png)

  - 执行完偏移地址为9、10、11的指令后

    ![](/blog/images/201706/8.png)

  - 执行偏移地址为12的指令 `invokestatic  #2` 其中 `#2  =  // com/github/wuzguo/CallFunc.calc:(III)I`，所以是调用static的`calc`方法。前面已经说过每次方法调用都会有新的栈帧产生，所以线程的栈帧结构如下：

    ![](/blog/images/201706/9.png)

  - 根据字节码可以看出，`calc`方法需要深度为2的操作数栈（Stack=2）和3个Slot的局部变量空间（Locals=3）。下面使用图片来描述上面的字节码代码执行过程中的代码、操作数栈和局部变量表的变化情况。

    ![](/blog/images/201706/10.png)

  - 执行偏移地址为0和1的指令将局部变量表对应index的值载入操作数栈中，然后执行偏移地址为2的指令，将相加的结果再次存入操作数栈中，操作数栈和局部变量表的变化情况如下：

    ![](/blog/images/201706/11.png)

  - 然后执行偏移地址为3、4的指令，将局部变量表中`index==2`的值载入操作数栈然后做减法操作，结束后栈帧的变化情况如下：

    ![](/blog/images/201706/12.png)

  - 接着执行 5: `ireturn`指令，退出当前栈帧，栈帧被弹出当前线程栈，然后接着执行main方法的偏移 地址为15的指令，将`calc`方法运算结果保存到main方法栈帧的`index==4`的局部变量表，`mian`方法栈帧的变化情况如下：

    ![](/blog/images/201706/13.png)

  - 然后按类似的方式执行`main`方法栈帧中偏移地址17至43的指令，将运算结果打印至控制台，然后退出程序。

    ​
#### 四、基于寄存器的指令集执行过程

- 寄存器

  CPU中临时存数据的结构。一个IA32的CPU包含一组8个存储32位值得寄存器，这些寄存器用来存储整数数据和指针。在多数情况下前6个寄存器可以看成通用寄存器，对他们的使用没有限制。最后两个寄存器（%ebp和%esp）保存着指向程序栈中重要位置的指针。只有根据栈管理的标准惯例才能修改这两个寄存器的值。

![](/blog/images/201706/14.png)

![](/blog/images/201706/15.png)

- 以下以简单演示的C语言函数调用过程来演示基于寄存器的指令执行过程

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

- 图解CPU执行过程

  - 初始状态假设`ebp`寄存器执行内存地址的512处，`esp`寄存器指向内存地址的544处。

    ![](/blog/images/201706/16.png)

    - 之所以将`ebp`压栈是为了保存上一个栈帧的起始位置，当执行完序号（0、1）的指令后，栈中的结构如下：

    ![](/blog/images/201706/17.png)

    - 然后序号（2、3、4）操作对其内存，并为当前栈帧分配32个字节的空间，操作结束后栈结构如下：

      ![](/blog/images/201706/18.png)

    - 序号为（5、6）的指令将立即数10和20压入寄存器`esp`指向的位置偏移24字节和20字节的地方，leal指令是指将寄存器的值加上立即数然后存放到对应的寄存器，如：

      `leal 20(%esp), %eax` 是将寄存器`esp`指向的内存地址加上20然后取值，然后存放在寄存器`eax`中，所以执行完序号（5、6、7、8、9、10）后栈的结构如下：

      ![](/blog/images/201706/19.png)

    - call指令的效果是将返回地址入栈（`push`），并跳转到被调用函数的起始处。返回地址是在程序中紧跟在call后面的那条指令的地址，这样当被调用函数返回时，执行会从此处继续，ret指令从栈中弹出（`pop`）地址，并跳转到这个位置。所以 call _calc 就是调用calc函数并将函数地址入栈，执行完成后结果如下：

      ![](/blog/images/201706/20.png)

    - 然后是执行calc函数序号为（0、1、2）指令将上一个函数（`mian`）的栈帧开始位置入栈并分配内存地址，操作完成后对应的栈帧结构如下：
      ![](/blog/images/201706/21.png)

    - 序号为（3、4、5、6、7、8）的操作就是通过栈帧中存放的内存地址，获取变量10和20。

      操作`movl 8(%ebp), %eax` 表示将`ebp`指向的内存地址加8然后取值（484）然后放入`eax`寄存器。

      操作`movl (%eax), %eax` 表示将`eax`指向的内存地址的值（20）然后放入`eax`寄存器。操作完成后结构如下：

      ![](/blog/images/201706/22.png)

    - 操作（9、10、11）将10和20相加，并将结果存放在`eax`中。然后执行`leave`（为返回操作准备栈）指令和ret（从调用函数返回）指令。然后返回到`main`方法序号为12的指令，此时`esp`寄存器地址还原指向464，结构如下：

      ![](/blog/images/201706/23.png)

    - 后续操作类似，不再详述。

      ​