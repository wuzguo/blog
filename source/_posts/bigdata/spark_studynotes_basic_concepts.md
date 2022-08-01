---
title: Spark学习笔记之基本概念
date: 2017-08-04 20:45:20 
author: Zak
avatar: /blog/images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wuzguo
authorDesc: 一个追求进步的「十八线码农」
categories: 大数据
tags: 
	- Spark
	- 日志分析
	- RDD
keywords: Spark，日志分析，RDD
photos:
	- /blog/images/201708/1.jpg
description: 关于Spark的基本概念和相关术语
---


### 基本概念

1. Hadoop只提供了Map和Reduce两种操作，Spark提供的数据集操作类型有很多种，大致分为：Transformations和Actions两大类。

   - Transformations包括Map、Filter、FlatMap、Sample、GroupByKey、ReduceByKey、Union、Join、Cogroup、MapValues、Sort和PartionBy等多种操作类型，同时还提供Count。
   - Actions包括Collect、Reduce、Lookup和Save等操作。

   另外各个处理节点之间的通信模型不再像Hadoop只有Shuffle一种模式，用户可以命名、物化，控制中间结果的存储、分区等。

2. Spark的适用场景

   目前大数据处理场景有以下几个类型：

   - 复杂的批量处理（Batch Data Processing），偏重点在于处理海量数据的能力，至于处理速度可忍受，通常的时间可能是在数十分钟到数小时
   - 基于历史数据的交互式查询（Interactive Query），通常的时间在数十秒到数十分钟之间
   - 基于实时数据流的数据处理（Streaming Data Processing），通常在数百毫秒到数秒之间

   目前对以上三种场景需求都有比较成熟的处理框架，第一种情况可以用Hadoop的MapReduce来进行批量海量数据处理，第二种情况可以Impala进行交互式查询，对于第三中情况可以用Storm分布式处理框架处理实时流式数据。以上三者都是比较独立，各自一套维护成本比较高，而Spark的出现能够一站式平台满意以上需求。

   通过以上分析，总结Spark场景有以下几个：

   - Spark是基于内存的迭代计算框架，适用于需要多次操作特定数据集的应用场合。需要反复操作的次数越多，所需读取的数据量越大，受益越大，数据量小但是计算密集度较大的场合，受益就相对较小
   - 由于RDD的特性，Spark不适用那种异步细粒度更新状态的应用，例如web服务的存储或者是增量的web爬虫和索引。就是对于那种增量修改的应用模型不适合
   - 数据量不是特别大，但是要求实时统计分析需求

   ​

3. RDD (Resilient Distributed Dataset，弹性分布式数据集) 的抽象，它是分布在一组节点中的只读对象集合，这些集合是弹性的，如果数据集一部分丢失，则可以根据“血统”对它们进行重建，保证了数据的高容错性

4. Tachyon是一个高容错的分布式文件系统，允许文件以内存的速度在集群框架中进行可靠的共享，就像Spark和 MapReduce那样。通过利用信息继承，内存侵入，Tachyon获得了高性能。

5. DAG（有向无环图）

6. Spark生态圈以Spark Core为核心，从HDFS、Amazon S3和HBase等持久层读取数据，以MESS、YARN和自身携带的Standalone为资源管理器调度Job完成Spark应用程序的计算。 这些应用程序可以来自于不同的组件，如Spark Shell/Spark Submit的批处理、Spark Streaming的实时处理应用、Spark SQL的即席查询、BlinkDB的权衡查询、MLlib/MLbase的机器学习、GraphX的图处理和SparkR的数学计算等等。

   ![](/images/201708/1.jpg)

### 模型组成

   Spark应用程序可分两部分：Driver部分和Executor部分


![](/images/201708/2.jpg)

#### Driver部分

Driver部分主要是对SparkContext进行配置、初始化以及关闭。初始化SparkContext是为了构建Spark应用程序的运行环境，在初始化SparkContext，要先导入一些Spark的类和隐式转换；在Executor部分运行完毕后，需要将SparkContext关闭。

#### Executor部分

Spark应用程序的Executor部分是对数据的处理，数据分三种：

- 原生数据（包含原生的输入数据和输出数据）

   对于输入原生数据，Spark目前提供了两种：

​      Scala集合数据集：如Array(1,2,3,4,5)，Spark使用parallelize方法转换成RDD

​      Hadoop数据集：Spark支持存储在hadoop上的文件和hadoop支持的其他文件系统，如本地文件、HBase、SequenceFile和Hadoop的输入格式。例如Spark使用txtFile方法可以将本地文件或HDFS文件转换成RDD。

  对于输出数据，Spark除了支持以上两种数据，还支持scala标量

​      生成Scala标量数据，如count（返回RDD中元素的个数）、reduce、fold/aggregate；返回几个标量，如take（返回前几个元素）。

