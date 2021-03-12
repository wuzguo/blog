---
title: Spark学习笔记之RDD
date: 2017-08-04 21:12:34 
author: Zak
avatar: /blog/images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wuzguo
authorDesc: 一个追求进步的「十八线码农」
categories: 后台
tags: 
	- Spark
	- 日志分析
	- RDD
keywords: Spark，日志分析，RDD
photos:
	- /blog/images/201708/4.jpg
description: Spark弹性分布式数据集（RDD）的基本使用
---

### RDD

#### 术语定义

- 弹性分布式数据集（RDD）： Resillient Distributed Dataset，Spark的基本计算单元，可以通过一系列算子进行操作（主要有Transformation和Action操作）；
- 有向无环图（DAG）：Directed Acycle graph，反应RDD之间的依赖关系；
- 有向无环图调度器（DAG Scheduler）：根据Job构建基于Stage的DAG，并提交Stage给TaskScheduler；
- 任务调度器（Task Scheduler）：将Taskset提交给worker（集群）运行并回报结果；
- 窄依赖（Narrow dependency）：子RDD依赖于父RDD中固定的data partition；
- 宽依赖（Wide Dependency）：子RDD对父RDD中的所有data partition都有依赖。

#### RDD概念

  RDD是Spark的最基本抽象,是对分布式内存的抽象使用，实现了以操作本地集合的方式来操作分布式数据集的抽象实现。RDD是Spark最核心的东西，它表示已被分区，不可变的并能够被并行操作的数据集合，不同的数据集格式对应不同的RDD实现。RDD必须是可序列化的。RDD可以cache到内存中，每次对RDD数据集的操作之后的结果，都可以存放到内存中，下一个操作可以直接从内存中输入，省去了MapReduce大量的磁盘IO操作。这对于迭代运算比较常见的机器学习算法, 交互式数据挖掘来说，效率提升非常大。

  RDD 最适合那种在数据集上的所有元素都执行相同操作的批处理式应用。在这种情况下， RDD 只需记录血统中每个转换就能还原丢失的数据分区，而无需记录大量的数据操作日志。所以 RDD 不适合那些需要异步、细粒度更新状态的应用 ，比如 Web 应用的存储系统，或增量式的 Web 爬虫等。对于这些应用，使用具有事务更新日志和数据检查点的数据库系统更为高效。

#### RDD的特点

- 来源：一种是从持久存储获取数据，另一种是从其他RDD生成
- 只读：状态不可变，不能修改
- 分区：支持元素根据 Key 来分区 ( Partitioning ) ，保存到多个结点上，还原时只会重新计算丢失分区的数据，而不会影响整个系统
- 路径：在 RDD 中叫世族或血统 ( lineage ) ，即 RDD 有充足的信息关于它是如何从其他 RDD 产生而来的
- 持久化：可以控制存储级别（内存、磁盘等）来进行持久化
- 操作：丰富的动作 ( Action ) ，如Count、Reduce、Collect和Save 等

#### RDD基础数据类型

  目前有两种类型的基础RDD：并行集合（Parallelized Collections）：接收一个已经存在的Scala集合，然后进行各种并行计算。 Hadoop数据集（Hadoop Datasets）：在一个文件的每条记录上运行函数。只要文件系统是HDFS，或者hadoop支持的任意存储系统即可。这两种类型的RDD都可以通过相同的方式进行操作，从而获得子RDD等一系列拓展，形成lineage血统关系图。

- 并行化集合

​    并行化集合是通过调用SparkContext的parallelize方法，在一个已经存在的Scala集合上创建的（一个Seq对象）。集合的对象将会被拷贝，创建出一个可以被并行操作的分布式数据集。例如，下面的解释器输出，演示了如何从一个数组创建一个并行集合。

例如：`val rdd = sc.parallelize(Array(1 to 10))` 根据能启动的executor的数量来进行切分多个slice，每一个slice启动一个Task来进行处理。

