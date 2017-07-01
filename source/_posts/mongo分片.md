---
title: 构建MongoDB HA集群：副本集和分片
date: 2017-02-16 21:10:24 
author: wúzguó
avatar: /images/avatar.png
authorLink: https://wzguo.github.io
authorAbout: https://github.com/wzguo
authorDesc: 一个自强不息，艰苦奋斗的「十八线码农」
categories: 数据库
tags: 
	- MongoDB
keywords: MongoDB，HA集群，副本集，分片
photos:
	- /images/201701/1.png
description: 构建MongoDB HA集群
---

MongoDB复制是将数据同步在多个服务器的过程。

复制提供了数据的冗余备份，并在多个服务器上存储数据副本，提高了数据的可用性， 并可以保证数据的安全性。

复制还允许您从硬件故障和服务中断中恢复数据。


## 复制原理

mongodb的复制至少需要两个节点。其中一个是主节点，负责处理客户端请求，其余的都是从节点，负责复制主节点上的数据。

mongodb各个节点常见的搭配方式为：一主一从、一主多从。

主节点记录在其上的所有操作oplog，从节点定期轮询主节点获取这些操作，然后对自己的数据副本执行这些操作，从而保证从节点的数据与主节点一致。

MongoDB复制结构图如下所示：

![MongoDB复制结构图](/images/201701/replication.png)

以上结构图总，客户端总主节点读取数据，在客户端写入数据到主节点是， 主节点与从节点进行数据交互保障数据的一致性。

### 副本集特征：

- N 个节点的集群
- 任何节点可作为主节点
- 所有写入操作都在主节点上
- 自动故障转移
- 自动恢复

------

## 副本集设置

在本教程中我们使用同一个MongoDB来做MongoDB主从的实验， 操作步骤如下：

1、关闭正在运行的MongoDB服务器。

现在我们通过指定 --replSet 选项来启动mongoDB。--replSet 基本语法格式如下：

```shell
mongod --port "PORT" --dbpath "YOUR_DB_DATA_PATH" --replSet "REPLICA_SET_INSTANCE_NAME"
```

### 实例

```shell
mongod --port 27017 --dbpath "D:\set up\mongodb\data" --replSet rs0
```

以上实例会启动一个名为rs0的MongoDB实例，其端口号为27017。

启动后打开命令提示框并连接上mongoDB服务。

在Mongo客户端使用命令rs.initiate()来启动一个新的副本集。

我们可以使用rs.conf()来查看副本集的配置

查看副本集状态使用 rs.status() 命令

------

## 副本集添加成员

添加副本集的成员，我们需要使用多条服务器来启动mongo服务。进入Mongo客户端，并使用rs.add()方法来添加副本集的成员。

### 语法

rs.add() 命令基本语法格式如下：

```shell
>rs.add(HOST_NAME:PORT)
```

### 实例

假设你已经启动了一个名为mongod1.net，端口号为27017的Mongo服务。 在客户端命令窗口使用rs.add() 命令将其添加到副本集中，命令如下所示：

```shell
>rs.add("mongod1.net:27017")
>
```

MongoDB中你只能通过主节点将Mongo服务添加到副本集中， 判断当前运行的Mongo服务是否为主节点可以使用命令db.isMaster() 。

MongoDB的副本集与我们常见的主从有所不同，主从在主机宕机后所有服务将停止，而副本集在主机宕机后，副本会接管主节点成为主节点，不会出现宕机的情况。

## 分片原理

在Mongodb里面存在另一种集群，就是分片技术,可以满足MongoDB数据量大量增长的需求。

当MongoDB存储海量的数据时，一台机器可能不足以存储数据，也可能不足以提供可接受的读写吞吐量。这时，我们就可以通过在多台机器上分割数据，使得数据库系统能存储和处理更多的数据。

下图展示了在MongoDB中使用分片集群结构分布：

![img](/images/201701/sharding.png)

上图中主要有如下所述三个主要组件：