​      生成Scala集合数据集，如collect（把RDD中的所有元素倒入 Scala集合类型）、lookup（查找对应key的所有值）。

​      生成hadoop数据集，如saveAsTextFile、saveAsSequenceFile

- RDD，RDD提供了四种算子：

​    输入算子：将原生数据转换成RDD，如parallelize、txtFile等

​    转换算子：最主要的算子，是Spark生成DAG图的对象，转换算子并不立即执行，在触发行动算子后再提交给driver处理，生成DAG图 -->  Stage --> Task  --> Worker执行。

​    缓存算子：对于要多次使用的RDD，可以缓冲加快运行速度，对重要数据可以采用多备份缓存。

​    行动算子：将运算结果RDD转换成原生数据，如count、reduce、collect、saveAsTextFile等。

- 共享变量

​    在Spark运行时，一个函数传递给RDD内的patition操作时，该函数所用到的变量在每个运算节点上都复制并维护了一份，并且各个节点之间不会相互影响。但是在Spark Application中，可能需要共享一些变量，提供Task或驱动程序使用。Spark提供了两种共享变量：

​    一、广播变量（Broadcast Variables）：可以缓存到各个节点的共享变量，通常为只读

​        广播变量缓存到各个节点的内存中，而不是每个 Task

​        广播变量被创建后，能在集群中运行的任何函数调用

​        广播变量是只读的，不能在被广播后修改

​        对于大数据集的广播， Spark 尝试使用高效的广播算法来降低通信成本

   使用方法：
```java
val broadcastVar = sc.broadcast(Array(1, 2, 3))
```
   二、累计器

​       只支持加法操作的变量，可以实现计数器和变量求和。用户可以调用SparkContext.accumulator(v)创建一个初始值为v的累加器，而运行在集群上的Task可以使用“+=”操作，但这些任务却不能读取；只有驱动程序才能获取累加器的值。

使用方法：
```java
val accum = sc.accumulator(0)
sc.parallelize(Array(1, 2, 3, 4)).foreach(x => accum  + = x)
accum.value
val num=sc.parallelize(1 to 100)
```

### Spark术语

1. Spark运行模式

| Local      | 本地模式   | 常用于本地开发测试，本地还分为local单线程和local-cluster多线程; |
| ---------- | ------ | ---------------------------------------- |
| **运行环境**   | **模式** | **描述**                                   |
| Standalone | 集群模式   | 典型的Mater/slave模式，不过也能看出Master是有单点故障的；Spark支持 ZooKeeper来实现HA |
| On yarn    | 集群模式   | 运行在yarn资源管理器框架之上，由yarn负责资源管理，Spark负责任务调度和计算 |
| On mesos   | 集群模式   | 运行在mesos资源管理器框架之上，由mesos负责资源管理，Spark负责任务调度和计算 |
| On cloud   | 集群模式   | 比如AWS的EC2，使用这个模式能很方便的访问Amazon的S3;Spark支持多种分布式存储系统：HDFS和S3 |

1. Spark常用术语

| **术语**          | **描述**                                   |
| --------------- | ---------------------------------------- |
| Application     | Spark的应用程序，包含一个Driver program和若干Executor |
| SparkContext    | Spark应用程序的入口，负责调度各个运算资源，协调各个Worker Node上的Executor |
| Driver Program  | 运行Application的main()函数并且创建SparkContext   |
| Executor        | 是为Application运行在Worker node上的一个进程，该进程负责运行Task，并且负责将数据存在内存或者磁盘上。每个Application都会申请各自的Executor来处理任务 |
| Cluster Manager | 在集群上获取资源的外部服务(例如：Standalone、Mesos、Yarn)  |
| Worker Node     | 集群中任何可以运行Application代码的节点，运行一个或多个Executor进程 |
| Task            | 运行在Executor上的工作单元                        |
| Job             | SparkContext提交的具体Action操作，常和Action对应     |
| Stage           | 每个Job会被拆分很多组task，每组任务被称为Stage，也称TaskSet  |
| RDD             | 是Resilient distributed datasets的简称，中文为弹性分布式数据集;是Spark最核心的模块和类 |
| DAGScheduler    | 根据Job构建基于Stage的DAG，并提交Stage给TaskScheduler |
| TaskScheduler   | 将Taskset提交给Worker node集群运行并返回结果          |
| Transformations | 是Spark API的一种类型，Transformation返回值还是一个RDD，所有的Transformation采用的都是懒策略，如果只是将Transformation提交是不会执行计算的 |
| Action          | 是Spark API的一种类型，Action返回值不是一个RDD，而是一个scala集合；计算只有在Action被提交的时候计算才被触发 |
| Operation       | 操作，作用于RDD的各种操作分为Transformation和Action    |

