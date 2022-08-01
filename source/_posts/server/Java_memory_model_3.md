---
title: Java内存模型（三）
date: 2017-06-29 22:38:40 
author: Zak
avatar: /blog/images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wuzguo
authorDesc: 一个追求进步的「十八线码农」
categories: 后台
tags: 
- JVM
- JMM
- 内存模型
keywords: JVM，JMM，内存模型
photos:
- /blog/images/201706_1/3.png
description: Java内存模型定义及实现原理的介绍
---

### 六、支撑Java内存模型的基础原理

#### 指令重排序
  在执行程序时，为了提高性能，编译器和处理器会对指令做重排序。但是，`JMM`确保在不同的编译器和不同的处理器平台之上，通过插入特定类型的`Memory Barrier`来禁止特定类型的编译器重排序和处理器重排序。重排序分三种类型：

- 编译器优化的重排序：编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。

- 指令级并行的重排序：现代处理器采用了指令级并行技术（`Instruction-Level Parallelism， ILP`）来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。

- 内存系统的重排序：由于处理器使用缓存和读/写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

   从java源代码到最终实际执行的指令序列，会分别经历下面三种重排序：

   ![](/images/201706_1/5.png)

​    上图中的 **`1` 属于编译器重排序，`2` 和 `3` 属于处理器重排序。**这些重排序都可能会导致多线程程序出现内存可见性问题。

- 对于编译器，`JMM `的编译器重排序规则会禁止特定类型的编译器重排序（**不是所有的编译器重排序都要禁止**）。
- 对于处理器重排序，`JMM`的处理器重排序规则会要求java编译器在生成指令序列时，插入特定类型的内存屏障（`Memory Barrier`）指令，通过内存屏障指令来禁止特定类型的处理器重排序（**不是所有的处理器重排序都要禁止**）。

   **`JMM`确保在不同的编译器和不同的处理器平台之上，通过禁止特定类型的编译器重排序和处理器重排序，为程序员提供了一致的内存可见性保证。**

#### 内存屏障（`Memory Barrier`）

​    上面讲到了通过内存屏障可以禁止特定类型处理器的重排序，从而让程序按我们预想的流程去执行。**内存屏障，又称内存栅栏，是一个CPU指令**，具有以下作用：

- 保证特定操作的执行顺序。
- 影响某些数据的内存可见性。
- 编译器和CPU能够重排序指令，保证最终相同的结果，尝试优化性能。插入一条 `Memory Barrier` 指令会告诉编译器和CPU：不管什么指令都不能和这条`Memory Barrier`指令重排序。

​    `Memory Barrier`所做的另外一件事是强制刷出各种`CPU cache`，如一个`Write-Barrier`（写入屏障）将刷出所有在`Barrier`之前写入`cache`的数据，因此，任何`CPU`上的线程都能读取到这些数据的最新版本。

​    如果一个变量是`volatile`修饰的，`JMM`会在写入这个字段之后插进一个`Write-Barrier`指令，并在读这个字段之前插入一个`Read-Barrier`指令。这意味着，如果写入一个`volatile`变量，就可以保证：

- 一个线程写入变量`a`后，任何线程访问该变量都会拿到最新值。在写入变量`a`之前的写入操作，其更新的数据对于其他线程也是可见的。因为`Memory Barrier`会刷出`cache`中的所有先前的写入。

​    现代的处理器使用写缓冲区来临时保存向内存写入的数据。写缓冲区可以保证指令流水线持续运行，它可以避免由于处理器停顿下来等待向内存写入数据而产生的延迟。同时，通过以批处理的方式刷新写缓冲区，以及合并写缓冲区中对同一内存地址的多次写，可以减少对内存总线的占用。虽然写缓冲区有这么多好处，但每个处理器（**指多个独立CPU而不是多核**）上的写缓冲区，仅仅对它所在的处理器可见。这个特性会对内存操作的执行顺序产生重要的影响：**处理器对内存的 读/写 操作的执行顺序，不一定与内存实际发生的 读/写 操作顺序一致。**为了具体说明，请看下面示例：

| Processor A | Processor B |
| ----------- | ----------- |
| a = 1; //A1 | b = 2; //B1 |
| x = b; //A2 | y = a; //B2 |

**初始状态：a = b = 0；处理器允许执行后得到结果：x = y = 0；**

​    假设处理器A和处理器B按程序的顺序并行执行内存访问，最终却可能得到x = y = 0的结果。具体的原因如下图所示：

![](/images/201706_1/6.png)
​    