- **Shard:**用于存储实际的数据块，实际生产环境中一个shard server角色可由几台机器组个一个replica set承担，防止主机单点故障
- **Config Server:**mongod实例，存储了整个 ClusterMetadata，其中包括 chunk信息。
- **Query Routers:**前端路由，客户端由此接入，且让整个集群看上去像单一数据库，前端应用可以透明使用。

------

## 分片实例

分片结构端口分布如下：

```shell
Shard Server 1：10001
Shard Server 2：10002
Shard Server 3：10003
Config Server ：11000
Route Process：10000
```

步骤一：启动Shard Server

```shell
[root@100 /]# mkdir -p /www/mongoDB/shard/s0
[root@100 /]# mkdir -p /www/mongoDB/shard/s1
[root@100 /]# mkdir -p /www/mongoDB/shard/s2
[root@100 /]# mkdir -p /www/mongoDB/shard/s3
[root@100 /]# mkdir -p /www/mongoDB/shard/log
[root@100 /]# /usr/local/mongoDB/bin/mongod --port 10001 --dbpath=/www/mongoDB/shard/s0 --logpath=/www/mongoDB/shard/log/s0.log --logappend --fork
....
[root@100 /]# /usr/local/mongoDB/bin/mongod --port 10003 --dbpath=/www/mongoDB/shard/s2 --logpath=/www/mongoDB/shard/log/s3.log --logappend --fork
```

步骤二： 启动Config Server

```shell
[root@100 /]# mkdir -p /www/mongoDB/shard/config
[root@100 /]# /usr/local/mongoDB/bin/mongod --port 11000 --dbpath=/www/mongoDB/shard/config --logpath=/www/mongoDB/shard/log/config.log --logappend --fork
```

**注意：**这里我们完全可以像启动普通mongodb服务一样启动，不需要添加—shardsvr和configsvr参数。因为这两个参数的作用就是改变启动端口的，所以我们自行指定了端口就可以。

步骤三： 启动Route Process

```shell
/usr/local/mongoDB/bin/mongos --port 10000 --configdb localhost:11000 --fork --logpath=/www/mongoDB/shard/log/route.log --chunkSize 500
```

mongos启动参数中，chunkSize这一项是用来指定chunk的大小的，单位是MB，默认大小为200MB.

步骤四： 配置Sharding

接下来，我们使用MongoDB Shell登录到mongos，添加Shard节点

```shell
[root@100 shard]# /usr/local/mongoDB/bin/mongo admin --port 40000
MongoDB shell version: 2.0.7
connecting to: 127.0.0.1:10000/admin
mongos> db.runCommand({ addshard:"localhost:10001" })
{ "shardAdded" : "shard0000", "ok" : 1 }
......
mongos> db.runCommand({ addshard:"localhost:10003" })
{ "shardAdded" : "shard0002", "ok" : 1 }
mongos> db.runCommand({ enablesharding:"test" }) #设置分片存储的数据库
{ "ok" : 1 }
mongos> db.runCommand({ shardcollection: "test.log", key: { id:1,time:1}})
{ "collectionsharded" : "test.log", "ok" : 1 }
```

步骤五： 程序代码内无需太大更改，直接按照连接普通的mongo数据库那样，将数据库连接接入接口10000

------

##分片和副本集结合

以下文档引用至：

[搭建高可用MongoDB集群（四）：分片]: http://blog.jobbole.com/72643/

首先确定各个组件的数量，mongos 3个， config server 3个，数据分3片 shard server 3个，每个shard 有一个副本一个仲裁也就是 3 * 2 = 6 个，总共需要部署15个实例。这些实例可以部署在独立机器也可以部署在一台机器，我们这里测试资源有限，只准备了 3台机器，在同一台机器只要端口不同就可以，看一下物理部署图：

![1](/images/201701/1.png)

架构搭好了，安装软件。

1、准备机器，IP分别设置为： 192.168.0.136、192.168.0.137、192.168.0.138。

