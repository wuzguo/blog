---
title: Elasticsearch搜索引擎的搜索相关知识点
date: 2017-08-01 10:34:13 
author: wúzguó
avatar: /blog/images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wzguo
authorDesc: 一个自强不息，艰苦奋斗的「十八线码农」
categories: 后台
tags: 
	- Elasticsearch
	- 搜索引擎
	- ELK
keywords: Elasticsearch，ELK，solr，lucene
photos:
	- /blog/images/avatar.png
description: 主要是讲解Elasticsearch搜索引擎的搜索相关的知识
---


### 搜索相关知识点

- Elasticsearch 不只会*存储（stores）* 文档，为了能被搜索到也会为文档添加*索引（indexes）* ，这也是为什么我们使用结构化的 JSON 文档，而不是无结构的二进制数据。

#### 搜索（search） 可以做到

  - 在类似于 `gender` 或者 `age` 这样的字段 上使用结构化查询，`join_date` 这样的字段上使用排序，就像SQL的结构化查询一样。
  - 全文检索，找出所有匹配关键字的文档并按照*相关性（relevance）* 排序后返回结果。
  - 以上二者兼而有之。

- 很多搜索都是开箱即用的，为了充分挖掘 Elasticsearch 的潜力，你需要理解以下三个概念：

  - *映射（Mapping）*

    描述数据在每个字段内如何存储

  - *分析（Analysis）*

    全文是如何处理使之可以被搜索的

  - *领域特定查询语言（Query DSL）*

    Elasticsearch 中强大灵活的查询语言


#### 分析与分线器

- 分析 包含下面的过程：

  首先，将一块文本分成适合于倒排索引的独立的 词条，之后，将这些词条统一化为标准格式以提高它们的“可搜索性”，或者 recall分析器执行上面的工作。 分析器 实际上是将三个功能封装到了一个包里：

  - 字符过滤器

    首先，字符串按顺序通过每个 字符过滤器 。他们的任务是在分词前整理字符串。一个字符过滤器可以用来去掉HTML，或者将 & 转化成 `and`。

  - 分词器

    其次，字符串被 分词器 分为单个的词条。一个简单的分词器遇到空格和标点的时候，可能会将文本拆分成词条。

  - Token 过滤器

    最后，词条按顺序通过每个 token 过滤器 。这个过程可能会改变词条（例如，小写化 Quick ），删除词条（例如， 像 a`， `and`， `the 等无用词），或者增加词条（例如，像 jump 和 leap 这种同义词）。

- Elasticsearch提供了开箱即用的字符过滤器、分词器和token 过滤器。 这些可以组合起来形成自定义的分析器以用于不同的目的。我们会在 自定义分析器 章节详细讨论。

#### 内置分析器

- Elasticsearch还附带了可以直接使用的预包装的分析器。 接下来我们会列出最重要的分析器。为了证明它们的差异，我们看看每个分析器会从下面的字符串得到哪些词条：

  `"Set the shape to semi-transparent by calling set_trans(5)"`

#### 标准分析器

