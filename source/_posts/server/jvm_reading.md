---
title: 深入理解Java虚拟机读书笔记
date: 2016-06-11 01:43:52
author: Zak
avatar: /blog/images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wuzguo
authorDesc: 一个追求进步的「十八线码农」
categories: 后台
tags:
- 虚拟机
- JVM
keywords: JVM，Java虚拟机
photos:
- /blog/images/201606/8.png
description: 《深入理解Java虚拟机：JVM高级特性与最佳实践》读书笔记
---

最近有幸拜读了周志明先生所著《深入理解Java虚拟机：JVM高级特性与最佳实践》一书，感触颇深，遂写读书笔记一篇以便加深记忆。

全书分五部分，共13章讲解了Java虚拟机的由来到发展成熟的过程，深入分析了Java虚拟机的实现原理、Class文件格式及加载过程、自动回收机制的实现及垃圾回收算法、虚拟机性能调优和高效多并发等方面的知识，语言通俗易懂，深入浅出，实属我等屌丝程序猿学习的佳作。

Java之所以能获得如此广泛的认可，除了它拥有结构严谨、面向对象的编程、好用到哭的API（跟某些非人类能理解的语言相比，如某++）、垃圾回收机制（写过某++的都知道内存溢出有多蛋疼）等特性之外，还有许多不可忽视的优点，比如：摆脱硬件平台的束缚，实现了“一次编写，到处运行”的理想。

传统的编程语言，如C、C++都是跟平台相关的语言，代码编写受平台约束，编译出来的程序只能在特定环境中运行，在多套操作系统或多CPU架构中，需要重新编译甚至重写代码，无形中增加了程序猿的工作难度。而Java语言引进了虚拟机的概率，通过虚拟机层（ “ 计算机科学领域的任何问题都可以通过增加一个间接的中间层来解决”）隐藏了底层技术的复杂性以及物理机和操作系统的差异性，这样程序猿们可以把主要精力放在业务实现和代码算法上。

凡事具有两面性，虽然Java虚拟机屏蔽了平台的差异性，但是程序仍旧是要在硬件平台中运行的，始终逃不过兼容硬件带来的性能损耗。传统的编译性语言（对应是解释性语言）直接编译成平台相关的程序运行起来效率肯定优于需要虚拟机做适配的语言。Java虚拟机为了使程序运行速度能达到或者接近直接在硬件平台上运行的编程语言的速度，做了一系列的优化，如：JIT技术、方法内联、逃逸分析等等。

说了这么多废话，下面真正开始讲述Java虚拟机的内容。

目前主流的主流的商业VM（虚拟机）有Sun HotSpot VM、BEA JRockit、IBM J9 VM、每家厂商都说自己的是最好的，不过Sun有开源版的JDK（OpenJDK）、可以在官网上下载，有时间了去看下源码（程序猿会有时间吗？感觉从来都是时间不够用）。注：后续所说的虚拟机都是指Sun HotSpot VM。

Java虚拟机在执行程序的过程中会把管理的内存划分为若干个不同数据区域，各个区域有不同用途，有的区域随着虚拟机进程的启动而存在，有的区域则依赖用户线程的启动和结束而建立和销毁。
![](/images/201606/1.png)

方法区和堆是线程共享的，方法区存储已被虚拟机的类加载器（ExtClassLoader、AppClassLoader、...）加载的类信息、产量、静态变量、即时编译器编译后的代码、Class对象（只有HotSpot VM 是这样）等数据。堆（Heap）是Java虚拟机管理的内存中最大的一块。几乎所有的对象实例都分配在这里（不是所有哦），也是垃圾回收器工作的主要战场，因此也被被称作“GC”堆（Garbage Collected Heap）。
![](/images/201606/2.png)

由于Java虚拟机的多线程实现四通过线程轮流切换（抢占式线程调度机制，主要的调度机制还有一种是协同式线程调度--即某个线程的任务执行完了才把CPU让给别的线程），在任何时刻，一个处理器内核只能执行一个线程中的指令。程序计数器（PC 寄存器）的作用就是为了当前线程被CPU翻牌（选中）的时候能回到上次执行到的位置，否则又得从头开始。Java虚拟机的栈（Stack）也是线程私有的，他是一种“先进后出、后进先出”的数据结构，描述的是Java方法执行的内存模型， 他的生命周期和线程相同，每个方法执行都会创建一个栈帧（Stack Frame），用于存储局部变量表、操作数栈、动态链接、方法的返回地址等信息。

如果线程请求的栈深度大于虚拟机允许的深度将抛出StackOverflowError的异常（可以解释递归函数为什么容易栈溢出）。如果虚拟机栈可以动态扩展（Windows系统32位用户态线程最大分配内存为2GB），如果无法申请到足够的内存就会抛出OutOfMenoryError异常。
![](/images/201606/3.png)

以下是抛出 StackOverflowError 异常的测试代码
<code>
/***
  StackOverflowError
  VM Args：-Xss128k
  @author zzm
 */
public class JavaVMStackSOF {

	private int stackLength = 1;
	
	public void stackLeak() {
	    stackLength++;
	    stackLeak();
	}
	
	public static void main(String[] args) throws Throwable {
	    JavaVMStackSOF oom = new JavaVMStackSOF();
	    try {
	        oom.stackLeak();
	    } catch (Throwable e) {
	        System.out.println("stack length:" + oom.stackLength);
	        throw e;
	    }
	}
	}
</code>
局部变量表存放了编译期可知的各种基本数据类型（boolean、byte、char、short、int、float、long、double）和对象引用（reference类型吗，指向对象起始地址的引用指针或代表对象的句柄）。long、double类型数据占用2个局部变量空间（Slot）。
![](/images/201606/4.png)

开始有说到堆（Heap）是Java虚拟机管理的内存中最大的一块，堆中存放了大部分对象，为了有序的管理这些对象和GC的顺利进行（GC算法主要作用于堆中，其实方法区也有GC，不过条件相对苛刻，性价比不高），虚拟机把堆分成了多个块来管理，一般分为 Eden（新生代）、Tenured（老年代）、Permgen（永生代）。
![](/images/201606/5.png)

正常情况下 Surivivor1 和 Surivivor2中有一块是空的，垃圾回收算法运行时，会将 Eden区中存活的对象和Surivivor中存活的对象复制到另外一个Surivivor区中（复制算法），然后清理掉Eden区和Surivivor区。常用的垃圾回收算法有标记-清理算法（Mark-Sweep）、复制算法（Copying，多少商业虚拟机都用这种算法回收新生代对象）、标记-整理算法（Mark-Compact，回收老年代的对象）。
![](/images/201606/6.png)