2、分别在每台机器上建立mongodb分片对应测试文件夹。
```shell
#存放mongodb数据文件
mkdir -p /data/mongodbtest

#进入mongodb文件夹
cd  /data/mongodbtest
```
3、下载mongodb的安装程序包
```shell
wget http://fastdl.mongodb.org/linux/mongodb-linux-x86_64-2.4.8.tgz

#解压下载的压缩包
tar xvzf mongodb-linux-x86_64-2.4.8.tgz
```
4、分别在每台机器建立mongos 、config 、 shard1 、shard2、shard3 五个目录。
因为mongos不存储数据，只需要建立日志文件目录即可。
```shell
#建立mongos目录

mkdir -p /data/mongodbtest/mongos/log

#建立config server 数据文件存放目录
mkdir -p /data/mongodbtest/config/data

#建立config server 日志文件存放目录
mkdir -p /data/mongodbtest/config/log

#建立config server 日志文件存放目录
mkdir -p /data/mongodbtest/mongos/log

#建立shard1 数据文件存放目录
mkdir -p /data/mongodbtest/shard1/data

#建立shard1 日志文件存放目录
mkdir -p /data/mongodbtest/shard1/log

#建立shard2 数据文件存放目录
mkdir -p /data/mongodbtest/shard2/data

#建立shard2 日志文件存放目录
mkdir -p /data/mongodbtest/shard2/log

#建立shard3 数据文件存放目录
mkdir -p /data/mongodbtest/shard3/data

#建立shard3 日志文件存放目录
mkdir -p /data/mongodbtest/shard3/log
```
5、规划5个组件对应的端口号，由于一个机器需要同时部署 mongos、config server 、shard1、shard2、shard3，所以需要用端口进行区分。
这个端口可以自由定义，在本文 mongos为 20000， config server 为 21000， shard1为 22001 ， shard2为22002， shard3为22003.

6、在每一台服务器分别启动配置服务器。
```shell
/data/mongodbtest/mongodb-linux-x86_64-2.4.8/bin/mongod --configsvr --dbpath /data/mongodbtest/config/data --port 21000 --logpath /data/mongodbtest/config/log/config.log --fork
```
7、在每一台服务器分别启动mongos服务器。
```shell
/data/mongodbtest/mongodb-linux-x86_64-2.4.8/bin/mongos  --configdb 192.168.0.136:21000,192.168.0.137:21000,192.168.0.138:21000  --port 20000   --logpath  /data/mongodbtest/mongos/log/mongos.log --fork
```
！！！注意，报错：
```
BadValue: configdb supports only replica set connection string
```

查看MongoDB的帮助发现在3.2版本以后，configdb的方式已经变成了： 

```shell
configReplSet/<cfgsvr1:port1>,<cfgsvr2:port2>,<cfgsvr3:port3>
```

解决方案：

登录configdb 在mongo中执行：

```shell
rs.initiate( {_id: "configReplSet",
              configsvr: true,
              members:[
    		   { _id: 0, host: "192.168.190.136:21000"},
    		   { _id: 1, host: "192.168.190.137:21000"},
    		   { _id: 2, host: "192.168.190.138:21000"}
               ]
          } )
```
8、配置各个分片的副本集。
```shell
#在每个机器里分别设置分片1服务器及副本集shard1

/data/mongodbtest/mongodb-linux-x86_64-2.4.8/bin/mongod --shardsvr --replSet shard1 --port 22001 --dbpath /data/mongodbtest/shard1/data  --logpath /data/mongodbtest/shard1/log/shard1.log --fork --nojournal  --oplogSize 10
```
为了快速启动并节约测试环境存储空间，这里加上 nojournal 是为了关闭日志信息，在我们的测试环境不需要初始化这么大的redo日志。同样设置 oplogsize是为了降低 local 文件的大小，oplog是一个固定长度的 capped collection,它存在于”local”数据库中,用于记录Replica Sets操作日志。注意，这里的设置是为了测试！
```shell
#在每个机器里分别设置分片2服务器及副本集shard2

/data/mongodbtest/mongodb-linux-x86_64-2.4.8/bin/mongod --shardsvr --replSet shard2 --port 22002 --dbpath /data/mongodbtest/shard2/data  --logpath /data/mongodbtest/shard2/log/shard2.log --fork --nojournal  --oplogSize 10

#在每个机器里分别设置分片3服务器及副本集shard3

/data/mongodbtest/mongodb-linux-x86_64-2.4.8/bin/mongod --shardsvr --replSet shard3 --port 22003 --dbpath /data/mongodbtest/shard3/data  --logpath /data/mongodbtest/shard3/log/shard3.log --fork --nojournal  --oplogSize 10
```
分别对每个分片配置副本集，深入了解副本集参考本系列前几篇文章。

