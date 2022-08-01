---
title: JVM执行引擎（一）
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

### 一、JVM的结构
   执行引擎是Java虚拟机（`JVM`）最核心的组成部分之一。执行引擎在Class文件的时候可能会有解释执行（通过解释器执行）和编译执行（通过及时编译器产生本地代码执行）两种选择，也可以二者兼备，甚至还可能会包含几个不同级别的编译器执行引擎，但是从外观上看所有的执行引擎都是一致的：**输入的是字节码，处理过程是字节码解析的等效过程，输出的是执行结果。**

![](/images/201706/1.png)

`Java虚拟机 = 类加载器（classloader） + 执行引擎（execution engine） + 运行时数据区域（runtime data area）`

### 二、机器级代码

**指令体系结构**（`Instruction set architecture, ISA`）是计算机系统对机器级程序的行为的抽象，它是指挥机器工作的指示和命令的集合，大多数ISA（如 `IA32`、`X86-64`）将程序的行为描述成**好像每条指令是按顺序执行的**，但是处理器的硬件远比描述的精细复杂，它们可以并发的执行许多指令，但是可以采取措施保证整体行为与ISA指定的顺序执行完全一致。执行程序的过程就是计算机的工作过程。指令通常包括两方面的内容：**操作码和操作数，操作码决定要完成的操作，操作数指参加运算的数据及其所在的单元地址。**在计算机中，操作要求和操作数地址都由二进制数码表示，分别称作操作码和地址码，**整条指令以二进制编码的形式存放在存储器中**。

**指令集是CPU(或虚拟机)支持的指令的集合，指令集种类和多少与具体的机型有关**：

#### CPU指令集
存储在CPU内部，对CPU运算进行指导和优化的硬程序。拥有这些指令集，CPU就可以更高效地运行。Intel主要有`x86`，`EM64T`，`MMX`，`SSE`，`SSE2`，`SSE3`，`VMX`等指令集。AMD主要是`x86`，`x86-64`，`3D-Now!`指令集。

1. 指令集的分类：
 - 精简指令集，即RISC指令集`Reduced Instruction Set Computer`：

   这种指令集的特点是指令数目少，每条指令都采用标准字长、执行时间短、中央处理器的实现细节对于机器级程序是可见的。

 - 复杂指令集，即CISC指令集`Complex Instruction Set Computer`：
   在CISC微处理器中，程序的各条指令是按顺序串行执行的，每条指令中的各个操作也是按顺序串行执行的。顺序执行的优点是控制简单，但计算机各部分的利用率不高，执行速度慢。

 通俗的理解，RICS指令集是针对CISC指令集中的一些常用指令进行优化设计，放弃了一些复杂的指令，对于复杂的功能，需要通过组合指令来完成。自然，两者的使用场合不一样，对于复杂的系统，CISC更合适，否则，RICS更合适，且低功耗。

2. 常见指令（**后续说的指令都指IA32体系指令**）：
 **大多数指令有一个或多个操作数（Operand）,指示出执行一个操作中药引用的源数据值，以及放置结果的位置。**操作数可以被分为以下三种类型：

 - 立即数（immediate），也就是常数值。`ATT(AT&T)`格式的汇编代码中，立即数书写格式为：**‘$’ + 标准C表示法表示的整数。**

 - 寄存器（register），它表示某个寄存器的内容，对双字操作来说，可以是8个32位寄存器中的一个（如：`%eax`）,对字操作来说，可以是8个16位寄存器中的一个（如：`%ax`）,或者对字节操作来说，可以是8个单字节寄存器元素总的一个（如：`%al`）,**用E$a$来表示任意寄存器，用引用R[E$a$]来表示它的值。**

 - 存储器（memory）引用，它会根据计算出来的地址（通常称为有效地址）访问某个存储器位置，可以把存储器看成一个很大的字节数组，用符号M$b$[Addr]表示对存储在存储器中从地址Addr开始的b个字节值得引用。
       
|  类型    | 格式                   | 操作数值                     | 名称       |
| -------- | ---------------------- | ---------------------------- | ---------- |
| 立即数  | $*Imm*               | *Imm*                        | 立即数寻址    |
| 寄存器  | E$a$                 | R[E$a$]                      | 寄存器寻址    |
| 存储器  | *Imm*                | M[*Imm*]                     | 绝对寻址     |
| 存储器  | (E$a$)               | M[R[E$a$]]                   | 间接寻址     |
| 存储器  | *Imm*(E$b$)          | M[*Imm*+R[E$b$]]             | （基址+偏移量）寻址 |
| 存储器  | (E$b$,E$i$)          | M[R[Eb]+R[E$i$]]             | 变址寻址       |
| 存储器  | *Imm*(E$b$,E$i$)     | M[*Imm*+R[E$b$]+R[E$i$]]     | 变址寻址       |
| 存储器  | (,E$i$,$s$)          | M[R[E$i$]*$s$]               | 比例变址寻址     |
| 存储器  | *Imm*(,E$i$,$s$)     | M[*Imm*+R[E$i$]*$s$]         | 比例变址寻址     |
| 存储器  | (E$b$,E$i$,$s$)      | M[R[E$b$]+R[E$i$]*$s$]       | 比例变址寻址     |
| 存储器  | *Imm*(E$b$,E$i$,$s$) | M[*Imm*+R[E$b$]+R[E$i$]*$s$] | 比例变址寻      |

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
|  OR  S, D   | D <-- D  ①S    | 或          |
|  AND  S, D  | D <-- D  & S     | 与          |
|  SAL  k, D  | D <-- D  << k    | 左移         |
|  SHL  k, D  | D <-- D  << k    | 左移（等同于SAL） |
|  SAR  k, D  | D <-- D  >>$A$k  | 算术右移       |
|  SHR  k, D  | D <-- D  >> $L$k | 逻辑右移       |
**注意： ① 处的符号是 / , 跟markdown有冲突 **

