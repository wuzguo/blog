---
title: Elasticsearch基础知识
date: 2017-07-28 19:20:46 
author: Zak
avatar: /blog/images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wuzguo
authorDesc: 一个追求进步的「十八线码农」
categories: 后台
tags: 
	- Elasticsearch
	- 搜索引擎
	- ELK
keywords: Elasticsearch，ELK，solr，lucene
photos:
	- /blog/images/avatar.png
description: Elasticsearch搜索引擎的基本知识和相关概念
---


###1. 基本概念

#### 索引（名词）

- 一个 *索引* 类似于传统关系数据库中的一个 *数据库* ，是一个存储关系型文档的地方。 *索引* (*index*) 的复数词为 *indices* 或 *indexes*.

#### 索引（动词）

- 索引一个文档* 就是存储一个文档到一个 *索引* （名词）中以便它可以被检索和查询到。这非常类似于 SQL 语句中的 `INSERT` 关键词，除了文档已存在时新文档会替换旧文档情况之外。

#### 倒排索引

- 关系型数据库通过增加一个 *索引* 比如一个 B树（B-tree）索引 到指定的列上，以便提升数据检索速度。Elasticsearch 和 Lucene 使用了一个叫做 *倒排索引* 的结构来达到相同的目的。

#### 集群健康

-  Elasticsearch 的集群监控信息中包含了许多的统计数据，其中最为重要的一项就是 *集群健康* ， 它在 `status` 字段中展示为 `green` 、 `yellow` 或者 `red` 。`status` 字段指示着当前集群在总体上是否工作正常。它的三种颜色含义如下：

    - `green`

      所有的主分片和副本分片都正常运行。

    - `yellow`

      所有的主分片都正常运行，但不是所有的副本分片都正常运行。（当只有一个结点的时候，状态值为 yellow）

    - `red`

      有主分片没能正常运行。

###2. 知识点

#### 默认值

- 一个搜索默认返回十条结果。格式： *GET /索引库/索引类型/_search* ，例如： *GET /megacorp/employee/_search*.
- Elasticsearch 默认按照相关性得分排序，即每个文档跟查询的匹配程度。
- ElasticSearch 的主旨是随时可用和按需扩容。 而扩容可以通过购买性能更强大（ *垂直扩容* ，或 *纵向扩容* ） 或者数量更多的服务器（ *水平扩容* ，或 *横向扩容* ）来实现。
- 一个运行中的 Elasticsearch 实例称为一个 节点，而集群是由一个或者多个拥有相同 `cluster.name` 配置的节点组成， 它们共同承担数据和负载的压力。当有节点加入集群中或者从集群中移除节点时，集群将会重新平均分布所有的数据。
- 当一个节点被选举成为 *主* 节点时， 它将负责管理集群范围内的所有变更，例如增加、删除索引，或者增加、删除节点等。 而主节点并不需要涉及到文档级别的变更和搜索等操作，所以当集群只拥有一个主节点的情况下，即使流量的增加它也不会成为瓶颈。 任何节点都可以成为主节点。我们的示例集群就只有一个节点，所以它同时也成为了主节点。
- 当你在同一台机器上启动了第二个节点时，只要它和第一个节点有同样的 `cluster.name` 配置，它就会自动发现集群并加入到其中。 但是在不同机器上启动节点的时候，为了加入到同一集群，你需要配置一个可连接到的单播主机列表。 

#### 文档元数据

- 一个文档不仅仅包含它的数据 ，也包含 *元数据* —— *有关* 文档的信息。 三个必须的元数据元素如下：

  - `_index`

    文档在哪存放 （一个 *索引* 应该是因共同的特性被分组到一起的文档集合。 例如，你可能存储所有的产品在索引 `products` 中，而存储所有销售的交易到索引 `sales` 中。 虽然也允许存储不相关的数据到一个索引中，但这通常看作是一个反模式的做法。）

  - `_type`

    文档表示的对象类别（数据可能在索引中只是松散的组合在一起，但是通常明确定义一些数据中的子分区是很有用的。 例如，所有的产品都放在一个索引中，但是你有许多不同的产品类别，比如 "electronics" 、 "kitchen" 和 "lawn-care"。）

  - `_id`

    文档唯一标识（*ID* 是一个字符串， 当它和 `_index` 以及 `_type` 组合就可以唯一确定 Elasticsearch 中的一个文档。 当你创建一个新的文档，要么提供自己的 `_id` ，要么让 Elasticsearch 帮你生成。）

#### 取回文档

- 为了从 Elasticsearch 中检索出文档 ，我们仍然使用相同的 `_index` , `_type` , 和 `_id` ，但是 HTTP 谓词 更改为 `GET` :

  `GET /website/blog/123?pretty`