HotSpot VM根据以上算法，实现了不同的类型的垃圾回收器，如下图：
![](/images/201606/7.jpg)

Serial(串行GC、复制算法 )收集器
Serial收集器是一个新生代收集器，单线程执行，使用复制算法。它在进行垃圾收集时，必须暂停其他所有的工作线程(用户线程)。是JVM Client模式下默认的新生代收集器。对于限定单个CPU的环境来说，Serial收集器由于没有线程交互的开销，专心做垃圾收集自然可以获得最高的单线程收集效率。

ParNew(并行GC、复制算法 )收集器
ParNew收集器其实就是Serial收集器的多线程版本，除了使用多条线程进行垃圾收集之外，其余行为包括Serial收集器可用的控制参数、收集算法、Stop The World、对象分配规则、回收策略等都与Serial收集器完全一样。

Parallel Scavenge(并行GC、复制算法 )收集器
Parallel Scavenge收集器也是一个新生代收集器，它也是使用复制算法的收集器，又是并行多线程收集器。Parallel Scavenge收集器的特点是它的关注点与其他收集器不同，CMS等收集器的关注点是尽可能地缩短垃圾收集时用户线程的停顿时间，而Parallel Scavenge收集器的目标则是达到一个可控制的吞吐量。吞吐量= 程序运行时间/(程序运行时间 + 垃圾收集时间)，虚拟机总共运行了100分钟。其中垃圾收集花掉1分钟，那吞吐量就是99%。由于与吞吐量关系密切，Parallel Scavenge收集器也称为“吞吐量优先”收集器。

Serial Old(串行GC、“标记-整理”算法 )收集器
Serial Old是Serial收集器的老年代版本，它同样使用一个单线程执行收集，使用“标记-整理”算法。主要使用在Client模式下的虚拟机。

Parallel Old(并行GC、“标记-整理”算法 )收集器
Parallel Old是Parallel Scavenge收集器的老年代版本，使用多线程和“标记-整理”算法。自JDK1.6开始提供。

CMS(并发GC、“标记-清除”算法 )收集器
CMS(Concurrent Mark Sweep)收集器是一种以获取最短回收停顿时间为目标的收集器。CMS收集器是基于“标记-清除”算法实现的，整个收集过程大致分为4个步骤：
	 ①.初始标记(CMS initial mark)
	 ②.并发标记(CMS concurrenr mark)
	 ③.重新标记(CMS remark)
	 ④.并发清除(CMS concurrent sweep)

其中初始标记、重新标记这两个步骤任然需要停顿其他用户线程。初始标记仅仅只是标记出GC ROOTS能直接关联到的对象，速度很快，并发标记阶段是进行GC ROOTS 根搜索算法阶段，会判定对象是否存活。而重新标记阶段则是为了修正并发标记期间，因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间会被初始标记阶段稍长，但比并发标记阶段要短。由于整个过程中耗时最长的并发标记和并发清除过程中，收集器线程都可以与用户线程一起工作，所以整体来说，CMS收集器的内存回收过程是与用户线程一起并发执行的。

CMS收集器的优点：并发收集、低停顿。
CMS收集器的缺点：
	 CMS收集器对CPU资源非常敏感。在并发阶段，虽然不会导致用户线程停顿，但是会占用CPU资源而导致引用程序变慢，总吞吐量下降。CMS默认启动的回收线程数是：(CPU数量+3) / 4。
	 CMS收集器无法处理浮动垃圾（在垃圾回收器回收期间产生的垃圾），可能出现“Concurrent Mode Failure“，失败后而导致另一次Full  GC的产生。
	 CMS是基于“标记-清除”算法实现的收集器，使用“标记-清除”算法收集后，会产生大量碎片。空间碎片太多时，将会给对象分配带来很多麻烦，比如说大对象，内存空间找不到连续的空间来分配不得不提前触发一次Full  GC。

G1（并发和并行GC、“标记-整理”算法 ）收集器
G1(Garbage First)收集器是JDK1.7提供的一个面向服务应用的收集器，G1收集器基于“标记-整理”算法实现，也就是说不会产生内存碎片。还有一个特点之前的收集器进行收集的范围都是整个新生代或老年代，而G1将整个Java堆(包括新生代，老年代)。
G1收集器大致可划分为以下几个步骤：
     ①.初始标记(Initial Marking)
     ②.并发标记(Concurrenr Marking)
     ③.最终标记(Final Marking)
     ④.筛选回收(Live Data Counting and Evacuation)
对于追求低停顿的应用，那G1现在可以作为一个可尝试的选择，如果追求吞吐量，那G1并不是特别好的选择。

GC日志
首先看一下如下代码：
<code>
public class PrintGCDetails {
	public static void main(String[] args) {
		Object obj = new Object();
		System.gc();
		System.out.println();
		obj = new Object();
		obj = new Object();
		System.gc();
		System.out.println();
	}
	}
</code>
设置JVM参数为-XX:+PrintGCDetails,执行结果如下:
<code>[GC [PSYoungGen: 1019K->568K(28672K)] 1019K->568K(92672K), 0.0529244 secs] [Times: user=0.00 sys=0.00, real=0.06 secs]</code>
自定义注解：[GC [新生代: MinorGC前新生代内存使用->MinorGC后新生代内存使用(新生代总的内存大小)] MinorGC前JVM堆内存使用的大小->MinorGC后JVM堆内存使用的大小(堆的可用内存大小), MinorGC总耗时] [Times: 用户耗时=0.00 系统耗时=0.00, 实际耗时=0.06 secs] 