任意登陆一个机器，比如登陆192.168.0.136，连接mongodb
```shell
#设置第一个分片副本集

/data/mongodbtest/mongodb-linux-x86_64-2.4.8/bin/mongo  127.0.0.1:22001

#使用admin数据库

use admin

#定义副本集配置

config = { _id:"shard1", members:[
				 {_id:0,host:"192.168.0.136:22001"},
				 {_id:1,host:"192.168.0.137:22001"},
				 {_id:2,host:"192.168.0.138:22001",arbiterOnly:true}
	            ]
	     }

#初始化副本集配置

rs.initiate(config);

#设置第二个分片副本集

/data/mongodbtest/mongodb-linux-x86_64-2.4.8/bin/mongo  127.0.0.1:22002

#使用admin数据库

use admin

#定义副本集配置

config = { _id:"shard2", members:[
				 {_id:0,host:"192.168.0.136:22002"},
				 {_id:1,host:"192.168.0.137:22002"},
				 {_id:2,host:"192.168.0.138:22002",arbiterOnly:true}
	            ]
	     }

#初始化副本集配置

rs.initiate(config);

#设置第三个分片副本集

/data/mongodbtest/mongodb-linux-x86_64-2.4.8/bin/mongo    127.0.0.1:22003

#使用admin数据库

use admin

#定义副本集配置

config = {_id:"shard3", members:[
				 {_id:0,host:"192.168.0.136:22003"},
				 {_id:1,host:"192.168.0.137:22003"},
				 {_id:2,host:"192.168.0.138:22003",arbiterOnly:true}
	            ]
	     }


#初始化副本集配置

rs.initiate(config);
```
9、目前搭建了mongodb配置服务器、路由服务器，各个分片服务器，不过应用程序连接到 mongos 路由服务器并不能使用分片机制，还需要在程序里设置分片配置，让分片生效。
```shell
#连接到mongos

/data/mongodbtest/mongodb-linux-x86_64-2.4.8/bin/mongo  127.0.0.1:20000

#使用admin数据库

user  admin

#串联路由服务器与分配副本集1

db.runCommand( { addshard : "shard1/192.168.0.136:22001,192.168.0.137:22001,192.168.0.138:22001"});
```
如里shard是单台服务器，用
```
db.runCommand( { addshard : “[: ]” } )
```
这样的命令加入，如果shard是副本集，用
```
db.runCommand( { addshard : “replicaSetName/[:port[,serverhostname2[:port],…]” });
```
这样的格式表示 。

```shell
#串联路由服务器与分配副本集2

db.runCommand( { addshard : "shard2/192.168.0.136:22002,192.168.0.137:22002,192.168.0.138:22002"});

#串联路由服务器与分配副本集3

db.runCommand( { addshard : "shard3/192.168.0.136:22003,192.168.0.137:22003,192.168.0.138:22003"});

#查看分片服务器的配置

db.runCommand( { listshards : 1 } );

#内容输出

{
	 "shards" : [
		{
			"_id" : "shard1",
			"host" : "shard1/192.168.0.136:22001,192.168.0.137:22001"
		},
		{
			"_id" : "shard2",
			"host" : "shard2/192.168.0.136:22002,192.168.0.137:22002"
		},
		{
			"_id" : "shard3",
			"host" : "shard3/192.168.0.136:22003,192.168.0.137:22003"
		}
	],
	"ok" : 1
}
```
因为192.168.0.138是每个分片副本集的仲裁节点，所以在上面结果没有列出来。
10、目前配置服务、路由服务、分片服务、副本集服务都已经串联起来了，但我们的目的是希望插入数据，数据能够自动分片，就差那么一点点，一点点。。。连接在mongos上，准备让指定的数据库、指定的集合分片生效。
```shell
#指定testdb分片生效

db.runCommand( { enablesharding :"testdb"});

#指定数据库里需要分片的集合和片键

db.runCommand( { shardcollection : "testdb.table1",key : {id: 1} } )
```
我们设置testdb的 table1 表需要分片，根据 id 自动分片到 shard1 ，shard2，shard3 上面去。要这样设置是因为不是所有mongodb 的数据库和表 都需要分片！
11、测试分片配置结果。
```shell
#连接mongos服务器

/data/mongodbtest/mongodb-linux-x86_64-2.4.8/bin/mongo  127.0.0.1:20000

#使用testdb

use testdb;
```