​    这里处理器A和处理器B可以同时把共享变量写入自己的写缓冲区（`A1，B1`），然后从内存中读取另一个共享变量（`A2，B2`），最后才把自己写缓存区中保存的脏数据刷新到内存中（`A3，B3`）。当以这种时序执行时，程序就可以得到`x = y = 0`的结果。

​    从内存操作实际发生的顺序来看，直到处理器A执行A3来刷新自己的写缓存区，写操作A1才算真正执行了。虽然处理器A执行内存操作的顺序为：`A1->A2`，但内存操作实际发生的顺序却是：`A2->A1`。此时，处理器A的内存操作顺序被重排序了。这里的关键是，由于写缓冲区仅对自己的处理器可见，它会导致处理器执行内存操作的顺序可能会与内存实际的操作执行顺序不一致。由于现代的处理器都会使用写缓冲区，因此现代的处理器都会允许对写-读操作重排序。

下面是常见处理器允许的重排序类型的列表：

|           | Load-Load | Load-Store | Store-Store | Store-Load | 存在数据依赖 |
| --------- | --------- | ---------- | ----------- | ---------- | ------ |
| sparc-TSO | N         | N          | N           | Y          | N      |
| x86       | N         | N          | N           | Y          | N      |
| ia64      | Y         | Y          | Y           | Y          | N      |
| PowerPC   | Y         | Y          | Y           | Y          | N      |

上表单元格中的 “N” 表示处理器不允许两个操作重排序，“Y” 表示允许重排序。

从上表我们可以看出：

- 常见的处理器都允许Store-Load重排序。
- 常见的处理器都不允许对存在数据依赖的操作做重排序。
- sparc-TSO和x86拥有相对较强的处理器内存模型，它们仅允许对写-读操作做重排序（因为它们都使用了写缓冲区）。

**注：**
1. sparc-TSO是指以TSO(Total Store Order)内存模型运行时，sparc处理器的特性。
2. 上表中的x86包括x64及AMD64。
3. 由于ARM处理器的内存模型与PowerPC处理器的内存模型非常类似，本文将忽略它。
4. 数据依赖性后文会专门说明。

为了保证内存可见性，**`Java`编译器在生成指令序列的适当位置会插入内存屏障指令来禁止特定类型的处理器重排序。**`JMM`把内存屏障指令分为下列四类：

| 屏障类型                | 指令示例                        | 说明                                       |
| ------------------- | --------------------------- | ---------------------------------------- |
| LoadLoad Barriers   | Load1; LoadLoad; Load2;     | 确保`Load1`数据的装载，之前于`Load2`及所有后续装载指令的装载。   |
| StoreStore Barriers | Store1; StoreStore; Store2; | 确保`Store1`数据对其他处理器可见（刷新到内存），之前于`Store2`及所有后续存储指令的存储。 |
| LoadStore Barriers  | Load1; LoadStore; Store2;   | 确保`Load1`数据装载，之前于`Store2`及所有后续的存储指令刷新到内存。 |
| StoreLoad Barriers  | Store1; StoreLoad; Load2;   | 确保`Store1`数据对其他处理器变得可见（刷新到内存），之前于`Load2`及所有后续装载指令的装载。StoreLoad Barriers会使该屏障之前的所有内存访问指令（存储和装载指令）完成之后，才执行该屏障之后的内存访问指令。 |

​    `StoreLoad Barriers` 是一个 “全能型” 的屏障，它同时具有其他三个屏障的效果。现代的多处理器大都支持该屏障。执行该屏障开销会很昂贵，因为当前处理器通常要把写缓冲区中的数据全部刷新到内存中（`buffer fully flush`）。

#### happens-before（先行发生原则）

​    Java语言定义了先行发生原则来辅助`volatile`和`synchronized`等来保证内存模型所有操作的有序性，它是判断数据是否存在竞争、线程是否安全的主要依据。

​    先行发生原则是Java内存模型中定义的两项操作之间的偏序关系，如果说操作A先行发生于操作B，其实就是说在发生操作B之前，操作A所产生的影响能被操作B观察到，“影响”包括修改了内存中共享变量的值，发送了消息，调用了方法等。

​    下面是Java内存模型下一些“天然的”先行发生原则，这些先行发生原则无需任何同步机制协助就已经存在，可以在编码中直接使用。如果两个操作之间的关系不在此列，并且无法从下列规则中推导出来的话，它们就是没有顺序性保障，虚拟机接可以对它们随意进行重排序。