#### JVM指令集
JVM的指令由一个字节长度的（**指令总数不超过256个**）、代表着某种特定操作含义的数字（`Opcode,操作码`）以及跟随其后的零至多个代表操作所需参数（`Operands,操作数`）而构成，由于Java虚拟机曹勇面向操作数栈而不是寄存器的架构，所有大多数的指令都不包含操作数，只有一个操作码：

1. 与数据类型相关的指令
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

2. 加载和存储指令
 加载和存储指令用于将数据在栈帧中的局部变量表和操作数栈之间来回传输，这类指令包括如下内容：

 - 将一个局部变量加载到操作栈：`iload`、`iload_<n>`、`lload`、`lload_<n>`、`fload`、`fload_<n>`、`dload`、`dload_<n>`、`aload`、`aload_<n>`。
 - 将一个数值从操作数栈存储到局部变量表：`istore`、`istore_<n>`、`lstore`、`lstore_<n>`、`fstore`、`fstore_<n>`、`dstore`、`dstore__<n>`、`astore`、`astore_<n>`。
 - 将一个常量加载到操作数栈：`bipush`、`sipush`、`idc`、`idc_w`、`idc2_w`、`aconst_null`、`iconst_ml`、`iconst_<i>`、`lconst_<l>`、
   `fconst_<f>`、`dconst_<d>`。
 - 扩充局部变量表的访问索引的指令：`wide`。

3. 运算指令：
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

4. 类型转换指令：
 类型转换指令可以将两种不同的数值类型相互转换，这些转换操作将一般用于实现用户代码总的显式类型转换操作，或者用来处理字节码指令中数据类型相关指令无法与数据类型一一对应的问题。Java虚拟机直接支持（即转换时无需显式的转换指令）以下数值类型的宽化类型转换（即小范围类型向大范围类型的安全转换）。

 - int类型到long、float或者double类型。
 - long类型到float、double类型。
 - float类型到double类型。

 相对的，处理窄化类型转换时，必须显式地使用转换指令完成，这些指令包括：i2b、i2c、i2s、l2i、f2i、f2l、d2i、d2l和d2f。窄化类型转换可能导致转换结果产生不同的正负号，不同的数量级的情况，转换过程很可能导致数值精度的丢失。

5. 对象创建和访问指令
 虽然类实例和数组都是对象，但Java虚拟机对类实例和数组的创建与操作使用了不同的字节码指令。对象创建后就可以通过对象访问指令获取对象实例或者数组实例中的字段或者数组元素，这些指令如下：

 - 创建类实例的指令：new。
 - 创建数组的指令：newarray、anewarray、multianewarray。
 - 访问类字段（static字段）和实例字段（非static字段）的指令：getfield、putfiled、getstatic、putstatic。
 - 把一个数组元素加载到操作数栈的指令：baload、caload、saload、iaload、laload、faload、daload、aaload。
 - 将一个操作数栈的值存储到数组元素中的指令：bastore、castore、sastore、iastore、fastore、dastore、aastore。
 - 取数组长度的指令：arraylength。
 - 检查类实例类型的指令：instanceof、checkcast。

6. 操作数栈管理指令
 - 将操作数栈的栈顶一个或两个元素出栈：pop、pop2。
 - 复制栈顶一个或两个数值并将复制值或双份的复制值重新压入栈顶：`dup`、`dup2`、`dup_x1`、`dup2_x1`、`dup_x2`、`dup2_x2`。
 - 将栈最顶端的两个数值互换：swap。

7. 控制转移指令
 控制转移指令可以让Java虚拟机有条件或无条件地从指定的位置指令而不是控制转移指令的下一条指令继续执行程序，从概念上理解，可以认为控制转移指令就是在有条件或无条件的修改PC寄存器的值。

 - 条件分支：ifeq、iflt、ifle、ifne、ifgt、ifnull、ifnonnull、if_icmpeq、if_icmpne、if_icmplt、if_icmpgt、if_icmple、if_icmpge、if_acmpeq和if_acmpne。
 - 复合条件分支：tableswitch、lookupswitch。
 - 无条件分支：goto、goto_w、jsr、jsr_w、ret。

8. 方法调用和返回指令
 - invokevirtual指令用于调用对象的实例方法，根据对象的实例类型进行分派（虚方法分派），这也是Java语言中最常见的方法分派方式。
 - invokeinterface指令用于调用接口方法，他会在运行时搜索一个实现了这个接口方法的对象，找出合适的方法进行调用。
 - invokespecial指令用于调用一些需要特殊处理的方法，包括实例初始化方法，私有方法和父类方法。
 - invokestatic指令用于调用类方法（static方法）。
 - invokedynamic指令用于在运行时动态解析出调用点限定符所引用的方法并调用该方法。

9. 异常处理指令：
 在Java程序中显示抛出异常的操作（throw语句）都由athrow指令来实现，除了用throw语句显式抛出异常的情况之外，Java虚拟机规范还规定了许多运行时异常会在其他Java虚拟机指令监测到异常状况时自动抛出。

10. 同步指令
 Java虚拟机可以支持方法级的同步和方法内部一段指令序列的同步，这两种同步的结构都是使用管程（Monitor）来支持的。