`val rdd = sc.parallelize(Array(1 to 10), 5)` 指定了partition的数量

- Hadoop数据集

​    Spark可以将任何Hadoop所支持的存储资源转化成RDD,如本地文件（需要网络文件系统，所有的节点都必须能访问到）、HDFS、Cassandra、HBase、Amazon S3等，Spark支持文本文件、SequenceFiles和任何Hadoop InputFormat格式。

​ 1. 使用textFile()方法可以将本地文件或HDFS文件转换成RDD

​    支持整个文件目录读取，文件可以是文本或者压缩文件(如gzip等，自动执行解压缩并加载数据)。如textFile（”file:///dfs/data”）

​    支持通配符读取,例如：

```java
val rdd1 = sc.textFile("file:///root/access_log/access_log*.filter");
val rdd2=rdd1.map(_.split("t")).filter(_.length==6)
rdd2.count()
......
14/08/20 14:44:48 INFO HadoopRDD: Input split: file:/root/access_log/access_log.20080611.decode.filter:134217728+20705903
......
```
   textFile()可选第二个参数slice，默认情况下为每一个block分配一个slice。用户也可以通过slice指定更多的分片，但不能使用少于HDFS block的分片数。

2. 使用wholeTextFiles()读取目录里面的小文件，返回（用户名、内容）对
3. 使用sequenceFile[K,V]()方法可以将SequenceFile转换成RDD。SequenceFile文件是Hadoop用来存储二进制形式的key-value对而设计的一种平面文件(Flat File)。
4. 使用SparkContext.hadoopRDD方法可以将其他任何Hadoop输入类型转化成RDD使用方法。一般来说，HadoopRDD中每一个HDFS block都成为一个RDD分区。