1. 程序次序原则（Program Order Rule）
  在一个线程内，按照程序代码顺序，书写在前面的操作先行发生于书写在后面的操作。
   
2. 管程锁定规则(`Monitor Lock Rule`)
对某个锁的`unlock`操作先行发生于后面对同一个锁的`lock`操作。这里必须强调的是同一个锁，这里的“后面”是指时间上的先后顺序。

3. `volatile`变量规则(`Volatile Variable Rule`)
对一个volatile变量的写操作先行发生于后面对这个变量的读操作，这里的“后面”同样是指时间上的先后顺序。也就是说，某个线程对`volatile`变量写入某个值后，能立即被其它线程读取到。

4. 线程启动规则(`Thread Start Rule`)
Thread对象的start方法先行发生于此线程的每个动作。

5. 线程终止规则(`Thread Termination Rule`)
线程中的所有操作都先行发生于对此线程的终止检测，我们可以通过`Thread.join()`方法结束，`Thread.isAlive()`的返回值等手段检测到线程是否已经终止运行。

6. 线程中断规则(`Thread Interruption Rule`)
对线程`interrupt()`方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过`Thread.interrupted()`方法检测到是否有中断发生。

7. 对象终结规则(`Thread Termination Rule`)
一个对象的初始化完成(构造函数执行结束)先行发生于它的`finalize()`方法的开始。

8. 传递性(`Transitivity`)
如果操作A先行发生于操作B，操作B先行发生于操作C，那就可以得出操作A先行发生于操作C的结论。
其中程序次序规则，管程锁定规则，`volatile`变量规则，传递性规则经常用来推断先行发生关系。需要注意的是，**只有满足以上几条先行发生原则的时间上的先后操作具备顺序可靠性，时间上的先后顺序不能得出先行发生关系**，如下示例代码所示：

```java
private int value = 0;

public void setValue(int value) {
	this.value=value;
}

public int getValue() {
	return value;
}
```

​    假设存在线程A和线程B，线程A先（时间上的先后）调用了`setValue(1)`，然后线程B调用了同一个对象的`getValue()`，那么线程B收到的返回值是不确定的，由于工作内存和主内存同步存在延迟，也由于可能存在重排序现象。 虽然时间上线程A的`setValue()`操作先于线程B的 `getValue()` 操作，但是并不能推断出线程A的`setValue()`操作先行发生于线程B的`getValue()`操作，如果有这种先行发生关系，那么可以推断出线程B的`getValue()`操作获得的值。

​    如果我们给`getValue()`方法和`setValue()`方法添加 `synchronized` 关键字，就能利用管程锁定规则推断出线程A的`setValue`操作先行发生于线程B的`getValue`操作，或者我们也可以将`value`定义为 `volatile` 变量，也能利用 `volatile` 变量规则推断出先行发生关系。

​    **先行发生关系也不能推断出时间上的先后执行顺序**，示例代码如下所示：

```java
int i=1;
int j=2;
```

​    根据程序次序规则，我们可以推断出`int i=1`的操作先行发生于`int j=2`的操作，但是`int j=2`的代码完全有可能先被处理器执行（时间上的先后），这就是重排序，虚拟机规范是允许这种特性存在的，虚拟机可利用这种特性提高性能。

**注意：**

  两个操作之间具有 `happens-before` 关系，并不意味前一个操作必须要在后一个操作之前执行。仅仅要求前一个操作的执行结果，对于后一个操作是可见的，且前一个操作按顺序排在后一个操作之前。

### 七、volatile关键字

#### volatile的特性

**`volatile`关键字是Java虚拟机提供的最轻量级的同步机制**，当一个变量定义为`volatile`后，他具备两种特性：
- 保证此变量对所有线程的可见性（当某个线程修改了这个变量的值，其他线程可以立刻得知）。
- 禁止指令重排序优化。

理解`volatile`特性的一个好方法是：把对`volatile`变量的单个读/写，看成是使用同一个锁对这些单个读/写操作做了同步。下面我们通过具体的示例来说明，请看下面的示例代码：

```java
class VolatileFeaturesExample {
    //使用volatile声明64位的long型变量
    volatile long vl = 0L;

    public void set(long l) {
        vl = l;   //单个volatile变量的写
    }

    public void getAndIncrement () {
        vl++;    //复合（多个）volatile变量的读/写
    }

    public long get() {
        return vl;   //单个volatile变量的读
    }
}
```

假设有多个线程分别调用上面程序的三个方法，这个程序在语义上和下面程序等价：