<code>[Full GC [PSYoungGen: 568K->0K(28672K)] [ParOldGen: 0K->478K(64000K)] 568K->478K(92672K) [PSPermGen: 2484K->2483K(21504K)], 0.0178331 secs] [Times: user=0.01 sys=0.00, real=0.02 secs]</code>
自定义注解：[Full GC [PSYoungGen: 568K->0K(28672K)] [老年代: FullGC前老年代内存使用->FullGC后老年代内存使用(老年代总的内存大小)] FullGC前JVM堆内存使用的大小->FullGC后JVM堆内存使用的大小(堆的可用内存大小) [永久代: 2484K->2483K(21504K)], 0.0178331 secs] [Times: user=0.01 sys=0.00, real=0.02 secs]
<code>
[GC [PSYoungGen: 501K->64K(28672K)] 980K->542K(92672K), 0.0005080 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
[Full GC [PSYoungGen: 64K->0K(28672K)] [ParOldGen: 478K->479K(64000K)] 542K->479K(92672K) [PSPermGen: 2483K->2483K(21504K)], 0.0133836 secs] [Times: user=0.05 sys=0.00, real=0.01 secs]

Heap
 PSYoungGen      total 28672K, used 1505K [0x00000000e0a00000, 0x00000000e2980000, 0x0000000100000000)  eden space 25088K, 6% used [0x00000000e0a00000,0x00000000e0b78690,0x00000000e2280000)
  from space 3584K, 0% used [0x00000000e2600000,0x00000000e2600000,0x00000000e2980000)
  to   space 3584K, 0% used [0x00000000e2280000,0x00000000e2280000,0x00000000e2600000)
 ParOldGen       total 64000K, used 479K [0x00000000a1e00000, 0x00000000a5c80000, 0x00000000e0a00000)  object space 64000K, 0% used [0x00000000a1e00000,0x00000000a1e77d18,0x00000000a5c80000)
 PSPermGen       total 21504K, used 2492K [0x000000009cc00000, 0x000000009e100000, 0x00000000a1e00000)  object space 21504K, 11% used [0x000000009cc00000,0x000000009ce6f2d0,0x000000009e100000)
</code>
注：你可以用JConsole或者Runtime.getRuntime().maxMemory(),Runtime.getRuntime().totalMemory(), Runtime.getRuntime().freeMemory()来查看Java中堆内存的大小。

再看一个例子：
<code>
public class PrintGCDetails2 {
    /**
     ** -Xms60m -Xmx60m -Xmn20m -XX:NewRatio=2 ( 若 Xms = Xmx, 并且设定了 Xmn,
     ** 那么该项配置就不需要配置了 ) -XX:SurvivorRatio=8 -XX:PermSize=30m -XX:MaxPermSize=30m
     ** -XX:+PrintGCDetails
     **/
    public static void main(String[] args)  {
          new PrintGCDetails2().doTest();
    }
     public void doTest()  {
          Integer M = new Integer(1024 * 1024 * 1); // 单位, 兆(M)
          byte[] bytes = new byte[1 * M]; // 申请 1M 大小的内存空间
          bytes = null; // 断开引用链
          System.gc(); // 通知 GC 收集垃圾
          System.out.println();
          bytes = new byte[1 * M]; // 重新申请 1M 大小的内存空间
          bytes = new byte[1 * M]; // 再次申请 1M 大小的内存空间
          System.gc();
          System.out.println();
    }
	}
</code>
运行结果：
<code>
[GC [PSYoungGen: 2007K->568K(18432K)] 2007K->568K(59392K), 0.0059377 secs] [Times: user=0.02 sys=0.00, real=0.01 secs]
[Full GC [PSYoungGen: 568K->0K(18432K)] [ParOldGen: 0K->479K(40960K)] 568K->479K(59392K) [PSPermGen: 2484K->2483K(30720K)], 0.0223249 secs] [Times: user=0.01 sys=0.00, real=0.02 secs]

[GC [PSYoungGen: 3031K->1056K(18432K)] 3510K->1535K(59392K), 0.0140169 secs] [Times: user=0.05 sys=0.00, real=0.01 secs]
[Full GC [PSYoungGen: 1056K->0K(18432K)] [ParOldGen: 479K->1503K(40960K)] 1535K->1503K(59392K) [PSPermGen: 2486K->2486K(30720K)], 0.0119497 secs] [Times: user=0.02 sys=0.00, real=0.01 secs]

Heap
 PSYoungGen      total 18432K, used 163K [0x00000000fec00000, 0x0000000100000000, 0x0000000100000000)
  eden space 16384K, 1% used [0x00000000fec00000,0x00000000fec28ff0,0x00000000ffc00000)
  from space 2048K, 0% used [0x00000000ffe00000,0x00000000ffe00000,0x0000000100000000)
  to   space 2048K, 0% used [0x00000000ffc00000,0x00000000ffc00000,0x00000000ffe00000)
 ParOldGen       total 40960K, used 1503K [0x00000000fc400000, 0x00000000fec00000, 0x00000000fec00000)
  object space 40960K, 3% used [0x00000000fc400000,0x00000000fc577e10,0x00000000fec00000)
 PSPermGen       total 30720K, used 2493K [0x00000000fa600000, 0x00000000fc400000, 0x00000000fc400000)
  object space 30720K, 8% used [0x00000000fa600000,0x00000000fa86f4f0,0x00000000fc400000)
</code>
从打印结果可以看出，堆中新生代的内存空间为 18432K ( 约 18M )，eden 的内存空间为 16384K ( 约 16M)，from / to survivor 的内存空间为 2048K ( 约 2M)。 这里所配置的 Xmn 为 20M，也就是指定了新生代的内存空间为 20M，可是从打印的堆信息来看，新生代怎么就只有 18M 呢? 另外的 2M 哪里去了? 别急，是这样的。新生代 = eden + from + to = 16 + 2 + 2 = 20M，可见新生代的内存空间确实是按 Xmn 参数分配得到的。 而且这里指定了 SurvivorRatio = 8，因此，eden = 8/10 的新生代空间 = 8/10 * 20 = 16M。from = to = 1/10 的新生代空间 = 1/10 * 20 = 2M。 堆信息中新生代的 total 18432K 是这样来的： eden + 1 个 survivor = 16384K + 2048K = 18432K，即约为 18M。 因为 jvm 每次只是用新生代中的 eden 和 一个 survivor，因此新生代实际的可用内存空间大小为所指定的 90%。 因此可以知道，这里新生代的内存空间指的是新生代可用的总的内存空间，而不是指整个新生代的空间大小。 另外，可以看出老年代的内存空间为 40960K ( 约 40M )，堆大小 = 新生代 + 老年代。因此在这里，老年代 = 堆大小 - 新生代 = 60 - 20 = 40M。 最后，这里还指定了 PermSize = 30m，PermGen 即永久代 ( 方法区 )，它还有一个名字，叫非堆，主要用来存储由 jvm 加载的类文件信息、常量、静态变量等。

垃圾收集器参数总结
<code>
<-XX:+<option> 启用选项
<-XX:-<option> 不启用选项
<-XX:<option>=<number>
<-XX:<option>=<string>
</code>
堆设置
![](/images/201606/30.png)

垃圾收集器
![](/images/201606/29.png)

Client、Server模式默认GC
![](/images/201606/28.png)

Sun/Oracle JDK GC组合方式
![](/images/201606/27.png)

垃圾回收统计信息
![](/images/201606/26.png)

说完JVM的结构及垃圾回收机制后，下面继续说类文件结构（.Class 文件）。

计算机只认识0和1，我们写的程序需要经过编译器翻译成0和1构成的二进制格式才能由计算机执行。同时我们在计算机存储的任何东西都是以0和1的序列存储的， 不同的文件格式，0和1的组合长度及顺序各不相同。

虚拟机发展成为语言无关性是未来的趋势，即未来的HotSpot VM 虚拟机可能不止能作为Java虚拟机，也可能成为Clojure、Groovy、JRuby、Scala等语言的运行载体，只要他们通过编译器编译出来的文件是HotSpot VM规定的文件格式，然而虚拟机才不关心这个文件是什么需要编译出来的。
![](/images/201606/8.png)

Class文件是一组以8位字节为单位的二进制流，各个数据项目严格按照顺序紧凑排列在文件中，根据Java虚拟机规范规定，Class文件格式采用一种类似C语言结构体的伪结构来存储数据，这个伪结构只有两种数据类型：无符号数和表。其中表示由多个无符号数或者其他表作为数据项构成的复合数据类型，所有表都习惯性的以“_info”结尾。
![](/images/201606/31.png)

常量池中常量项的结构表
![](/images/201606/9.png)

字段表（fields_info）、方法表（methods_info）结构
![](/images/201606/32.png)

注：描述符的作用是描述字段的数据类型、方法的参数列表（数量、数据类型以及参数顺序）和返回值。根据描述符的规则，基本数据类型（byte、char、short、int、long、float、double、boolean）以及代表无返回的void类型都用一个大写字母表示。而对象类型则用字符L加对象的全限定名来表示，如下表：
<table>
<tr><th>标识字符</th><th>含义</th><th>标识字符</th><th>含义</th></tr>
<tr><td>B</td><td>基本类型Byte</td><td>J</td><td>基本类型Long</td></tr>
<tr><td>C</td><td>基本类型Char</td><td>S</td><td>基本类型Short</td></tr>
<tr><td>D</td><td>基本类型Double</td><td>Z</td><td>基本类型Boolean</td></tr>
<tr><td>F</td><td>基本类型Float</td><td>V</td><td>特殊类型Void</td></tr>
<tr><td>I</td><td>基本类型Int</td><td>L</td><td>对象类型，如Ljava/lang/Object</td></tr>
</table>

对于数组类型，每一个维度将使用一个前置的“[”的符号描述，如“java.lang.String[][]”类型的二维数组，将被记录为“[[Ljava/lang/String;”,一个整形数组“int[]”将被记录为“[I”。

以下是Class文件的概览图：
![](/images/201606/10.JPG)

为了了解JVM的数据类型规范和内存分配的大体情况，我新建了HelloService.java：
<code>
package com.isoftstone.test;

public class HelloService {

    private String friendName = "jack";

    public String sayHello(String userName) {

        return "Hello" + userName;
    }

    public String getName() {

        return friendName;
    }
	}
</code>
编译为HelloService .class后，在控制台通过javap -verbose  HelloService.class 查看Class文件中的字节码指令，如下图：
<code>$ javap -verbose HelloService.class</code><code>
Classfile /D:/HelloService.class
  Last modified 2016-4-29; size 749 bytes
  MD5 checksum e6fc587dbaa73afb54da49737bf785ec
  Compiled from "HelloService.java"
public class com.isoftstone.test.HelloService
  SourceFile: "HelloService.java"
  minor version: 0
  major version: 51
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #10.#27        //  java/lang/Object."<init>":()V
   #2 = String             #28            //  jack
   #3 = Fieldref           #9.#29         //  com/isoftstone/test/HelloService.friendName:Ljava/lang/String;
   #4 = Class              #30            //  java/lang/StringBuilder
   #5 = Methodref          #4.#27         //  java/lang/StringBuilder."<init>":()V
   #6 = String             #31            //  Hello
   #7 = Methodref          #4.#32         //  java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilde                                                                                                                  r;
   #8 = Methodref          #4.#33         //  java/lang/StringBuilder.toString:()Ljava/lang/String;
   #9 = Class              #34            //  com/isoftstone/test/HelloService
  #10 = Class              #35            //  java/lang/Object
  #11 = Utf8               friendName
  #12 = Utf8               Ljava/lang/String;
  #13 = Utf8               <init>
  #14 = Utf8               ()V
  #15 = Utf8               Code
  #16 = Utf8               LineNumberTable
  #17 = Utf8               LocalVariableTable
  #18 = Utf8               this
  #19 = Utf8               Lcom/isoftstone/test/HelloService;
  #20 = Utf8               sayHello
  #21 = Utf8               (Ljava/lang/String;)Ljava/lang/String;
  #22 = Utf8               userName
  #23 = Utf8               getName
  #24 = Utf8               ()Ljava/lang/String;
  #25 = Utf8               SourceFile
  #26 = Utf8               HelloService.java
  #27 = NameAndType        #13:#14        //  "<init>":()V
  #28 = Utf8               jack
  #29 = NameAndType        #11:#12        //  friendName:Ljava/lang/String;
  #30 = Utf8               java/lang/StringBuilder
  #31 = Utf8               Hello
  #32 = NameAndType        #36:#37        //  append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
  #33 = NameAndType        #38:#24        //  toString:()Ljava/lang/String;
  #34 = Utf8               com/isoftstone/test/HelloService
  #35 = Utf8               java/lang/Object
  #36 = Utf8               append
  #37 = Utf8               (Ljava/lang/String;)Ljava/lang/StringBuilder;
  #38 = Utf8               toString
{
  public com.isoftstone.test.HelloService();
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: ldc           #2                  // String jack
         7: putfield      #3                  // Field friendName:Ljava/lang/String;
        10: return
      LineNumberTable:
        line 6: 0
        line 8: 4
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
               0      11     0  this   Lcom/isoftstone/test/HelloService;

  public java.lang.String sayHello(java.lang.String);
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=2, args_size=2
         0: new           #4                  // class java/lang/StringBuilder
         3: dup
         4: invokespecial #5                  // Method java/lang/StringBuilder."<init>":()V
         7: ldc           #6                  // String Hello
         9: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/St                                                                                                                     ringBuilder;
        12: aload_1
        13: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/St                                                                                                                     ringBuilder;
        16: invokevirtual #8                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
        19: areturn
      LineNumberTable:
        line 12: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
               0      20     0  this   Lcom/isoftstone/test/HelloService;
               0      20     1 userName   Ljava/lang/String;

  public java.lang.String getName();
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: getfield      #3                  // Field friendName:Ljava/lang/String;
         4: areturn
      LineNumberTable:
        line 17: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
               0       5     0  this   Lcom/isoftstone/test/HelloService;
}
</code>
通过WinHex查看HelloService.class文件，对应字节码文件各个部分不同的定义，我了解了下面16进制数值的具体含义，尽管不清楚ClassLoader的具体实现逻辑，但是可以想象这样一个严谨格式的文件给JVM对于内存管理和执行程序提供了多大的帮助。
![](/images/201606/11.png)

根据上面的讲述我们知道，Class文件其实就是Java语言和JVM交流的媒介，是Java虚拟机执行引擎的入口。里面既包含字符常量也包含了JVM能直接执行的指令，指令跟操作码（特定操作含义的数字，0和1的组合）对应，操作码长度为1个字节，所以Java虚拟机的指令最多有255个。JVM指令按功能可分为加载和存储指令（iload、lload、istore、lstore...）、运算指令（iadd、ladd、isub...）、类型转换指令、对象创建与访问指令、操作数栈管理指令、控制转移指令、方法调用和返回指令、异常处理指令、同步指令等。

虚拟机把描述类的数据从Class文件通过类加载器（ClassLoader）加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的Java类型，这就是虚拟机的类加载机制。
以下是JVM提供的类加载器：
     Bootstrap ClassLoader：负责加载<JAVA_HOME>\lib目录中的类库，无法被Java程序直接引用；
     Extension ClassLoader：负责加载<JAVA_HOME>\lib\ext，开发者可以直接使用；
     Application ClassLoader：加载ClassPath上所指定的类库，如果没有自己定义过自己的类加载器则会使用它；
![](/images/201606/12.png)

类加载器将Class文件加载到方法区并初始化完成后，接下来就是执行引擎开始发挥作用的时候了，执行引擎在执行Java代码的时候肯能会有解释执行（通过解释器执行）和编译执行（通过及时编译器（JIT）产生本地代码执行）两种选择，或者是两者兼备。

栈帧(Stack Frame)是用于支持虚拟机进行方法调用和方法执行的数据结构，它是虚拟机运行时数据区的虚拟机栈(Virtual Machine Stack)的栈元素。栈帧存储了方法的局部变量表，操作数栈，动态连接和方法返回地址等信息。栈帧的概念结构如下图所示：
![](/images/201606/13.png)

1.局部变量表
     变量值存储空间，用于存放方法参数和方法内部定义的局部变量。

2.操作数栈
     操作数栈也常被称为操作栈，它是一个后入先出栈。当一个方法刚刚执行的时候，这个方法的操作数栈是空的，在方法执行的过程中，会有各种字节码指向操作数栈中写入和提取值，也就是入栈与出栈操作。另外，在概念模型中，两个栈帧作为虚拟机栈的元素，相互之间是完全独立的，但是大多数虚拟机的实现里都会作一些优化处理，令两个栈帧出现一部分重叠。让下栈帧的部分操作数栈与上面栈帧的部分局部变量表重叠在一起，这样在进行方法调用返回时就可以共用一部分数据，而无须进行额外的参数复制传递了，重叠过程如下图：
![](/images/201606/14.png)

3.动态连接
     每个栈帧都包含一个指向运行时常量池中该栈帧所属性方法的引用，持有这个引用是为了支持方法调用过程中的动态连接。在Class文件的常量池中存有大量的符号引用，字节码中的方法调用指令就以常量池中指向方法的符号引用为参数。

4.方法返回地址
     方法被执行后，有两种方式退出这个方法：第一种方式是执行引擎遇到任意一个方法返回的字节码指令，这时候可能会有返回值传递给上层的方法调用者(调用当前方法的的方法称为调用者)，这种退出方法方式称为正常完成出口。另外一种退出方式是，在方法执行过程中遇到了异常，并且这个异常没有在方法体内得到处理，无论是Java虚拟机内部产生的异常，还是代码中使用athrow字节码指令产生的异常，只要在本方法的异常表中没有搜索到匹配的异常处理器，就会导致方法退出，这种退出方式称为异常完成出口。使用异常完成出口的方式退出，是不会给它的调用都产生任何返回值的。

5.附加信息
     虚拟机规范允许具体的虚拟机实现增加一些规范里没有描述的信息到栈帧中，例如与高度相关的信息，这部分信息完全取决于具体的虚拟机实现。在实际开发中，一般会把动态连接，方法返回地址与其它附加信息全部归为一类，称为栈帧信息。

Java 虚拟机里共提供了四条方法调用字节指令，分别是：
	invokestatic：调用静态方法。
	invokespecial：调用实例构造器方法、私有方法和父类方法。
	invokevirtual：调用所有的虚方法。
	invokeinterface：调用接口方法，会在运行时再确定一个实现此接口的对象。

只要能被 invokestatic 和 invokespecial 指令调用的方法，都可以在解析阶段确定唯一的调用版本，符合这个条件的有静态方法、私有方法、实例构造器和父类方法四类，它们在类加载时就会把符号引用解析为该方法的直接引用。这些方法可以称为非虚方法（还包括 final 方法），与之相反，其他方法就称为虚方法（final 方法除外）。这里要特别说明下 final 方法，虽然调用 final 方法使用的是 invokevirtual 指令，但是由于它无法覆盖，没有其他版本，所以也无需对方发接收者进行多态选择。

解析调用一定是个静态过程，在编译期间就完全确定，在类加载的解析阶段就会把涉及的符号引用转化为可确定的直接引用，不会延迟到运行期再去完成。而分派调用则可能是静态的也可能是动态的，根据分派依据的宗量数（方法的调用者和方法的参数统称为方法的宗量）又可分为单分派和多分派。两类分派方式两两组合便构成了静态单分派、静态多分派、动态单分派、动态多分派四种分派情况。

静态分派
所有依赖静态类型来定位方法执行版本的分派动作，都称为静态分派，静态分派的最典型应用就是多态性中的方法重载。静态分派发生在编译阶段，因此确定静态分配的动作实际上不是由虚拟机来执行的。
<code>
/**
 ** 方法静态分派演示
 ** @author wzguo
 */
public class StaticDispatch {

    static abstract class Human {
    }

    static class Man extends Human {
    }

    static class Woman extends Human {
    }

    public void sayHello(Human guy) {
        System.out.println("hello,guy!");
    }

    public void sayHello(Man guy) {
        System.out.println("hello,gentleman!");
    }

    public void sayHello(Woman guy) {
        System.out.println("hello,lady!");
    }

    public static void main(String[] args) {
        Human man = new Man();
        Human woman = new Woman();
        StaticDispatch sr = new StaticDispatch();
        sr.sayHello(man);
        sr.sayHello(woman);
    }
	}
</code>
程序运行结果：
<code>
hello,guy!
hello,guy!
</code>
没错，程序就是大家熟悉的重载（Overload），而且大家也应该能知道输出结果，但是为什么输出结果会是这个呢？

先来谈一下以下代码的定义：
<code>Human man = new Man();</code>
我们把 Human称为变量的 静态类型， Man称为变量的 实际类型。
其中，变量的静态类型和动态类型在程序中都可以发生变化，而区别是变量的静态类型是在编译阶段就可知的，但是动态类型要在运行期才可以确定，编译器在编译的时候并不知道变量的实际类型是什么（个人认为可能也是因为要实现多态，所以才会这样设定）。

现在回到代码中，由于方法的接受者已经确定是StaticDispatch的实例sd了，所以最终调用的是哪个重载版本也就取决于传入参数的类型了。

实际上，虚拟机（应该说是编译器）在重载时时通过参数的静态类型来当判定依据的，而且静态类型在编译期就可知，所以编译器在编译阶段就可根据静态类型来判定究竟使用哪个重载版本。于是对于例子中的两个方法的调用都是以Human为参数的版本。

再来看动态分派，它和多态的另外一个重要体现有很大的关联，这个体现是什么，可能大家也能猜出，没错，就是重写（override）。
<code>
/**
 ** 方法动态分派演示
 ** @author wzguo
 */
public class DynamicDispatch {

    static abstract class Human {
        protected abstract void sayHello();
    }

    static class Man extends Human {
        @Override
        protected void sayHello() {
            System.out.println("man say hello");
        }
    }

    static class Woman extends Human {
        @Override
        protected void sayHello() {
            System.out.println("woman say hello");
        }
    }

    public static void main(String[] args) {
        Human man = new Man();
        Human woman = new Woman();
        man.sayHello();
        woman.sayHello();
        man = new Woman();
        man.sayHello();
    }
	}
</code>
输出结果：
<code>
hello man!
hello women!
hello women!
</code>
其实由两次改变man变量的实际类型导致调用函数版本不同，我们就可以知道，虚拟机是根据变量的实际类型来调用重写方法的。

我们也可以从例子中看出，变量的实际类型是在运行期确定的，重写方法的调用也是根据实际类型来调用的。

我们把这种在运行期根据实际类型来确定方法执行版本的分派动作，称为动态分派。

前面讲了这么多理论知识，下面通过一个简单的程序来看看虚拟机是如何执行的。
<code>
public int calc() {
    int a = 100;
    int b = 200;
    int c = 300;
    return (a + b) * c;
	}
</code>
由上面的代码可以看出，该方法的逻辑很简单，就是进行简单的四则运算加减乘除，我们编译代码后使用javap -verbose命令查看字节码指令，具体字节码代码如下所示： 
<code>
public int calc();
  Code:
   Stack=2, Locals=4, Args_size=1
   0:   bipush  100
   2:   istore_1
   3:   sipush  200
   6:   istore_2
   7:   sipush  300
   10:  istore_3
   11:  iload_1
   12:  iload_2
   13:  iadd
   14:  iload_3
   15:  imul
   16:  ireturn
  LineNumberTable:
   line 3: 0
   line 4: 3
   line 5: 7
   line 6: 11
}
</code>
 根据字节码可以看出，这段代码需要深度为2的操作数栈（Stack=2）和4个Slot的局部变量空间（Locals=4）。下面，使用7张图片来描述上面的字节码代码执行过程中的代码、操作数栈和局部变量表的变化情况。
执行偏移地址为0的指令的情况
![](/images/201606/15.jpg)

上图展示了执行偏移地址为0的指令的情况，bipush指令的作用是将单字节的整型常量值（-128~127）推入操作数栈顶，后跟一个参数，指明推送的常量值，这里是100。
执行偏移地址为1的指令的情况
![](/images/201606/16.jpg)

上图则是执行偏移地址为1的指令，istore_1指令的作用是将操作数栈顶的整型值出栈并存放到第1个局部变量Slot中。后面四条指令（3、6、7、10）都是做同样的事情，也就是在对应代码中把变量a、b、c赋值为100、200、300。后面四条指令的图就不重复画了。
执行偏移地址为11的指令的情况
![](/images/201606/17.jpg)

上面展示了执行偏移地址为11的指令，iload_1指令的作用是将局部变量第1个Slot中的整型值复制到操作数栈顶。
执行偏移地址为12的指令的情况
![](/images/201606/18.jpg)

上图为执行偏移地址12的指令，iload_2指令的执行过程与iload_1类似，把第2个Slot的整型值入栈。
执行偏移地址为13的指令的情况
![](/images/201606/19.jpg)

上图展示了执行偏移地址为13的指令情况，iadd指令的作用是将操作数栈中前两个栈顶元素出栈，做整型加法，然后把结果重新入栈。在iadd指令执行完毕后，栈中原有的100和200出栈，它们相加后的和300重新入栈。
执行偏移地址为14的指令的情况
![](/images/201606/20.jpg)

上图为执行偏移地址为14的指令的情况，iload_3指令把存放在第3个局部变量Slot中的300入栈到操作数栈中。这时操作数栈为两个整数300,。
下一条偏移地址为15的指令imul是将操作数栈中前两个栈顶元素出栈，做整型乘法，然后把结果重新入栈，这里和iadd指令执行过程完全类似，所以就不重复画图了。
执行偏移地址为16的指令的情况
![](/images/201606/21.jpg)

上图是最后一条指令也就是偏移地址为16的指令的执行过程，ireturn指令是方法返回指令之一，它将结束方法执行并将操作数栈顶的整型值返回给此方法的调用者。到此为止，该方法执行结束。

下面是本书的最后一部分的内容，Java的内存模型和线程安全相关的知识。
并发处理的广泛应用使得Amdahl定律替代摩尔定律成为计算机性能发展源动力的根本原因，也是人类压榨计算机运算能力的最有力武器。
![](/images/201606/22.png)

上图是处理器、高速缓存、主内存间的交互关系
![](/images/201606/23.png)

上图是线程、主内存、工作内存三者的交互关系

volatile关键字是Java虚拟机提供的最轻量级的同步机制，当一个变量定义为volatile后，他具备两种特性：第一是保证此变量对所有线程的可见性（当某个线程修改了这个变量的值，其他线程可以立刻得知）。第二是禁止指令重排序优化。

先行发生原则(Happen-before)
     先行发生是Java内存模型中定义的两项操作之间的偏序关系，如果说操作A先行发生于操作B，其实就是说在发生操作B之前，操作A所产生的影响能被操作B观察到，“影响”包括修改了内存中共享变量的值，发送了消息，调用了方法等。

Java内存模型中天然的先行发生关系：
程序次序规则(Program Order Rule)
     在同一个线程内，程序代码里写在前面的操作先行发生于写在后面的代码。准确地说，因该是控制流顺序而不是程序代码顺序，因为要考虑分支，循环等结构。

管程锁定规则(Monitor Lock Rule)
     对某个锁的unlock操作先行发生于后面对同一个锁的lock操作。这里必须强调的是同一个锁，这里的“后面”是指时间上的先后顺序。也就是说，如果某个锁已经被lock了，那么只有它被unlock之后，其他线程才能lock该锁。表现在代码上，如果是某个同步方法，如果某个线程已经进入了该同步方法，只有当这个线程退出了该同步方法(unlock操作)，别的线程才可以进入该同步方法。
volatile变量规则(Volatile Variable Rule)
     对一个volatile变量的写操作先行发生于对这个变量的读操作，这里的“后面”同样是指时间上的先后顺序。也就是说，某个线程对volatile变量写入某个值后，能立即被其它线程读取到。
线程启动规则(Thread Start Rule)
     Thread对象的start方法先行发生于此线程的每个动作。
线程终止规则(Thread Termination Rule)
     线程中的所有操作都先行发生于对此线程的终止检测，我们可以通过Thread.join()方法结束，Thread.isAlive()的返回值等手段检测到线程是否已经终止运行。
线程中断规则(Thread Interruption Rule)
     对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过Thread.interrupted()方法检测到是否有中断发生。
对象终结规则(Finalizer Rule)
     一个对象的初始化完成(构造函数执行结束)先行发生于它的finalize()方法的开始。
传递性(Transitivity)
     如果操作A先行发生于操作B，操作B先行发生于操作C，那就可以得出操作A先行发生于操作C的结论。

其中程序次序规则，管程锁定规则，volatile变量规则，传递性规则经常用来推断先行发生关系。需要注意的是，先行发生关系和时间上的先后顺序基本没有太大的关系。
时间上的先后顺序不能得出先行发生关系，示例代码如下所示：
<code>
private int value = 0;

public void setValue(int value) {
	this.value=value;
}
public int getValue() {
	return value;
}
</code>

假设存在线程A和线程B，线程A先(时间上的先后)调用了”setValue(1)”，然后线程B调用了同一个对象的”getValue()”，那么线程B收到的返回值是不确定的，由于工作内存和主内存同步存在延迟，也由于可能存在重排序现象。 虽然时间上线程A的setValue()操作先于线程B的getValue()操作，但是并不能推断出线程A的setValue()操作先行发生于线程B的getValue()操作，如果有这种先行发生关系，那么可以推断出线程B的getValue()操作获得的值。
如果我们给getValue()方法和setValue()方法添加synchronized关键字，就能利用管程锁定规则推断出线程A的setValue操作先行发生于线程B的getValue操作，或者我们也可以将value定义为volatile变量，也能利用volatile变量规则推断出先行发生关系。

先行发生关系也不能推断出时间上的先后执行顺序，示例代码如下所示：
<code>
int i=1;
int j=2;
</code>
根据程序次序规则，我们可以推断出”int i=1”的操作先行发生于”int j=2”的操作，但是”int j=2”的代码完全有可能先被处理器执行(时间上的先后)，这就是重排序，虚拟机规范是允许这种特性存在的，虚拟机可利用这种特性提高性能。

线程是比进程更轻量级的调度执行单位，线程的引入，可以把一个进程的资源分配和执行调度分开，各个线程既可以共享进程资源（内存地址，文件I/O等），又可以独立调度（线程是CPU的调度基本单位）。

进程和线程的主要差别在于它们是不同的操作系统资源管理方式。进程有独立的地址空间，一个进程崩溃后，在保护模式下不会对其它进程 产生影响，而线程只是一个进程中的不同执行路径。线程有自己的堆栈和局部变量，但线程之间没有单独的地址空间，一个线程死掉就等于整个进程死掉，所以多进 程的程序要比多线程的程序健壮，但在进程切换时，耗费资源较大，效率要差一些。但对于一些要求同时进行并且又要共享某些变量的并发操作，只能用线程，不能 用进程。如果有兴趣深入的话，我建议你们看看《现代操作系统》或者《操作系统的设计与实现》。对就个问题说得比较清楚。


实现线程主要有三种方式：内核线程实现、用户线程实现和使用用户线程加轻量级进程混合实现。

内核线程
     内核线程就是内核的分身，一个分身可以处理一件特定事情。这在处理异步事件如异步IO时特别有用。内核线程的使用是廉价的，唯一使用的资源就是内核栈和上下文切换时保存寄存器的空间。支持多线程的内核叫做多线程内核(Multi-Threads kernel )。

轻量级进程
     轻量级线程(LWP)是一种由内核支持的用户线程。它是基于内核线程的高级抽象，因此只有先支持内核线程，才能有LWP。每一个进程有一个或多个LWPs，每个LWP由一个内核线程支持。这种模型实际上就是恐龙书上所提到的一对一线程模型。在这种实现的操作系统中，LWP就是用户线程。
由于每个LWP都与一个特定的内核线程关联，因此每个LWP都是一个独立的线程调度单元。即使有一个LWP在系统调用中阻塞，也不会影响整个进程的执行。 轻量级进程具有局限性。首先，大多数LWP的操作，如建立、析构以及同步，都需要进行系统调用。系统调用的代价相对较高：需要在user mode和kernel mode中切换。其次，每个LWP都需要有一个内核线程支持，因此LWP要消耗内核资源（内核线程的栈空间）。因此一个系统不能支持大量的LWP。
![](/images/201606/24.jpg)

用户线程
     LWP虽然本质上属于用户线程，但LWP线程库是建立在内核之上的，LWP的许多操作都要进行系统调用，因此效率不高。而这里的用户线程指的是完全建立在用户空间的线程库，用户线程的建立，同步，销毁，调度完全在用户空间完成，不需要内核的帮助。因此这种线程的操作是极其快速的且低消耗的。
![](/images/201606/25.jpg)

上图是最初的一个用户线程模型，从中可以看出，进程中包含线程，用户线程在用户空间中实现，内核并没有直接对用户线程进程调度，内核的调度对象和传统进程一样，还是进程本身，内核并不知道用户线程的存在。用户线程之间的调度由在用户空间实现的线程库实现。
这种模型对应着恐龙书中提到的多对一线程模型，其缺点是一个用户线程如果阻塞在系统调用中，则整个进程都将会阻塞。
加强版的用户线程——用户线程+LWP
这种模型对应着恐龙书中多对多模型。用户线程库还是完全建立在用户空间中，因此用户线程的操作还是很廉价，因此可以建立任意多需要的用户线程。操作系统提供了LWP作为用户线程和内核线程之间的桥梁。LWP还是和前面提到的一样，具有内核线程支持，是内核的调度单元，并且用户线程的系统调用要通过LWP，因此进程中某个用户线程的阻塞不会影响整个进程的执行。用户线程库将建立的用户线程关联到LWP上，LWP与用户线程的数量不一定一致。当内核调度到某个LWP上时，此时与该LWP关联的用户线程就被执行。
![](/images/201606/26.jpg)

当多个线程访问一个对象时，如果不用考虑这个线程在运行时环境下的调度和交替执行，也不需要进行额外的同步，或者在调用方进行任何其他的协调操作，调用这个对象的行为都可以获得正确的结果，那这个对象是线程安全的。
线程安全的实现方法：
互斥同步：同步是指在多个线程并发访问共享数据时，保证共享数据在同一个时刻自卑一个（或者是一些，使用信号量的时候）线程使用。同步的最常见方法是使用锁（Lock），锁是一种非强制机制，每一个线程在访问数据或资源之前首先试图获取（Acquire）锁，并在访问结束之后释放（Release）锁。在锁已经被占用的时候试图获取锁时，线程会等待，直到锁重新可用。临界区（Critical Section）、互斥量（Mutex）、和信号量（Semaphore）都是主要的锁（互斥 ）实现方式。在Java中，最基本的互斥同步手段就是synchronized，synchronized关键字经过编译之后，会在同步块前后分别形成monitorenter和monitorexit这两个字节码指令，这两个字节码都需要一个reference类型的参数来指明要锁定和解锁的对象。如果Java程序中synchronized明确指定了参数对象，那就是这个对象的reference；如果没有明确指定，那就根据synchronized修饰的是实例方法还是类方法，去取对于的对象实例或Class对象来作为锁对象。

二元信号量（最简单的锁）：它只有两种状态，占用与非占用。第一个试图获取二元信号量的线程会得到该锁，并将二元信号量置为占用状态，此后其他的所有试图获取该二元信号量的线程将会等待，直到该锁被释放。对于运行多个线程并发访问的资源，多元信号量检查信号量（Semaphore ），一个初始值为N的信号量允许N个线程并发访问。
线程访问资源的时候首先获取信号量，进行如下操作：
     将信号量的值减1。
     如果信号量的值小于0，则进入等待状态，否则继续执行。
访问完资源后，线程释放信号量，进行如下操作：
     将信号量的值加1。
     如果信号量的值小于1，唤醒一个等待总共的线程。

互斥量（Mutex）：类似二元信号量，资源仅同时允许一个线程访问，单核信号量不同的是，信号量在整个系统中可以被任意的线程获取和释放，也就是说同一个信号量可以被系统中的一个线程获取之后由另一个线程释放。而互斥量则要求哪个线程获取了互斥量，哪个线程就要负责释放它，其他线程越俎代庖区释放互斥量是无效的。   

临界区（Critical Section ）：比互斥量更加严格的同步手段。在术语中，把临界区的锁获取称为进入临界区，而把锁的释放称为离开临界区。临界区和互斥量与信号量的区别在于，互斥量和信号量在系统的任何进程都是可见的，也就是说，一个进程创建了一个互斥量或一个信号量，另一个进程试图去获取该锁是合法的。然而，临界区的作用范围仅限于本进程，其他的线程无法获取该锁。除此之外，临界区具有金额互斥量相同的性质。

读写锁（Read-Write Lock）：致力于根据特定的场合的同步。对于一段数据，多个线程同时读取时没有问题的，但是假设操作都不是原子型的。只要有任何一个线程试图对这个数据进行修改，就必须使用同步手段来避免出错。如果我们使用上述信号量，互斥量和临界区的任何一个来进行同步，尽管可以保证程序正确，但对于读取频繁，而仅仅偶尔写入的情况就显得非常低效。读写锁可以避免这个问题。对于同一个锁，读写锁有两种获取方式，共享的（Shared）或独占的（Exclusive）。当锁处于自由的状态时，试图以任何一种方式获取锁都能成功，并将锁置于对应得状态。如果锁处于共享状态，其他线程以共享的方式获取锁仍然会成功，此时整个锁分配给了多个线程。然而，如果其他线程试图以独占的方式获取已经处于共享状态的锁，那么它将必须等待锁被所有的线程释放。相应的，处于独占状态的锁将阻止任何其他的线程获取该锁，不论它们试图以哪种方式获取。读写锁的行为可以总结如下表：
<table><tr><th>读写锁状态</th><th>以共享方式获取</th><th>以独占方式获取</th></tr><tr><td>自由</td><td>成功</td><td>成功</td></tr><tr><td>共享</td><td>成功</td><td>等待</td></tr><tr><td>独占</td><td>等待</td><td>等待</td></tr></table>

条件变量（Condition Variable）：作用类似于栅栏。对于条件变量，线程可以有两种操作，首先线程可以等待条件变量，一个条件变量可以被多个线程等待。其次，线程可以唤醒条件变量，此时某个或多个等待此条件的线程都会被唤醒并继续支持。也就是说，使用条件变量可以让许多线程一起等待某个事件的发生，当事件发生时（条件变量被唤醒），所有的线程可以一起恢复执行。


##参考文章或书籍 
《深入理解Java虚拟机：JVM高级特性与最佳实践》--周志明著
[微信公众号-码农翻身](http://mp.weixin.qq.com/s?__biz=MzIxNjA5MTM2MA==&mid=2652432769&idx=1&sn=e369989268a3442d614c63db1b45c848&scene=0#wechat_redirect)
[垃圾收集器简介](http://blog.csdn.net/china_wanglong/article/details/38816575)
[三种线程——内核线程、轻量级进程、用户线程](http://blog.sina.com.cn/s/blog_446b43c10100cius.html)

##鸣谢
感谢本人女朋友为本文提供技术支持

##声明
本人原创作品，未经许可，不得转载