- 响应体包括目前已经熟悉了的元数据元素，再加上 `_source` 字段，这个字段包含我们索引数据时发送给 Elasticsearch 的原始 JSON 文档：
  ```json
  {
  	"_index": "website",
      "_type": "blog",
      "_id": "123",
      "_version": 1,
      "found": true,
      "_source": {
          "title": "My first blog entry",
          "text": "Just trying this out...",
          "date": "2014/01/01"
      }
   }
  ```

- `GET` 请求的响应体包括 `{"found": true}` ，这证实了文档已经被找到。 如果我们请求一个不存在的文档，我们仍旧会得到一个 JSON 响应体，但是 `found` 将会是 `false` 。 此外， HTTP 响应码将会是 `404 Not Found` ，而不是 `200 OK` 。

- 我们可以通过传递 `-i` 参数给 `curl` 命令，该参数 能够显示响应的头部：

  `curl -i -XGET http://localhost:9200/website/blog/124?pretty`

- 显示响应头部的响应体现在类似这样：

   ```json
   HTTP/1.1 404 Not Found
   Content-Type: application/json; charset=UTF-8
   Content-Length: 83
   ```

- 返回文档的一部分：

   ```
   {
      "_index" : "website",
      "_type" :  "blog",
      "_id" :    "124",
      "found" :  false
   }
   ```

- 默认情况下， `GET` 请求 会返回整个文档，这个文档正如存储在 `_source` 字段中的一样。但是也许你只对其中的 `title` 字段感兴趣。单个字段能用 `_source` 参数请求得到，多个字段也能使用逗号分隔的列表来指定。

   `GET /website/blog/123?_source=title,text`

- 该 `_source` 字段现在包含的只是我们请求的那些字段，并且已经将 `date` 字段过滤掉了。

   ```json
   {
     "_index" :   "website",
     "_type" :    "blog",
     "_id" :      "123",
     "_version" : 1,
     "found" :   true,
     "_source" : {
       "title": "My first blog entry" ,
       "text":  "Just trying this out..."
      }
   }  
   ```

- 或者，如果你只想得到 `_source` 字段，不需要任何元数据，你能使用 `_source` 端点：

   `GET /website/blog/123/_source`

- 那么返回的的内容如下所示：

   ```json
   {
   	"title": "My first blog entry",
        "text":  "Just trying this out...",
        "date":  "2014/01/01"
   }
   ```

#### 关于冲突

- 当我们使用 index API 更新文档 ，可以一次性读取原始文档，做我们的修改，然后重新索引 整个文档 。 最近的索引请求将获胜：无论最后哪一个文档被索引，都将被唯一存储在 Elasticsearch 中。如果其他人同时更改这个文档，他们的更改将丢失。

#### 解决冲突（乐观锁）

-  当我们之前讨论 `index` ， `GET` 和 `delete` 请求时，我们指出每个文档都有一个 `_version` （版本）号，当文档被修改时版本号递增。 Elasticsearch 使用这个 `_version` 号来确保变更以正确顺序得到执行。如果旧版本的文档在新版本之后到达，它可以被简单的忽略。

-  我们可以利用 `_version` 号来确保 应用中相互冲突的变更不会导致数据丢失。我们通过指定想要修改文档的 `version` 号来达到这个目的。 如果该版本不是当前版本号，我们的请求将会失败。

#### 通过外部系统使用版本控制

- 一个常见的设置是使用其它数据库作为主要的数据存储，使用 Elasticsearch 做数据检索， 这意味着主数据库的所有更改发生时都需要被复制到 Elasticsearch ，如果多个进程负责这一数据同步，你可能遇到类似于之前描述的并发问题。

-  如果你的主数据库已经有了版本号 — 或一个能作为版本号的字段值比如 `timestamp` — 那么你就可以在 Elasticsearch 中通过增加 `version_type=external` 到查询字符串的方式重用这些相同的版本号， 版本号必须是大于零的整数， 且小于 `9.2E+18` — 一个 Java 中 `long` 类型的正值。

-  外部版本号的处理方式和我们之前讨论的内部版本号的处理方式有些不同， Elasticsearch 不是检查当前 `_version` 和请求中指定的版本号是否相同， 而是检查当前 `_version` 是否 *小于* 指定的版本号。 如果请求成功，外部的版本号作为文档的新 `_version` 进行存储。

-   外部版本号不仅在索引和删除请求是可以指定，而且在 *创建* 新文档时也可以指定。例如，要创建一个新的具有外部版本号 `5` 的博客文章，我们可以按以下方法进行：

   ```json
   PUT /website/blog/2?version=5&version_type=external
   {
     "title": "My first external blog entry",
     "text":  "Starting to get the hang of this..."
   }
   ```