此外，通过Transformation可以将HadoopRDD等转换成FilterRDD(依赖一个父RDD产生）和JoinedRDD（依赖所有父RDD）等。



#### 例子：控制台日志挖掘

  假设网站中的一个 WebService 出现错误，我们想要从数以 TB 的 HDFS 日志文件中找到问题的原因，此时我们就可以用 Spark 加载日志文件到一组结点组成集群的 RAM 中，并交互式地进行查询。以下是代码示例：

![](/blog/images/201708/3.jpg)
  首先行 1 从 HDFS 文件中创建出一个 RDD ，而行 2 则衍生出一个经过某些条件过滤后的 RDD 。行 3 将这个 RDD errors 缓存到内存中，然而第一个 RDD lines 不会驻留在内存中。这样做很有必要，因为 errors 可能非常小，足以全部装进内存，而原始数据则会非常庞大。经过缓存后，现在就可以反复重用 errors 数据了。我们这里做了两个操作，第一个是统计 errors 中包含 MySQL 字样的总行数，第二个则是取出包含 HDFS 字样的行的第三列时间，并保存成一个集合。

![](/blog/images/201708/4.jpg)
这里要注意的是前面曾经提到过的 Spark 的延迟处理。Spark 调度器会将 filter 和 map 这两个转换保存到管道，然后一起发送给结点去计算。



#### 转换与操作

对于RDD可以有两种计算方式：转换（返回值还是一个RDD）与操作（返回值不是一个RDD）

- 转换(Transformations) (如：map, filter, groupBy, join等)，Transformations操作是Lazy的，也就是说从一个RDD转换生成另一个RDD的操作不是马上执行，Spark在遇到Transformations操作时只会记录需要这样的操作，并不会去执行，需要等到有Actions操作的时候才会真正启动计算过程进行计算。
- 操作(Actions) (如：count, collect, save等)，Actions操作会返回结果或把RDD数据写到存储系统中。Actions是触发Spark启动计算的动因。

![](/blog/images/201708/5.jpg)

-  转换

| reduce(func)             | 通过函数func聚集数据集中的所有元素。Func函数接受2个参数，返回一个值。这个函数必须是关联性的，确保可以被正确的并发执行 |
| ------------------------ | ---------------------------------------- |
| collect()                | 在Driver的程序中，以数组的形式，返回数据集的所有元素。这通常会在使用filter或者其它操作后，返回一个足够小的数据子集再使用，直接将整个RDD集Collect返回，很可能会让Driver程序OOM |
| count()                  | 返回数据集的元素个数                               |
| take(n)                  | 返回一个数组，由数据集的前n个元素组成。注意，这个操作目前并非在多个节点上，并行执行，而是Driver程序所在机器，单机计算所有的元素(Gateway的内存压力会增大，需要谨慎使用） |
| first()                  | 返回数据集的第一个元素（类似于take（1）                   |
| saveAsTextFile(path)     | 将数据集的元素，以textfile的形式，保存到本地文件系统，hdfs或者任何其它hadoop支持的文件系统。Spark将会调用每个元素的toString方法，并将它转换为文件中的一行文本 |
| saveAsSequenceFile(path) | 将数据集的元素，以sequencefile的格式，保存到指定的目录下，本地系统，hdfs或者任何其它hadoop支持的文件系统。RDD的元素必须由key-value对组成，并都实现了Hadoop的Writable接口，或隐式可以转换为Writable（Spark包括了基本类型的转换，例如Int，Double，String等等） |
| foreach(func)            | 在数据集的每一个元素上，运行函数func。这通常用于更新一个累加器变量，或者和外部存储系统做交互 |

- 操作

| map(func)                            | 返回一个新的分布式数据集，由每个原元素经过func函数转换后组成         |
| ------------------------------------ | ---------------------------------------- |
| filter(func)                         | 返回一个新的数据集，由经过func函数后返回值为true的原元素组成       |
| flatMap(func)                        | 类似于map，但是每一个输入元素，会被映射为0到多个输出元素（因此，func函数的返回值是一个Seq，而不是单一元素） |
| flatMap(func)                        | 类似于map，但是每一个输入元素，会被映射为0到多个输出元素（因此，func函数的返回值是一个Seq，而不是单一元素） |
| sample(withReplacement,  frac, seed) | 根据给定的随机种子seed，随机抽样出数量为frac的数据            |
| union(otherDataset)                  | 返回一个新的数据集，由原数据集和参数联合而成                   |
| groupByKey([numTasks])               | 在一个由（K,V）对组成的数据集上调用，返回一个（K，Seq[V])对的数据集。注意：默认情况下，使用8个并行任务进行分组，你可以传入numTask可选参数，根据数据量设置不同数目的Task |
| reduceByKey(func,  [numTasks])       | 在一个（K，V)对的数据集上使用，返回一个（K，V）对的数据集，key相同的值，都被使用指定的reduce函数聚合到一起。和groupbykey类似，任务的个数是可以通过第二个可选参数来配置的。 |
| join(otherDataset,  [numTasks])      | 在类型为（K,V)和（K,W)类型的数据集上调用，返回一个（K,(V,W))对，每个key中的所有元素都在一起的数据集 |
| groupWith(otherDataset,  [numTasks]) | 在类型为（K,V)和(K,W)类型的数据集上调用，返回一个数据集，组成元素为（K, Seq[V], Seq[W]) Tuples。这个操作在其它框架，称为CoGroup |
| cartesian(otherDataset)              | 笛卡尔积。但在数据集T和U上调用时，返回一个(T，U）对的数据集，所有元素交互进行笛卡尔积。 |
| flatMap(func)                        | 类似于map，但是每一个输入元素，会被映射为0到多个输出元素（因此，func函数的返回值是一个Seq，而不是单一元素） |



#### 依赖类型

  在 RDD 中将依赖划分成了两种类型：窄依赖 (Narrow Dependencies) 和宽依赖 (Wide Dependencies) 。

- 窄依赖是指 父 RDD 的每个分区都只被子 RDD 的一个分区所使用 。
- 宽依赖就是指父 RDD 的分区被多个子 RDD 的分区所依赖。例如， Map 就是一种窄依赖，而 Join 则会导致宽依赖 ( 除非父 RDD 是 hash-partitioned ，见下图 ) 

![](/blog/images/201708/6.png)

- 窄依赖（Narrow Dependencies ）

  子RDD 的每个分区依赖于常数个父分区（即与数据规模无关）

  输入输出一对一的算子，且结果RDD 的分区结构不变，主要是map 、flatMap

  输入输出一对一，但结果RDD 的分区结构发生了变化，如union 、coalesce

  从输入中选择部分元素的算子，如filter 、distinct 、subtract 、sample

- 宽依赖（Wide Dependencies ）

  子RDD 的每个分区依赖于所有父RDD 分区

  对单个RDD 基于Key 进行重组和reduce，如groupByKey 、reduceByKey ；

  对两个RDD 基于Key 进行join 和重组，如join


#### RDD缓存

  Spark可以使用 persist 和 cache 方法将任意 RDD 缓存到内存、磁盘文件系统中。缓存是容错的，如果一个 RDD 分片丢失，可以通过构建它的 transformation自动重构。被缓存的 RDD 被使用的时，存取速度会被大大加速。一般的executor内存60%做 cache， 剩下的40%做task。

  Spark中，RDD类可以使用cache() 和 persist() 方法来缓存。cache()是persist()的特例，将该RDD缓存到内存中。而persist可以指定一个StorageLevel。StorageLevel的列表可以在StorageLevel 伴生单例对象中找到：
```java
object StorageLevel {
  val NONE = new StorageLevel(false, false, false, false)
  val DISK_ONLY = new StorageLevel(true, false, false, false)
  val DISK_ONLY_2 = new StorageLevel(true, false, false, false, 2)
  val MEMORY_ONLY = new StorageLevel(false, true, false, true)
  val MEMORY_ONLY_2 = new StorageLevel(false, true, false, true, 2)
  val MEMORY_ONLY_SER = new StorageLevel(false, true, false, false)
  val MEMORY_ONLY_SER_2 = new StorageLevel(false, true, false, false, 2)
  val MEMORY_AND_DISK = new StorageLevel(true, true, false, true)
  val MEMORY_AND_DISK_2 = new StorageLevel(true, true, false, true, 2)
  val MEMORY_AND_DISK_SER = new StorageLevel(true, true, false, false)
  val MEMORY_AND_DISK_SER_2 = new StorageLevel(true, true, false, false, 2)
  val OFF_HEAP = new StorageLevel(false, false, true, false) // Tachyon
}
```
其中，StorageLevel 类的构造器参数如下：
```java
class StorageLevel private(  private var useDisk_ : Boolean,  private var useMemory_ : Boolean,  private var useOf
```
Spark的不同StorageLevel ，目的满足内存使用和CPU效率权衡上的不同需求。我们建议通过以下的步骤来进行选择：

- 如果你的RDDs可以很好的与默认的存储级别(MEMORY_ONLY)契合，就不需要做任何修改了。这已经是CPU使用效率最高的选项，它使得RDDs的操作尽可能的快；
- 如果不行，试着使用MEMORY_ONLY_SER并且选择一个快速序列化的库使得对象在有比较高的空间使用率的情况下，依然可以较快被访问；
- 尽可能不要存储到硬盘上，除非计算数据集的函数，计算量特别大，或者它们过滤了大量的数据。否则，重新计算一个分区的速度，和与从硬盘中读取基本差不多快；
- 如果你想有快速故障恢复能力，使用复制存储级别(例如：用Spark来响应web应用的请求)。所有的存储级别都有通过重新计算丢失数据恢复错误的容错机制，但是复制存储级别可以让你在RDD上持续的运行任务，而不需要等待丢失的分区被重新计算；
- 如果你想要定义你自己的存储级别(比如复制因子为3而不是2)，可以使用StorageLevel 单例对象的apply()方法；
- 在不使用cached RDD的时候，及时使用unpersist方法来释放它。