- 标准分析器是Elasticsearch默认使用的分析器。它是分析各种语言文本最常用的选择。它根据 [Unicode 联盟](http://www.unicode.org/reports/tr29/) 定义的 *单词边界* 划分文本。删除绝大部分标点。最后，将词条小写。它会产生`set, the, shape, to, semi, transparent, by, calling, set_trans, 5`

#### 简单分析器

- 简单分析器在任何不是字母的地方分隔文本，将词条小写。它会产生`set, the, shape, to, semi, transparent, by, calling, set, trans`

#### 空格分析器

- 空格分析器在空格的地方划分文本。它会产生`Set, the, shape, to, semi-transparent, by, calling, set_trans(5)`

#### 语言分析器

- 特定语言分析器可用于 [很多语言](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-lang-analyzer.html)。它们可以考虑指定语言的特点。例如， `英语` 分析器附带了一组英语无用词（常用单词，例如 `and` 或者 `the` ，它们对相关性没有多少影响），它们会被删除。 由于理解英语语法的规则，这个分词器可以提取英语单词的 *词干* 。`英语` 分词器会产生下面的词条：`set, shape, semi, transpar, call, set_tran, 5`注意看 `transparent`、 `calling` 和 `set_trans` 已经变为词根格式。

#### 测试分析器

- 有些时候很难理解分词的过程和实际被存储到索引中的词条，特别是你刚接触Elasticsearch。为了理解发生了什么，你可以使用 `analyze` API 来看文本是如何被分析的。在消息体里，指定分析器和要分析的文本：

  ```json
  GET /_analyze
  {
    "analyzer": "standard",
    "text": "Text to analyze"
  }
  ```

- 结果中每个元素代表一个单独的词条：

  ```json
  {
  	"tokens": [
          {
             "token":        "text",
             "start_offset": 0,
             "end_offset":   4,
             "type":         "<ALPHANUM>",
             "position":     1
          },
          {
             "token":        "to",
             "start_offset": 5,
             "end_offset":   7,
             "type":         "<ALPHANUM>",
             "position":     2
          },
          {
             "token":        "analyze",
             "start_offset": 8,
             "end_offset":   15,
             "type":         "<ALPHANUM>",
             "position":     3
          }
      ]
  }
  ```

- token` 是实际存储到索引中的词条。 `position` 指明词条在原始文本中出现的位置。 `start_offset` 和 `end_offset` 指明字符在原始字符串中的位置。

### 核心简单域类型

#### Elasticsearch 支持 如下简单域类型

- 字符串: `string`
- 整数 : `byte`, `short`, `integer`, `long`

- 浮点数: `float`, `double`

- 布尔型: `boolean`

- 日期: `date`
- 当你索引一个包含新域的文档--之前未曾出现-- Elasticsearch 会使用 [*动态映射*](https://www.elastic.co/guide/cn/elasticsearch/guide/current/dynamic-mapping.html) ，通过JSON中基本数据类型，尝试猜测域类型，使用如下规则：

| **JSON type**          | **域 type** |
| ---------------------- | ---------- |
| 布尔型: `true` 或者 `false` | `boolean`  |
| 整数: `123`              | `long`     |
| 浮点数: `123.45`          | `double`   |
| 字符串，有效日期: `2014-09-15` | `date`     |
| 字符串: `foo bar`         | `string`   |

#### 自定义域映射

- 尽管在很多情况下基本域数据类型 已经够用，但你经常需要为单独域自定义映射 ，特别是字符串域。自定义映射允许你执行下面的**操作**

  - 全文字符串域和精确值字符串域的区别
  - 使用特定语言分析器
  - 优化域以适应部分匹配
  - 指定自定义数据格式
  - 还有更多

- 域最重要的属性是 `type` 。对于不是 `string` 的域，你一般只需要设置 `type` ：

  ```json
  {	
      "number_of_clicks": {
          "type": "integer"
      }
  }    
  ```

- 默认， `string` 类型域会被认为包含全文。就是说，它们的值在索引前，会通过 一个分析器，针对于这个域的查询在搜索前也会经过一个分析器。

- `string` 域映射的两个最重要 属性是 `index` 和 `analyzer` 。

#### index

- `index` 属性控制怎样索引字符串。它可以是下面三个值：

  - `analyzed`

    首先分析字符串，然后索引它。换句话说，以全文索引这个域。

  - `not_analyzed`

    索引这个域，所以可以搜索到它，但索引指定的精确值。不对它进行分析。

  - `no`

    `Don’t index this field at all `不索引这个域。这个域不会被搜索到。

- `string` 域 `index` 属性默认是 `analyzed` 。如果我们想映射这个字段为一个精确值，我们需要设置它为 `not_analyzed` ：

  ```json
  {
    "tag": {
        "type":     "string",
        "index":    "not_analyzed"
    }
  }
  ```

- 其他简单类型（例如 `long` ， `double` ， `date` 等）也接受 `index` 参数，但有意义的值只有 `no` 和 `not_analyzed` ， 因为它们永远不会被分析。

#### analyzer

- 对于 `analyzed` 字符串域，用 `analyzer` 属性指定在搜索和索引时使用的分析器。默认， Elasticsearch 使用 `standard` 分析器， 但你可以指定一个内置的分析器替代它，例如 `whitespace` 、 `simple` 和 `english`：

  ```json
  {
    "tweet": {
        "type":     "string",
        "analyzer": "english"
    }
  }
  ```

- 在 [自定义分析器](https://www.elastic.co/guide/cn/elasticsearch/guide/current/custom-analyzers.html) ，我们会展示怎样定义和使用自定义分析器。

#### 查询表达式

- 查询表达式(Query DSL)是一种非常灵活又富有表现力的 查询语言。 Elasticsearch 使用它可以以简单的 JSON 接口来展现 Lucene 功能的绝大部分。

- 一个查询语句 的典型结构：

  ```json
  {
      QUERY_NAME: {
          ARGUMENT: VALUE,
          ARGUMENT: VALUE,...
      }
  }	
  ```

- 如果是针对某个字段，那么它的结构如下：

  ```json
  {
    QUERY_NAME: {
        FIELD_NAME: {
            ARGUMENT: VALUE,
            ARGUMENT: VALUE,...
        }
    }
  }
  ```

- 举个例子，你可以使用 `match` 查询语句 来查询 `tweet` 字段中包含 `elasticsearch` 的 tweet：

      {
        "match": {
            "tweet": "elasticsearch"
        }
      }

- 完整的查询请求如下：

  ```json
  GET /_search
  {
      "query": {
          "match": {
              "tweet": "elasticsearch"
          }
      }
  }
  ```

#### 组合多查询

- 为了构建类似的高级查询，你需要一种能够将多查询组合成单一查询的查询方法。你可以用 `bool` 查询来实现你的需求。这种查询将多查询组合在一起，成为用户自己想要的布尔查询。它接收以下参数：

  - `must`

    文档 *必须* 匹配这些条件才能被包含进来。

  - `must_not`

    文档 *必须不* 匹配这些条件才能被包含进来。

  - `should`

    如果满足这些语句中的任意语句，将增加 `_score` ，否则，无任何影响。它们主要用于修正每个文档的相关性得分。

  - `filter`

    *必须* 匹配，但它以不评分、过滤模式来进行。这些语句对评分没有贡献，只是根据过滤标准来排除或包含文档。

- 由于这是我们看到的第一个包含多个查询的查询，所以有必要讨论一下相关性得分是如何组合的。每一个子查询都独自地计算文档的相关性得分。一旦他们的得分被计算出来， `bool` 查询就将这些得分进行合并并且返回一个代表整个布尔操作的得分。

- 下面的查询用于查找 `title` 字段匹配 `how to make millions` 并且不被标识为 `spam` 的文档。那些被标识为 `starred` 或在2014之后的文档，将比另外那些文档拥有更高的排名。如果 _两者_ 都满足，那么它排名将更高：

  ```json
  {
      "bool": {
          "must":     { "match": { "title": "how to make millions" }},
          "must_not": { "match": { "tag":   "spam" }},
          "should": [
              { "match": { "tag": "starred" }},
              { "range": { "date": { "gte": "2014-01-01" }}}
          ]
      }
  }
  ```

#### 相关性

- 每个文档都有相关性评分，用一个正浮点数字段 `_score` 来表示。`_score` 的评分越高，相关性越高。

- 查询语句会为每个文档生成一个 `_score` 字段。评分的计算方式取决于查询类型 不同的查询语句用于不同的目的： `fuzzy` 查询会计算与关键词的拼写相似程度，`terms` 查询会计算 找到的内容与关键词组成部分匹配的百分比，但是通常我们说的 *relevance* 是我们用来计算全文本字段的值相对于全文本检索词相似程度的算法。

- Elasticsearch 的相似度算法 被定义为检索词频率/反向文档频率， *TF/IDF* ，包括以下内容：

  - 检索词频率

    检索词在该字段出现的频率？出现频率越高，相关性也越高。 字段中出现过 5 次要比只出现过 1 次的相关性高。

  - 反向文档频率

    每个检索词在索引中出现的频率？频率越高，相关性越低。检索词出现在多数文档中会比出现在少数文档中的权重更低。

  - 字段长度准则

    字段的长度是多少？长度越长，相关性越低。 检索词出现在一个短的 title 要比同样的词出现在一个长的 content 字段权重更大。

- 单个查询可以联合使用 TF/IDF 和其他方式，比如短语查询中检索词的距离或模糊查询里的检索词相似度。相关性并不只是全文本检索的专利。也适用于 yes|no 的子句，匹配的子句越多，相关性评分越高。如果多条查询子句被合并为一条复合查询语句 ，比如 bool 查询，则每个查询子句计算得出的评分会被合并到总的相关性评分中。

#### 游标查询（可以替代深分页）

- `scroll` 查询 可以用来对 Elasticsearch 有效地执行大批量的文档查询，而又不用付出深度分页那种代价。游标查询允许我们 先做查询初始化，然后再批量地拉取结果。 这有点儿像传统数据库中的 *cursor* 。

#### 索引设置

- Elasticsearch 提供了优化好的默认配置。 除非你理解这些配置的作用并且知道为什么要去修改，否则不要随意修改。

- 下面是两个 最重要的设置：

  - `number_of_shards`

    每个索引的主分片数，默认值是 `5` 。这个配置在索引创建后不能修改。

  - `number_of_replicas`

    每个主分片的副本数，默认值是 `1` 。对于活动的索引库，这个配置可以随时修改。

- 例如，我们可以创建只有 一个主分片，没有副本的小索引：

  ```json
  PUT /my_temp_index
  {
      "settings": {
          "number_of_shards" :   1,
          "number_of_replicas" : 0
      }
  }
  ```

#### 自定义分析器

- 虽然Elasticsearch带有一些现成的分析器，然而在分析器上Elasticsearch真正的强大之处在于，你可以通过在一个适合你的特定数据的设置之中组合字符过滤器、分词器、词汇单元过滤器来创建自定义的分析器。

- 在 [分析与分析器](https://www.elastic.co/guide/cn/elasticsearch/guide/current/analysis-intro.html) 我们说过，一个 *分析器* 就是在一个包里面组合了三种函数的一个包装器， 三种函数按照顺序被执行:

  - 字符过滤器

    字符过滤器 用来 `整理` 一个尚未被分词的字符串。例如，如果我们的文本是HTML格式的，它会包含像 `<p>` 或者 `<div>` 这样的HTML标签，这些标签是我们不想索引的。我们可以使用 [`html清除` 字符过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-htmlstrip-charfilter.html) 来移除掉所有的HTML标签，并且像把 `&Aacute;` 转换为相对应的Unicode字符 `Á` 这样，转换HTML实体。一个分析器可能有0个或者多个字符过滤器。

  - 分词器

    一个分析器 *必须* 有一个唯一的分词器。 分词器把字符串分解成单个词条或者词汇单元。 `标准` 分析器里使用的 [`标准` 分词器](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-standard-tokenizer.html) 把一个字符串根据单词边界分解成单个词条，并且移除掉大部分的标点符号，然而还有其他不同行为的分词器存在。例如， [`关键词` 分词器](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-keyword-tokenizer.html) 完整地输出 接收到的同样的字符串，并不做任何分词。 [`空格`分词器](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-whitespace-tokenizer.html) 只根据空格分割文本 。 [`正则` 分词器](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-pattern-tokenizer.html) 根据匹配正则表达式来分割文本 。

  - 词单元过滤器

    经过分词，作为结果的 *词单元流* 会按照指定的顺序通过指定的词单元过滤器 。词单元过滤器可以修改、添加或者移除词单元。我们已经提到过 [`lowercase` ](http://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-lowercase-tokenizer.html)和[`stop` 词过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-stop-tokenfilter.html) ，但是在 Elasticsearch 里面还有很多可供选择的词单元过滤器。 [词干过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-stemmer-tokenfilter.html) 把单词 `遏制` 为 词干。 [`ascii_folding` 过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-asciifolding-tokenfilter.html)移除变音符，把一个像 `"très"` 这样的词转换为 `"tres"` 。 [`ngram`](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-ngram-tokenfilter.html) 和 [`edge_ngram` 词单元过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-edgengram-tokenfilter.html) 可以产生 适合用于部分匹配或者自动补全的词单元。

- 在 [深入搜索](https://www.elastic.co/guide/cn/elasticsearch/guide/current/search-in-depth.html)，我们讨论了在哪里使用，以及怎样使用分词器和过滤器。但是首先，我们需要解释一下怎样创建自定义的分析器。

#### 创建一个自定义分析器

- 和我们之前配置 `es_std` 分析器一样，我们可以在 `analysis` 下的相应位置设置字符过滤器、分词器和词单元过滤器:

  ```json
  PUT /my_index
  {
      "settings": {
          "analysis": {
              "char_filter": { ... custom character filters ... },
              "tokenizer":   { ...    custom tokenizers     ... },
              "filter":      { ...   custom token filters   ... },
              "analyzer":    { ...    custom analyzers      ... }
          }
      }
  }
  ```

- 作为示范，让我们一起来创建一个自定义分析器吧，这个分析器可以做到下面的这些事:

  - 使用 `html清除` 字符过滤器移除HTML部分。

  - 使用一个自定义的 `映射` 字符过滤器把 `&` 替换为 `" 和 "` ：

    ```json
    "char_filter": {
        "&_to_and": {
            "type":       "mapping",
            "mappings": [ "&=> and "]
        }
    }
    ```

  - 使用 `标准` 分词器分词。

  - 小写词条，使用 `小写` 词过滤器处理。

  - 使用自定义 `停止` 词过滤器移除自定义的停止词列表中包含的词：

    ```json
    "filter": {
        "my_stopwords": {
            "type":        "stop",
            "stopwords": [ "the", "a" ]
        }
    }
    ```

- 我们的分析器定义用我们之前已经设置好的自定义过滤器组合了已经定义好的分词器和过滤器：

  ```json
  "analyzer": {
      "my_analyzer": {
          "type":           "custom",
          "char_filter":  [ "html_strip", "&_to_and" ],
          "tokenizer":      "standard",
          "filter":       [ "lowercase", "my_stopwords" ]
      }
  }
  ```

- 汇总起来，完整的 `创建索引` 请求 看起来应该像这样：

  ```json
  PUT /my_index
  {
    "settings": {
        "analysis": {
            "char_filter": {
                "&_to_and": {
                    "type":       "mapping",
                    "mappings": [ "&=> and "]
            }},
            "filter": {
                "my_stopwords": {
                    "type":       "stop",
                    "stopwords": [ "the", "a" ]
            }},
            "analyzer": {
                "my_analyzer": {
                    "type":         "custom",
                    "char_filter":  [ "html_strip", "&_to_and" ],
                    "tokenizer":    "standard",
                    "filter":       [ "lowercase", "my_stopwords" ]
            }}

   }}}
  ```

### 根对象

- 映射的最高一层被称为 *根对象* ，它可能包含下面几项：
  - 一个 *properties* 节点，列出了文档中可能包含的每个字段的映射
  - 各种元数据字段，它们都以一个下划线开头，例如 `_type` 、 `_id` 和 `_source`
  - 设置项，控制如何动态处理新的字段，例如 `analyzer` 、 `dynamic_date_formats`和 `dynamic_templates`
  - 其他设置，可以同时应用在根对象和其他 `object` 类型的字段上，例如 `enabled`、 `dynamic` 和 `include_in_all`

#### 动态映射

- 当 Elasticsearch 遇到文档中以前 未遇到的字段，它用 [*dynamic mapping*](https://www.elastic.co/guide/cn/elasticsearch/guide/current/mapping-intro.html) 来确定字段的数据类型并自动把新的字段添加到类型映射。

- 如果Elasticsearch是作为重要的数据存储，可能就会期望遇到新字段就会抛出异常，这样能及时发现问题。

- 幸运的是可以用 `dynamic` 配置来控制这种行为 ，可接受的选项如下：

  - `true`

    动态添加新的字段--缺省

  - `false`

    忽略新的字段

  - `strict`

    如果遇到新字段抛出异常

- 配置参数 `dynamic` 可以用在根 `object` 或任何 `object` 类型的字段上。你可以将 `dynamic` 的默认值设置为 `strict` , 而只在指定的内部对象中开启它, 例如：

  ```json
  PUT /my_index
    {
          "mappings": {
              "my_type": {
                  "dynamic":      "strict", 
                  "properties": {
                      "title":  { "type": "string"},
                      "stash":  {
                          "type":     "object",
                          "dynamic":  true 
                      }
                  }
              }
          }
      }
  ```

- 把 `dynamic` 设置为 `false` 一点儿也不会改变 `_source` 的字段内容。 `_source` 仍然包含被索引的整个JSON文档。只是新的字段不会被加到映射中也不可搜索。

#### 动态模板

- 使用 `dynamic_templates` ，你可以完全控制 新检测生成字段的映射。你甚至可以通过字段名称或数据类型来应用不同的映射。

- 每个模板都有一个名称， 你可以用来描述这个模板的用途， 一个 `mapping` 来指定映射应该怎样使用，以及至少一个参数 (如 `match`) 来定义这个模板适用于哪个字段。

- 模板按照顺序来检测；第一个匹配的模板会被启用。例如，我们给 `string` 类型字段定义两个模板：

  - `es` ：以 `_es` 结尾的字段名需要使用 `spanish` 分词器。
  - `en` ：所有其他字段使用 `english` 分词器。

- 我们将 `es` 模板放在第一位，因为它比匹配所有字符串字段的 `en` 模板更特殊：

  ```json
  PUT /my_index
  {
    "mappings": {
        "my_type": {
            "dynamic_templates": [
                { "es": {
                      "match":              "*_es", 
                      "match_mapping_type": "string",
                      "mapping": {
                          "type":           "string",
                          "analyzer":       "spanish"
                      }
                }},
                { "en": {
                      "match":              "*", 
                      "match_mapping_type": "string",
                      "mapping": {
                          "type":           "string",
                          "analyzer":       "english"
                      }
                }}
            ]
   }}}       
  ```

- `match_mapping_type` 允许你应用模板到特定类型的字段上，就像有标准动态映射规则检测的一样， (例如 `string` 或 `long`)。

- `match` 参数只匹配字段名称， `path_match` 参数匹配字段在对象上的完整路径，所以 `address.*.name` 将匹配这样的字段：

  ```json
  {
    "address": {
        "city": {
            "name": "New York"
        }
    }
  }
  ```

- `unmatch` 和 `path_unmatch`将被用于未被匹配的字段。

#### 索引别名和零停机

- 在前面提到的，重建索引的问题是必须更新应用中的索引名称。 索引别名就是用来解决这个问题的！

- 索引 *别名* 就像一个快捷方式或软连接，可以指向一个或多个索引，也可以给任何一个需要索引名的API来使用。别名 带给我们极大的灵活性，允许我们做下面这些：

  - 在运行的集群中可以无缝的从一个索引切换到另一个索引
  - 给多个索引分组 (例如， `last_three_months`)
  - 给索引的一个子集创建 `视图`

- 在后面我们会讨论更多关于别名的使用。现在，我们将解释怎样使用别名在零停机下从旧索引切换到新索引。

  有两种方式管理别名： `_alias` 用于单个操作， `_aliases` 用于执行多个原子级操作。

#### 查询语句提升权重

- 当然 `bool` 查询不仅限于组合简单的单个词 `match` 查询， 它可以组合任意其他的查询，以及其他 `bool` 查询。 普遍的用法是通过汇总多个独立查询的分数，从而达到为每个文档微调其相关度评分 `_score` 的目的。

- 假设想要查询关于 “full-text search（全文搜索）” 的文档， 但我们希望为提及 “Elasticsearch” 或 “Lucene” 的文档给予更高的 *权重* ，这里 *更高权重* 是指如果文档中出现 “Elasticsearch” 或 “Lucene” ，它们会比没有的出现这些词的文档获得更高的相关度评分 `_score` ，也就是说，它们会出现在结果集的更上面。

- 一个简单的 `bool` *查询* 允许我们写出如下这种非常复杂的逻辑：

  ```json
  GET /_search
  {
      "query": {
          "bool": {
              "must": {
                  "match": {
                      "content": { 
                          "query":    "full text search",
                          "operator": "and"
                      }
                  }
              },
              "should": [ 
                  { "match": { "content": "Elasticsearch" }},
                  { "match": { "content": "Lucene"        }}
              ]
          }
      }
  }
  ```

- `content` 字段必须包含 `full` 、 `text` 和 `search` 所有三个词。如果 `content` 字段也包含 `Elasticsearch` 或 `Lucene` ，文档会获得更高的评分 `_score` 。

- `should` 语句匹配得越多表示文档的相关度越高。目前为止还挺好。

- 但是如果我们想让包含 `Lucene` 的有更高的权重，并且包含 `Elasticsearch` 的语句比 `Lucene` 的权重更高，该如何处理?

- 我们可以通过指定 `boost` 来控制任何查询语句的相对的权重， `boost` 的默认值为 `1` ，大于 `1` 会提升一个语句的相对权重。所以下面重写之前的查询：

  ```json
  GET /_search
  {
      "query": {
          "bool": {
              "must": {
                  "match": {  
                      "content": {
                          "query":    "full text search",
                          "operator": "and"
                      }
                  }
              },
              "should": [
                  { "match": {
                      "content": {
                          "query": "Elasticsearch",
                          "boost": 3 
                      }
                  }},
                  { "match": {
                      "content": {
                          "query": "Lucene",
                          "boost": 2 
                      }
                  }}
              ]
          }
      }
  }
  ```

- 这些语句使用默认的 `boost` 值 `1` 。

- 这条语句更为重要，因为它有最高的 `boost` 值。

- 这条语句比使用默认值的更重要，但它的重要性不及 `Elasticsearch` 语句。

- `boost` 参数被用来提升一个语句的相对权重（ `boost` 值大于 `1` ）或降低相对权重（ `boost` 值处于 `0` 到 `1` 之间），但是这种提升或降低并不是线性的，换句话说，如果一个 `boost` 值为 `2` ，并不能获得两倍的评分 `_score` 。

- 相反，新的评分 `_score` 会在应用权重提升之后被 *归一化* ，每种类型的查询都有自己的归一算法，细节超出了本书的范围，所以不作介绍。简单的说，更高的 `boost` 值为我们带来更高的评分 `_score` 。

- 如果不基于 TF/IDF 要实现自己的评分模型，我们就需要对权重提升的过程能有更多控制，可以使用 [`function_score` 查询](https://www.elastic.co/guide/cn/elasticsearch/guide/current/function-score-query.html)操纵一个文档的权重提升方式而跳过归一化这一步骤。

### 注意事项

#### 深分页

- 先查后取的过程支持用 `from` 和 `size` 参数分页，但是这是 *有限制的* 。 要记住需要传递信息给协调节点的每个分片必须先创建一个 `from + size` 长度的队列，协调节点需要根据 `number_of_shards * (from + size)` 排序文档，来找到被包含在 `size` 里的文档。
- 取决于你的文档的大小，分片的数量和你使用的硬件，给 10,000 到 50,000 的结果文档深分页（ 1,000 到 5,000 页）是完全可行的。但是使用足够大的 `from` 值，排序过程可能会变得非常沉重，使用大量的CPU、内存和带宽。因为这个原因，我们强烈建议你不要使用深分页。

#### Bouncing Results

- 想象一下有两个文档有同样值的时间戳字段，搜索结果用 `timestamp` 字段来排序。 由于搜索请求是在所有有效的分片副本间轮询的，那就有可能发生主分片处理请求时，这两个文档是一种顺序， 而副本分片处理请求时又是另一种顺序。
- 这就是所谓的 *bouncing results* 问题: 每次用户刷新页面，搜索结果表现是不同的顺序。 让同一个用户始终使用同一个分片，这样可以避免这种问题， 可以设置 `preference` 参数为一个特定的任意值比如用户会话ID来解决。

#### 超时问题

- 通常分片处理完它所有的数据后再把结果返回给协同节点，协同节点把收到的所有结果合并为最终结果。这意味着花费的时间是最慢分片的处理时间加结果合并的时间。如果有一个节点有问题，就会导致所有的响应缓慢。

- 参数 `timeout` 告诉 分片允许处理数据的最大时间。如果没有足够的时间处理所有数据，这个分片的结果可以是部分的，甚至是空数据。

- 搜索的返回结果会用属性 `timed_out` 标明分片是否返回的是部分结果：

  ```json
  ...
  "timed_out":     true,  
  ...
  ```

#### 路由

- 在 [路由一个文档到一个分片中](https://www.elastic.co/guide/cn/elasticsearch/guide/current/routing-value.html) 中, 我们解释过如何定制参数 `routing` ，它能够在索引时提供来确保相关的文档，比如属于某个用户的文档被存储在某个分片上。 在搜索的时候，不用搜索索引的所有分片，而是通过指定几个 `routing` 值来限定只搜索几个相关的分片：

  `GET /_search?routing=user_1,user2`

- 这个技术在设计大规模搜索系统时就会派上用场，我们在 [*扩容设计*](https://www.elastic.co/guide/cn/elasticsearch/guide/current/scale.html) 中详细讨论它。

#### 搜索类型

- 缺省的搜索类型是 `query_then_fetch` 。 在某些情况下，你可能想明确设置 `search_type` 为 `dfs_query_then_fetch` 来改善相关性精确度：

  `GET /_search?search_type=dfs_query_then_fetch`

- 搜索类型 `dfs_query_then_fetch` 有预查询阶段，这个阶段可以从所有相关分片获取词频来计算全局词频。 我们在 [被破坏的相关度！](https://www.elastic.co/guide/cn/elasticsearch/guide/current/relevance-is-broken.html) 会再讨论它。

#### 重新索引你的数据

- 尽管可以增加新的类型到索引中，或者增加新的字段到类型中，但是不能添加新的分析器或者对现有的字段做改动。 如果你那么做的话，结果就是那些已经被索引的数据就不正确， 搜索也不能正常工作。

- 对现有数据的这类改变最简单的办法就是重新索引：用新的设置创建新的索引并把文档从旧的索引复制到新的索引。

  字段 `_source` 的一个优点是在Elasticsearch中已经有整个文档。你不必从源数据中重建索引，而且那样通常比较慢。

  为了有效的重新索引所有在旧的索引中的文档，用 [*scroll*](https://www.elastic.co/guide/cn/elasticsearch/guide/current/scroll.html) 从旧的索引检索批量文档 ， 然后用 [`bulk` API](https://www.elastic.co/guide/cn/elasticsearch/guide/current/bulk.html) 把文档推送到新的索引中。

- 从Elasticsearch v2.3.0开始， [Reindex API](https://www.elastic.co/guide/en/elasticsearch/reference/master/docs-reindex.html) 被引入。它能够对文档重建索引而不需要任何插件或外部工具。

#### 批量重新索引

- 同时并行运行多个重建索引任务，但是你显然不希望结果有重叠。正确的做法是按日期或者时间 这样的字段作为过滤条件把大的重建索引分成小的任务：

      GET /old_index/_search?scroll=1m
      {
          "query": {
              "range": {
                  "date": {
                      "gte":  "2014-01-01",
                      "lt":   "2014-02-01"
                  }
              }
          },
          "sort": ["_doc"],
          "size":  1000
      }

### 配置参数

#### 禁止自动创建索引

- `config/elasticsearch.yml` 的每个节点下添加下面的配置：

  `action.auto_create_index: false`

- 我们会在之后讨论你怎么用 索引模板 来预配置开启自动创建索引。这在索引日志数据的时候尤其有用：你将日志数据索引在一个以日期结尾命名的索引上，子夜时分，一个预配置的新索引将会自动进行创建。

#### Groovy 脚本编程

- 您可以通过设置集群中的所有节点的 `config/elasticsearch.yml`文件来禁用动态 Groovy 脚本：

  `script.groovy.sandbox.enabled: false`

- 这将关闭 Groovy 沙盒，从而防止动态 Groovy 脚本作为请求的一部分被接受， 或者从特殊的 `.scripts` 索引中被检索。当然，你仍然可以使用存储在每个节点的 `config/scripts/` 目录下的 Groovy 脚本。

#### 避免意外的大量删除数据

`action.destructive_requires_name: true`

- 这个设置使删除只限于特定名称指向的数据, 而不允许通过指定 `_all` 或通配符来删除指定索引库。你同样可以通过 [Cluster State API](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_changing_settings_dynamically.html) 动态的更新这个设置。