#### Groovy 脚本编程

- 您可以通过设置集群中的所有节点的 `config/elasticsearch.yml`文件来禁用动态 Groovy 脚本：

  `script.groovy.sandbox.enabled: false`

- 这将关闭 Groovy 沙盒，从而防止动态 Groovy 脚本作为请求的一部分被接受， 或者从特殊的 `.scripts` 索引中被检索。当然，你仍然可以使用存储在每个节点的 `config/scripts/` 目录下的 Groovy 脚本。

#### 取回多个文档

-  Elasticsearch 的速度已经很快了，但甚至能更快。 将多个请求合并成一个，避免单独处理每个请求花费的网络时延和开销。 如果你需要从 Elasticsearch 检索很多文档，那么使用 *multi-get* 或者 `mget` API 来将这些检索请求放在一个请求中，将比逐个文档请求更快地检索到全部文档。

- `mget` API 要求有一个 `docs` 数组作为参数，每个 元素包含需要检索文档的元数据， 包括 `_index` 、 `_type` 和 `_id` 。如果你想检索一个或者多个特定的字段，那么你可以通过 `_source` 参数来指定这些字段的名字：

  ```json
  GET /_mget
  {
     "docs" : [
        {
           "_index" : "website",
           "_type" :  "blog",
           "_id" :    2
        },
        {
           "_index" : "website",
           "_type" :  "pageviews",
           "_id" :    1,
           "_source": "views"
        }
     ]
  }
  ```

-  该响应体也包含一个 `docs` 数组 ， 对于每一个在请求中指定的文档，这个数组中都包含有一个对应的响应，且顺序与请求中的顺序相同。 其中的每一个响应都和使用单个 [`get`request](https://www.elastic.co/guide/cn/elasticsearch/guide/current/get-doc.html) 请求所得到的响应体相同：

  ```json
  {
  	"docs" : [
        {
           "_index" :   "website",
           "_id" :      "2",
           "_type" :    "blog",
           "found" :    true,
           "_source" : {
              "text" :  "This is a piece of cake...",
              "title" : "My first external blog entry"
           },
           "_version" : 10
        },
        {
           "_index" :   "website",
           "_id" :      "1",
           "_type" :    "pageviews",
           "found" :    true,
           "_version" : 2,
           "_source" : {
              "views" : 2
           }
        }
     ]
  }
  ```
  - 批量操作：

     通过批量索引典型文档，并不断增加批量大小进行尝试。 当性能开始下降，那么你的批量大小就太大了。一个好的办法是开始时将 1,000 到 5,000 个文档作为一个批次, 如果你的文档非常大，那么就减少批量的文档个数。

    密切关注你的批量请求的物理大小往往非常有用，一千个 1KB 的文档是完全不同于一千个 1MB 文档所占的物理大小。 一个好的批量大小在开始处理后所占用的物理大小约为 5-15 MB。

  - 分片路由：

    当索引一个文档的时候，文档会被存储到一个主分片中。 Elasticsearch 如何知道一个文档应该存放到哪个分片中呢？当我们创建文档时，它如何决定这个文档应当被存储在分片 `1` 还是分片 `2` 中呢？

    首先这肯定不会是随机的，否则将来要获取文档的时候我们就不知道从何处寻找了。实际上，这个过程是根据下面这个公式决定的：

    `shard = hash(routing) % number_of_primary_shards`

     `routing` 是一个可变值，默认是文档的 `_id` ，也可以设置成一个自定义的值。 `routing` 通过 hash 函数生成一个数字，然后这个数字再除以 `number_of_primary_shards` （主分片的数量）后得到 **余数** 。这个分布在 `0` 到 `number_of_primary_shards-1` 之间的余数，就是我们所寻求的文档所在分片的位置。

    这就解释了为什么我们要在创建索引的时候就确定好主分片的数量 并且永远不会改变这个数量：因为如果数量变化了，那么所有之前路由的值都会无效，文档也再也找不到了。

     所有的文档 API（ `get` 、 `index` 、 `delete` 、 `bulk` 、 `update` 以及 `mget` ）都接受一个叫做 `routing` 的路由参数 ，通过这个参数我们可以自定义文档到分片的映射。一个自定义的路由参数可以用来确保所有相关的文档——例如所有属于同一个用户的文档——都被存储到同一个分片中。我们也会在[*扩容设计*](https://www.elastic.co/guide/cn/elasticsearch/guide/current/scale.html)这一章中详细讨论为什么会有这样一种需求。