```java
class VolatileFeaturesExample {
    long vl = 0L;               // 64位的long型普通变量

    //对单个的普通 变量的写用同一个锁同步
    public synchronized void set(long l) {             
       vl = l;
    }

    public void getAndIncrement () { //普通方法调用
        long temp = get();           //调用已同步的读方法
        temp += 1L;                  //普通写操作
        set(temp);                   //调用已同步的写方法
    }
    public synchronized long get() { 
        //对单个的普通变量的读用同一个锁同步
        return vl;
    }
}
```

如上面示例程序所示，对一个`volatile`变量的单个读/写操作，与对一个普通变量的读/写操作使用同一个锁（`synchronized`）来同步，它们之间的执行效果相同。

锁的`happens-before`规则保证释放锁和获取锁的两个线程之间的内存可见性，这意味着对一个`volatile`变量的读，总是能看到（任意线程）对这个`volatile`变量最后的写入。

简而言之，`volatile`变量自身具有下列特性：

- 可见性：对一个`volatile`变量的读，总是能看到（任意线程）对这个`volatile`变量最后的写入。
- 原子性：对任意单个`volatile`变量的读/写具有原子性（即使是64位的 `long` 型 和 `double` 型变量，只要它是`volatile`变量，对该变量的读写就将具有原子性），但类似于`volatile++`这种复合操作不具有原子性。

#### `volatile`的写-读建立的`happens before`关系
上面讲的是`volatile`变量自身的特性，对程序员来说，`volatile`对线程的内存可见性的影响比`volatile`自身的特性更为重要，也更需要我们去关注。

从**`JSR-133` （Java内存模型与线程规范）**开始，`volatile`变量的写-读可以实现线程之间的通信。从内存语义的角度来说，`volatile`与锁有相同的效果：

- volatile写和锁的释放有相同的内存语义。
- volatile读与锁的获取有相同的内存语义。

请看下面使用volatile变量的示例代码：

```java
class VolatileExample {
    int a = 0;
    volatile boolean flag = false;

    public void writer() {
        a = 1;                   //1
        flag = true;               //2
    }

    public void reader() {
        if (flag) {                //3
            int i =  a;           //4
            ……
        }
    }
}
```

假设线程A执行`writer()` 方法之后，线程B执行 `reader()` 方法。根据 `happens before` 规则，这个过程建立的`happens before` 关系可以分为两类：

- 根据程序次序规则：`1` `happens before` `2`；`3` `happens before` `4`。
- 根据`volatile`规则：`2` `happens before` `3`。
- 根据`happens before` 的传递性规则：`1` `happens before` `4`。

上述`happens before` 关系的图形化表现形式如下：

![](/images/201706_1/18.png)

​    在上图中，每一个箭头链接的两个节点，代表了一个`happens before` 关系。黑色箭头表示程序顺序规则；橙色箭头表示`volatile`规则；蓝色箭头表示组合这些规则后提供的`happens before`保证。这里A线程写一个`volatile`变量后，B线程读同一个`volatile`变量。A线程在写`volatile`变量之前所有可见的共享变量，在B线程读同一个`volatile`变量后，将立即变得对B线程可见。

#### `volatile`写-读的内存语义

1. `volatile`写的内存语义如下：

  **当写一个`volatile`变量时，`JMM`会把该线程对应的本地内存中的共享变量刷新到主内存。**

​    以上面示例程序`VolatileExample`为例，假设线程A首先执行`writer()`方法，随后线程B执行`reader()`方法，初始时两个线程的本地内存中的`flag`和`a`都是初始状态。下图是线程A执行`volatile`写后，共享变量的状态示意图：

![](/images/201706_1/19.png)

如上图所示，线程A在写`flag`变量后，本地内存A中被线程A更新过的两个共享变量的值被刷新到主内存中。此时，本地内存A和主内存中的共享变量的值是一致的。

2. `volatile`读的内存语义如下：

  **当读一个`volatile`变量时，`JMM`会把该线程对应的本地内存置为无效。线程接下来将从主内存中读取共享变量。**

下面是线程B读同一个`volatile`变量后，共享变量的状态示意图：

![](/images/201706_1/20.png)

​    如上图所示，在读`flag`变量后，本地内存B已经被置为无效。此时，线程B必须从主内存中读取共享变量。线程B的读取操作将导致本地内存B与主内存中的共享变量的值也变成一致的了。如果我们把`volatile`写和`volatile`读这两个步骤综合起来看的话，在读线程B读一个`volatile`变量后，写线程A在写这个`volatile`变量之前所有可见的共享变量的值都将立即变得对读线程B可见。