```java
#插入测试数据

public class MongoShard {

    public static void main(String args[]) {
        try {

            List<ServerAddress> addresses = new ArrayList<ServerAddress>();
            ServerAddress address1 = new ServerAddress("localhost" , 10000);
            ServerAddress address2 = new ServerAddress("localhost" , 20000);
            addresses.add(address1);
            addresses.add(address2);

            // 连接到 mongodb 服务
            MongoClient mongoClient = new MongoClient(addresses);

            // 连接到数据库
            MongoDatabase mongoDatabase = mongoClient.getDatabase("testdb");
            System.out.println("Connect to database successfully");


            MongoCollection<Document> collection = mongoDatabase.getCollection("test");
            System.out.println("集合 test 选择成功");

            for (int num = 0; num < 100000; num++) {
                Document document = new Document("title", "MYsql").
                        append("description", "database").
                        append("likes", Math.random()).
                        append("by", "xxx").
                        append("id", num);
                collection.insertOne(document);
            }
            System.out.println("#################################");
        } catch (Exception e) {
            System.err.println(e.getClass().getName() + ": " + e.getMessage());
        }

        System.out.println("+++++++++++++++++++++++++++++++");
    }
}
```
```shell
#查看分片情况如下，部分无关信息省掉了

db.table1.stats();

{
	"sharded" : true,
	"ns" : "testdb.table1",
	"count" : 100000,
	"numExtents" : 13,
	"size" : 5600000,
	"storageSize" : 22372352,
	"totalIndexSize" : 6213760,
	"indexSizes" : {
			"_id_" : 3335808,
			"id_1" : 2877952
	},
	"avgObjSize" : 56,
	"nindexes" : 2,
	"nchunks" : 3,
	"shards" : {
		"shard1" : {
			"ns" : "testdb.table1",
			"count" : 42183,
			"size" : 0,
			...
			"ok" : 1
		},
		"shard2" : {
			"ns" : "testdb.table1",
			"count" : 38937,
			"size" : 2180472,
			...
			"ok" : 1
		},
		"shard3" : {
			"ns" : "testdb.table1",
			"count" :18880,
			"size" : 3419528,
			...
			"ok" : 1
		}
	},
	"ok" : 1
}
```
可以看到数据分到3个分片，各自分片数量为： shard1 “count” : 42183，shard2 “count” : 38937，shard3 “count” : 18880。已经成功了！不过分的好像不是很均匀，所以这个分片还是很有讲究的，后续再深入讨论。
12、java程序调用分片集群，因为我们配置了三个mongos作为入口，就算其中哪个入口挂掉了都没关系，使用集群客户端程序如下：

```java
public class TestMongoDBShards {

   public static void main(String[] args) {

	 try {
		  List<ServerAddress> addresses = new ArrayList<ServerAddress>();
		  ServerAddress address1 = new ServerAddress("192.168.0.136" , 20000);
		  ServerAddress address2 = new ServerAddress("192.168.0.137" , 20000);
		  ServerAddress address3 = new ServerAddress("192.168.0.138" , 20000);
		  addresses.add(address1);
		  addresses.add(address2);
		  addresses.add(address3);
	
		  MongoClient client = new MongoClient(addresses);
		  DB db = client.getDB( "testdb" );
		  DBCollection coll = db.getCollection( "table1" );
	
		  BasicDBObject object = new BasicDBObject();
		  object.append( "id" , 1);
	
		  DBObject dbObject = coll.findOne(object);
	
		  System. out .println(dbObject);
	
	} catch (Exception e) {
		  e.printStackTrace();
	}
  }
}
```
整个分片集群搭建完了，思考一下我们这个架构是不是足够好呢？其实还有很多地方需要优化，比如我们把所有的仲裁节点放在一台机器，其余两台机器承担了全部读写操作，但是作为仲裁的192.168.0.138相当空闲。让机器3 192.168.0.138多分担点责任吧！架构可以这样调整，把机器的负载分的更加均衡一点，每个机器既可以作为主节点、副本节点、仲裁节点，这样压力就会均衡很多了，如图：