下面对`volatile`写和`volatile`读的内存语义做个总结：

- 线程A写一个`volatile`变量，实质上是线程A向接下来将要读这个`volatile`变量的某个线程发出了（其对共享变量所在修改的）消息。
- 线程B读一个`volatile`变量，实质上是线程B接收了之前某个线程发出的（在写这个`volatile`变量之前对共享变量所做修改的）消息。
- 线程A写一个`volatile`变量，随后线程B读这个`volatile`变量，这个过程实质上是线程A通过主内存向线程B发送消息。

#### `volatile`内存语义的实现
下面，让我们来看看`JMM`如何实现`volatile`写/读的内存语义。

前文我们提到过**重排序分为编译器重排序和处理器重排序**。为了实现`volatile`内存语义，`JMM`会分别限制这两种类型的重排序类型。下面是`JMM`针对编译器制定的`volatile`重排序规则表：

| 是否能重排序    | 第二个操作 |           |           |
| --------- | ----- | --------- | --------- |
| 第一个操作     | 普通读/写 | volatile读 | volatile写 |
| 普通读/写     |       |           | NO        |
| volatile读 | NO    | NO        | NO        |
| volatile写 |       | NO        | NO        |

举例来说，第三行最后一个单元格的意思是：在程序顺序中，当第一个操作为普通变量的读或写时，如果第二个操作为`volatile`写，则编译器不能重排序这两个操作。

从上表我们可以看出：

- 当第二个操作是`volatile`写时，不管第一个操作是什么，都不能重排序。这个规则确保`volatile`写之前的操作不会被编译器重排序到`volatile`写之后。
- 当第一个操作是`volatile`读时，不管第二个操作是什么，都不能重排序。这个规则确保`volatile`读之后的操作不会被编译器重排序到`volatile`读之前。
- 当第一个操作是`volatile`写，第二个操作是`volatile`读时，不能重排序。

为了实现`volatile`的内存语义，编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。对于编译器来说，发现一个最优布置来最小化插入屏障的总数几乎不可能，为此，`JMM`采取保守策略。下面是基于保守策略的`JMM`内存屏障插入策略：

- 在每个`volatile`写操作的前面插入一个`StoreStore`屏障。
- 在每个`volatile`写操作的后面插入一个`StoreLoad`屏障。
- 在每个`volatile`读操作的后面插入一个`LoadLoad`屏障。
- 在每个`volatile`读操作的后面插入一个`LoadStore`屏障。

上述内存屏障插入策略非常保守，但它可以保证在任意处理器平台，任意的程序中都能得到正确的`volatile`内存语义。

下面是保守策略下，`volatile`写插入内存屏障后生成的指令序列示意图：

![](/images/201706_1/21.png)

​    上图中的`StoreStore`屏障可以保证在`volatile`写之前，其前面的所有普通写操作已经对任意处理器可见了。这是因为`StoreStore`屏障将保障上面所有的普通写在`volatile`写之前刷新到主内存。

​    最后的`StoreLoad`屏障的作用是避免`volatile`写与后面可能有的`volatile`读/写操作重排序。因为编译器常常无法准确判断在一个`volatile`写的后面，是否需要插入一个`StoreLoad`屏障（如：一个`volatile`写之后方法立即`return`）。

​    为了保证能正确实现`volatile`的内存语义，`JMM` 有两种保守策略可选择：**在每个`volatile`写的后面或在每个`volatile`读的前面插入一个`StoreLoad`屏障。**

​    从整体执行效率的角度考虑，`JMM`选择了在每个`volatile`**写的后面**插入一个`StoreLoad`屏障。因为`volatile`写-读内存语义的常见使用模式是：一个写线程写volatile变量，多个读线程读同一个volatile变量。当读线程的数量大大超过写线程时，选择在volatile写之后插入StoreLoad屏障将带来可观的执行效率的提升。从这里我们可以看到JMM在实现上的一个特点：**首先确保正确性，然后再去追求执行效率。**

​    下面是在保守策略下，volatile读插入内存屏障后生成的指令序列示意图：

![](/images/201706_1/22.png)
​    上述volatile写和volatile读的内存屏障插入策略非常保守。在实际执行时，只要不改变volatile写-读的内存语义，编译器可以根据具体情况省略不必要的屏障。下面我们通过具体的示例代码来说明：