当然生产环境的数据远远大于当前的测试数据，大规模数据应用情况下我们不可能把全部的节点像这样部署，硬件瓶颈是硬伤，只能扩展机器。要用好mongodb还有很多机制需要调整，不过通过这个东东我们可以快速实现高可用性、高扩展性，所以它还是一个非常不错的Nosql组件。

再看看我们使用的mongodb java 驱动客户端 MongoClient(addresses)，这个可以传入多个mongos 的地址作为mongodb集群的入口，并且可以实现自动故障转移，但是负载均衡做的好不好呢？打开源代码查看：


它的机制是选择一个ping 最快的机器来作为所有请求的入口，如果这台机器挂掉会使用下一台机器。那这样。。。。肯定是不行的！万一出现双十一这样的情况所有请求集中发送到这一台机器，这台机器很有可能挂掉。一但挂掉了，按照它的机制会转移请求到下台机器，但是这个压力总量还是没有减少啊！下一台还是可能崩溃，所以这个架构还有漏洞！不过这个文章已经太长了，后续解决吧。

##配置文件

以下是个人测试时的使用的配置文件：

###mongos

```properties
port=10000

logpath=D:\MongoDB\repset\rep_a\mongos\logs\rep_a_mongos.log

#日志信息在日志文件中累加而不是覆盖
logappend=true


#配置数据库地址
configdb=rsconf/localhost:11000,localhost:21000
```
###configdb
```properties
port=11000

dbpath=D:\MongoDB\repset\rep_a\configdb\data

logpath=D:\MongoDB\repset\rep_a\configdb\logs\rep_a_configdb.log

#日志信息在日志文件中累加而不是覆盖
logappend=true

#代表启动日志
journal=true

#表允许通过http方式来访问jsonp格式数据
#jsonp=true

#RESTFUL
rest=true

#配置数据库
configsvr=true

#分片
replSet=rsconf
```
###rep_a/shard_a

```properties
port=10001

dbpath=D:\MongoDB\repset\rep_a\shard_a\data

logpath=D:\MongoDB\repset\rep_a\shard_a\logs\rep_a_shard_a.log


#日志信息在日志文件中累加而不是覆盖
logappend=true

#代表启动日志
journal=true

#表允许通过http方式来访问jsonp格式数据
#jsonp=true

#RESTFUL
rest=true

#启动分片
shardsvr=true

#设置副本集
replSet=rep_shard_1
```
###rep_a/shard_b
```properties
port=10002

dbpath=D:\MongoDB\repset\rep_a\shard_b\data

logpath=D:\MongoDB\repset\rep_a\shard_b\logs\rep_a_shard_b.log

#日志信息在日志文件中累加而不是覆盖
logappend=true

#代表启动日志
journal=true

#表允许通过http方式来访问jsonp格式数据
#jsonp=true

#RESTFUL
rest=true

#启动分片
shardsvr=true

#设置副本集
replSet=rep_shard_2
```
###rep_a/shard_c
```properties
port=10003

dbpath=D:\MongoDB\repset\rep_a\shard_c\data

logpath=D:\MongoDB\repset\rep_a\shard_c\logs\rep_a_shard_c.log

#日志信息在日志文件中累加而不是覆盖
logappend=true

#代表启动日志
journal=true

#表允许通过http方式来访问jsonp格式数据
#jsonp=true

#RESTFUL
rest=true

#启动分片
shardsvr=true

#设置副本集
replSet=rep_shard_3
```