```java
class VolatileBarrierExample {
    int a;
    volatile int v1 = 1;
    volatile int v2 = 2;
    
    void readAndWrite() {
        int i = v1;           //第一个volatile读
        int j = v2;           //第二个volatile读
        a = i + j;            //普通写
        v1 = i + 1;           //第一个volatile写
        v2 = j * 2;           //第二个 volatile写
    }
    
    //...                     //其他方法
}
```

​    针对`readAndWrite()`方法，编译器在生成字节码时可以做如下的优化：

![](/images/201706_1/23.png)

​    注意，最后的`StoreLoad`屏障不能省略。因为第二个`volatile`写之后，方法立即`return`。此时编译器可能无法准确断定后面是否会有`volatile`读或写，为了安全起见，编译器常常会在这里插入一个`StoreLoad`屏障。

​    上面的优化是针对任意处理器平台，由于不同的处理器有不同“松紧度”的处理器内存模型，内存屏障的插入还可以根据具体的处理器内存模型继续优化。以`x86`处理器为例，上图中除最后的`StoreLoad`屏障外，其它的屏障都会被省略。

前面保守策略下的`volatile`读和写，在 `x86`处理器平台可以优化成：

![](/images/201706_1/24.png)

#### `volatile` 与 `synchronized` 的比较
`volatile`本质是在告诉`JVM`当前变量在寄存器（工作内存）中的值是不确定的，需要从主存中读取。`synchronized`则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住。

它们之间的区别可以总结为以下几点：
- `volatile`仅能使用在变量级别。`synchronized`则可以使用在变量、方法、和类级别的。
- **`volatile`仅能实现变量的修改可见性，并不能保证原子性。`synchronized`则可以保证变量的修改可见性和原子性。**
- `volatile`不会造成线程的阻塞。`synchronized`可能会造成线程的阻塞。
- `volatile`标记的变量不会被编译器优化。`synchronized`标记的变量可以被编译器优化。

#### 关于`volatile`变量的可见性，经常会被误解，认为以下描述成立：
**volatile变量对所有线程是立即可见的，对`volatile`变量所有的写操作都能立即反应到其他线程中，`volatile`变量在各个线程中的值总是一致的，所以基于`volatile`变量的运算在并发操作下是安全的。**
其实`volatile`变量在各个线程的工作内存中不存在一致性问题，但是Java里面的运算并非原子操作，导致`volatile`变量的运算在并发下一样不是安全的，我们通过以下代码演示说明：

```java
public class VolatileTest {

    private static volatile int race = 0;

    private static final int THREADS_COUNT = 20;

    private static final CountDownLatch latch = new CountDownLatch(THREADS_COUNT);

    public static void increase() {
        race++;
    }

    public static void main(String[] args) {

        for (int n = 0; n < THREADS_COUNT; n++) {
            new Thread(new Runnable() {
                public void run() {
                    for (int i = 0; i < 10000; i++) {
                        increase();
                    }
                    latch.countDown();
                }
            }).start();
        }

        try {
            // 等待所有线程都结束
            latch.await();
        } catch (InterruptedException e) {
            System.out.println("exception: " + e.getMessage());
        }

        System.out.println("race: " + race);
    }
}
```
这段代码发起20个线程对`volatile`修饰的`race`变量进行`10000`次自增操作，预期结果应该是`200000`，但是每次运行得到的结果都不一样。问题就出在自增运算`race++`中，通过 `javap -verbose VolatileTest` 反编译代码的class文件发现只有一行代码的`increase()`方法在`Class`文件中是由四条指令构成的。
```java
......
  
public static void increase();

    descriptor: ()V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=0, args_size=0
         0: getstatic     #2                  // Field race:I
         3: iconst_1
         4: iadd
         5: putstatic     #2                  // Field race:I
         8: return
      LineNumberTable:
        line 32: 0
        line 33: 8    
......
```
从字节码层面上就很容易分析出原因：
当`getstatic`指令吧race的值渠道操作栈顶时，`volatile`关键字保证了`race`的值在此时是正确的，但是在执行`iconst_1、iadd`这些指令的时候，其他线程可能已经把`race`的值加大了，而在操作栈顶的值就变成了过去的数据，所以`putstatic`指令执行后可能就把较小的`race`值同步回主内存之中。

### 参考文献

- 《深入理解Java虚拟机：JVM高级特性与最佳实践》，周志明著
- [深入理解java内存模型系列文章](http://ifeve.com/java-memory-model-0/)
- [全面理解Java内存模型](http://blog.csdn.net/suifeng3051/article/details/52611310)

