## 1、ES 简介

##### 1）定义

1. ES是elaticsearch简写， Elasticsearch是一个**开源的高扩展的分布式全文检索引擎**，它可以**近乎实时的存储**、检索数据；本身扩展性很好，可以扩展到上百台服务器，**处理PB级别的数据**。 
2. Elasticsearch也使用Java开发并使用Lucene作为其核心来实现所有索引和搜索的功能，但是它的目的是通过简单的RESTful API来隐藏Lucene的复杂性，从而让全文搜索变得简单。

##### 2）特点和优势

1. 分布式实时文件存储，可将每一个字段存入索引，使其可以被检索到。 
2. **近乎实时分析**的分布式搜索引擎。 
3. 分布式：索引分拆成多个分片，每个分片可有零个或多个副本。集群中的每个数据节点都可承载一个或多个分片，并且协调和处理各种操作； 
4. 负载再平衡和路由在大多数情况下自动完成。 
5. 可以扩展到上百台服务器，处理PB级别的结构化或非结构化数据（官网是这么说的）。也可以运行在单台PC上（已测试）。 
6. 支持插件机制，分词插件、同步插件、Hadoop插件、可视化插件等。

#####   3）Elasticsearch 在业界的应用案例

​	国内现在有大量的公司都在使用 Elasticsearch，包括携程、滴滴、今日头条、饿了么、360安全、小米、vivo等诸多知名公司。

![1596675329800](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1596675329800.png)

除了搜索之外，结合Kibana、Logstash、Beats，Elastic Stack还被广泛运用在大数据近实时分析领域，包括***日志分析、指标监控、信息安全***等多个领域。它可以帮助你探索海量结构化、非结构化数据，按需创建可视化报表，对监控数据设置报警阈值，甚至通过使用机器学习技术，自动识别异常状况。

1.京东到家订单中心

​	京东到家订单中心系统业务中，无论是外部商家的订单生产，或是内部上下游系统的依赖，订单查询的调用量都非常大，造成了订单数据读多写少的情况。京东到家的订单数据存储在MySQL中，但显然只通过DB来支撑大量的查询是不可取的，同时对于一些复杂的查询，Mysql支持得不够友好，所以订单中心系统使用了Elasticsearch来承载订单查询的主要压力。

2.携程Elasticsearch应用

日志单集群有120个data node，运行于70台物理服务器上。数据规模如下:

- 单日索引数据条数600亿，新增索引文件25TB (含一个复制片则为50TB)
- 业务高峰期峰值索引速率维持在百万条/秒
- 历史数据保留时长根据业务需求制定，从10天 - 90天不等
- 集群共3441个索引、17000个分片、数据总量约9300亿, 磁盘总消耗1PB

3.去哪儿：订单中心基于elasticsearch 的应用

​	15年去哪儿网酒店日均订单量达到30w+，随着多平台订单的聚合日均订单能达到100w左右。原来采用的热表分库方式，即将最近6个月的订单的放置在一张表中，将历史订单放在在history表中。history表存储全量的数据，当用户查询的下单时间跨度超过6个月即查询历史订单表，此分表方式热表的数据量为4000w左右，当时能解决的问题。但是显然不能满足携程艺龙订单接入的需求。如果继续按照热表方式，数据量将超过1亿条。全量数据表保存2年的可能就超过4亿的数据量。所以寻找有效途径解决此问题迫在眉睫。由于对这预计4亿的数据量还需按照预定日期、入住日期、离店日期、订单号、联系人姓名、电话、酒店名称、订单状态……等多个条件查询。所以简单按照某一个维度进行分表操作没有意义。Elasticsearch分布式搜索储存集群的引入，就是为了解决订单数据的存储与搜索的问题。

对订单模型进行抽象和分类，将常用搜索字段和基础属性字段剥离。DB做分库分表，存储订单详情；Elasticsearch存储搜索字段。

订单复杂查询直接走Elasticsearch，基于OrderNo的简单查询走DB。

## 2、ES 基本概念

##### 1）节点（Node）

​        运行了**单个实例的ES主机称为节点**，它是集群的一个成员，可以存储数据、参与集群索引及搜索操作。节点通过为其配置的ES集群名称确定其所要加入的集群。

##### 2）集群（cluster）

​        ES可以作为一个独立的单个搜索服务器。不过，一般为了处理大型数据集，实现容错和高可用性，ES可以运行在许多互相合作的服务器上。这些服务器的集合称为集群。

##### 3）分片（Shard）

​        ES的“分片(shard)”机制可将一个索引内部的数据分布地存储于多个节点，它通过**将一个索引切分为多个**底层物理的Lucene索引完成**索引数据的分割存储**功能，这每一个物理的Lucene索引称为一个分片(shard)。

​        这样的好处是可以**把一个大的索引拆分成多个，分布到不同的节点上**。降低单服务器的压力，构成分布式搜索，**提高整体检索的效率（分片数的最优值与硬件参数和数据量大小有关）。**分片的数量**只能在索引创建前指定，并且索引创建后不能更改。**

##### 4）副本（Replica）

​        副本是一个分片的**精确复制**，每个分片可以有零个或多个副本。副本的作用一是**提高系统的容错性**，当某个节点某个分片损坏或丢失时可以从副本中恢复。二是**提高es的查询效率**，es会自动对搜索请求进行负载均衡。

## 3、ES的数据架构

##### 1）索引（index）

​        ES将数据存储于一个或多个索引中，索引是具有类似特性的文档的集合。类比传统的关系型数据库领域来说，**索引相当于SQL中的一个数据库。**

##### 2）类型（Type） （弃用）

类型是索引内部的逻辑分区(category/partition)，然而其意义完全取决于用户需求。因此，一个索引内部可定义一个或多个类型(type)。一般来说，类型就是为那些拥有相同的域的文档做的预定义。类比传统的关系型数据库领域来说，**类型相当于“表”**。

**特别注意的是，**根据官网信息：在Elasticsearch 6.0.0或更高版本中创建的索引**只能包含一个映射类型**。在5.x中创建的具有多种映射类型的索引将继续像在Elasticsearch 6.x中一样工作。**类型将在Elasticsearch 7.0.0中的API中弃用，并在8.0.0中完全删除。**

##### 3）文档（Document）

​        文档是Lucene索引和搜索的原子单位，它是包含了一个或多个域的容器，基于JSON格式进行表示。文档由一个或多个域组成，每个域拥有一个名字及一个或多个值，有多个值的域通常称为“多值域”。每个文档可以存储不同的域集，但同一类型下的文档至应该有某种程度上的相似之处。**相当于mysql表中的row**。

##### 4）映射（Mapping）

​        映射是定义文档及其包含的字段如何存储和索引的过程。例如，使用映射来定义：

- - 哪些字符串字段应该被视为全文字段。
  - 哪些字段包含数字、日期或地理位置。
  - 文档中所有字段的值是否应该被索引到catch-all _all字段中。
  - 日期值的格式。
  - 用于控制动态添加字段的映射的自定义规则。

​        **每个索引都有一个映射类型，它决定了文档的索引方式。**

##### 5）与 mysql 的对比

## ![1596675884813](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1596675884813.png)



## 4、索引的CURD

##### 1）新增

```json
PUT /tehero_index
{
   "mappings": {
     "doc":{
      "properties":{
       "id":{
         "type":"integer"
       },
       "name":{
         "type":"text"
       },
       "createAt":{
         "type":"date"
       }
     }
     }

   }
}
```

```json
# 返回值如下：
{
  "acknowledged": true, # 是否在集群中成功创建了索引
  "shards_acknowledged": true,
  "index": "tehero_index"
}
```

##### **2）查询**

```
GET /tehero_index  # 索引名
#可以同时检索多个索引或所有索引
如：GET /*    GET /tehero_index,other_index

GET /_cat/indices?v  #查看所有 index
```

结果：

```
{
  "tehero_index": {
    "aliases": {},
    "mappings": {
      "doc": {
        "properties": {
          "createAt": {
            "type": "date"
          },
          "id": {
            "type": "integer"
          },
          "name": {
            "type": "text"
          }
        }
      }
    },
    "settings": {
      "index": {
        "creation_date": "1596677819989",
        "number_of_shards": "5",
        "number_of_replicas": "1",
        "uuid": "EVsf6yYKRFqB2UQ6Qmx3Tw",
        "version": {
          "created": "6010199"
        },
        "provided_name": "tehero_index"
      }
    }
  }
}
```

##### **3）修改**

> ES提供了一系列对index修改的语句，包括**副本数量的修改、新增字段、refresh_interval值的修改、索引分词器的修改、别名的修改

先学习常用的语法：

```
# 修改副本数
PUT /tehero_index/_settings
{
    "index" : {
        "number_of_replicas" : 2
    }
}

# 修改分片刷新时间,默认为1s
PUT /tehero_index/_settings
{
    "index" : {
        "refresh_interval" : "2s"
    }
}

# 新增字段 age
PUT /tehero_index/doc/_mapping
{
  "properties": {
    "age": {
      "type": "integer"
    }
  }
}
```

更新完后，我们再次查看索引配置：

```
GET /tehero_index
结果：
{
  "tehero_index": {
    "aliases": {},
    "mappings": {
      "doc": {
        "properties": {
          "age": {
            "type": "integer"
          },
          "createAt": {
            "type": "date"
          },
          "id": {
            "type": "integer"
          },
          "name": {
            "type": "text"
          }
        }
      }
    },
    "settings": {
      "index": {
        "refresh_interval": "2s",
        "number_of_shards": "5",
        "provided_name": "tehero_index",
        "creation_date": "1596677819989",
        "number_of_replicas": "2",
        "uuid": "EVsf6yYKRFqB2UQ6Qmx3Tw",
        "version": {
          "created": "6010199"
        }
      }
    }
  }
}
已经修改成功
```

##### **4）删除**

```
# 删除索引
DELETE /tehero_index
# 验证索引是否存在
HEAD tehero_index
返回：404 - Not Found
```

## 5、文档的CURD

##### **1）新增**

新增单条数据，并指定es的id 为 1

```
PUT /tehero_index/doc/1
{
  "name": "Te Hero"
}

结果：
{
  "_index": "tehero_index",
  "_type": "doc",
  "_id": "1",
  "_version": 2,
  "result": "updated",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 1,
  "_primary_term": 1
}
```



新增单条数据，使用ES自动生成id

```json
POST /tehero_index/doc
{
  "name": "Te Hero2"
}

结果：
{
  "_index": "tehero_index",
  "_type": "doc",
  "_id": "MuJ0wXMBXLJ8u-9VMRcH",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 2,
  "_primary_term": 1
}

```



我们查询数据，看下效果：

GET   /tehero_index/doc/_search

```
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": 1,
    "hits": [
      {
        "_index": "tehero_index",
        "_type": "doc",
        "_id": "MeJwwXMBXLJ8u-9VjBfs",
        "_score": 1,
        "_source": {
          "name": "Te Hero2"
        }
      },
      {
        "_index": "tehero_index",
        "_type": "doc",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name": "Te Hero"
        }
      },
      {
        "_index": "tehero_index",
        "_type": "doc",
        "_id": "MuJ0wXMBXLJ8u-9VMRcH",
        "_score": 1,
        "_source": {
          "name": "Te Hero2"
        }
      }
    ]
  }
}
```

##### **2）修改**

```
# 根据id，修改单条数据
（ps：修改语句和新增语句相同，可以理解为根据ID，存在则更新；不存在则新增）
PUT /tehero_index/doc/1
{
  "name": "Te Hero-update"
}
```



##### **3）查询**

```
# 1、根据id，获取单个数据
GET /tehero_index/doc/1
结果：
{
  "_index": "tehero_index",
  "_type": "doc",
  "_id": "1",
  "_version": 4,
  "found": true,
  "_source": {
    "name": "Te Hero-update"
  }
}

# 2、获取索引下的所有数据
GET /tehero_index/_search
结果：
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": 1,
    "hits": [
      {
        "_index": "tehero_index",
        "_type": "doc",
        "_id": "MeJwwXMBXLJ8u-9VjBfs",
        "_score": 1,
        "_source": {
          "name": "Te Hero2"
        }
      },
      {
        "_index": "tehero_index",
        "_type": "doc",
        "_id": "MuJ0wXMBXLJ8u-9VMRcH",
        "_score": 1,
        "_source": {
          "name": "Te Hero2"
        }
      },
      {
        "_index": "tehero_index",
        "_type": "doc",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name": "Te Hero-update"
        }
      }
    ]
  }
}


```

##### **4）删除**

```
根据id，删除单个数据
DELETE /tehero_index/doc/1

```

##### 5）批量操作 Bulk API

```
# 批量操作
POST _bulk
{ "index" : { "_index" : "tehero_test1", "_type" : "doc", "_id" : "1" } }
{ "this_is_field1" : "this_is_index_value" }
{ "delete" : { "_index" : "tehero_test1", "_type" : "doc", "_id" : "2" } }
{ "create" : { "_index" : "tehero_test1", "_type" : "doc", "_id" : "3" } }
{ "this_is_field3" : "this_is_create_value" }
{ "update" : {"_id" : "1", "_type" : "doc", "_index" : "tehero_test1"} }
{ "doc" : {"this_is_field2" : "this_is_update_value"} }

结果
{
  "took": 101,
  "errors": false,
  "items": [
    {
      "index": {
        "_index": "tehero_test1",
        "_type": "doc",
        "_id": "1",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 0,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "delete": {
        "_index": "tehero_test1",
        "_type": "doc",
        "_id": "2",
        "_version": 1,
        "result": "not_found",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 0,
        "_primary_term": 1,
        "status": 404
      }
    },
    {
      "create": {
        "_index": "tehero_test1",
        "_type": "doc",
        "_id": "3",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 0,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "update": {
        "_index": "tehero_test1",
        "_type": "doc",
        "_id": "1",
        "_version": 2,
        "result": "updated",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 1,
        "_primary_term": 1,
        "status": 200
      }
    }
  ]
}


```

> 注：POST _bulk 都做了哪些操作呢？

> 1、若索引“tehero_test1”不存在，则创建一个名为“tehero_test1”的 index，同时若id = 1 的文档存在，则更新；不存在则插入一条 id=1 的文档；

> 2、删除 id=2 的文档；

> 3、插入 id=3 的文档；若文档已存在，则报异常；

> 4、更新 id = 1 的文档。



## 6、ElasticSearch倒排索引与分词*

#####  1）倒排索引概念

​        1.百度百科：倒排索引源于实际应用中需要根据属性的值来查找记录。这种索引表中的每一项都包括一个属性值和具有该属性值的各记录的地址。由于不是由记录来确定属性值，而是由属性值来确定记录的位置，因而称为倒排索引(inverted index)。带有倒排索引的文件我们称为倒排[索引文件](https://baike.baidu.com/item/索引文件)，简称[倒排文件](https://baike.baidu.com/item/倒排文件/4137688)(inverted file)。

​        2.以书举例：

​          目录页   ===>   正排索引

​          索引页   ===>   倒排索引

​		3.正排索引和倒排索引：

​		 A、正排索引：**文档ID**到**文档内容、单词**的关联关系	

| 文档ID | 文档内容                        |
| ------ | ------------------------------- |
| 1      | elasticsearch是最流行的搜索引擎 |
| 2      | php是世界上最好的语言           |
| 3      | 搜索引擎是如何产生的            |

B、倒排索引：**单词**到**文档ID**的关联关系

| 单词          | 文档ID列表 |
| ------------- | ---------- |
| elasticsearch | 1          |
| 流行          | 1          |
| 搜索引擎      | 1，3       |
| php           | 2          |
| 世界          | 2          |
| 最好          | 2          |
| 语言          | 2          |
| 如何          | 3          |
| 诞生          | 3          |

  C、查询包含“搜索引擎”的文档流程：

​            a、通过倒排索引，获得对应的文档ID：1，3；

​            b、通过文档ID，查询完整内容；

​            c、返回最终结果。



##### 2）倒排索引是怎么工作的

> 主要包括2个过程：1、创建倒排索引；2、倒排索引搜索

2.1 创建倒排索引

> 还是使用上面的例子。先对**文档的内容**进行分词，形成一个个的 **token**，也就是 ***单词***，然后保存这些 token 与文档的对应关系。

2.2 倒排索引搜索

> 搜索示例1：“学习索引”

- 先分词，得到两个Token：“学习”、“索引”

- 然后去倒排索引中进行匹配

  

##### 3）倒排索引详解

   1、组成：

​          A、单词词典（Term Dictionary）

​          B、倒排列表（Posting List）

​    2、单词词典：

​          记录所有文档的单词，一般都比较大，记录了**单词**到**倒排列表**的关联信息，一般使用B + Tree实现。

​    3、倒排列表：

​          记录单词对应的文档集合，由倒排索引项Posting List组成。

​           A、倒排索引项主要包含：

​            a、文档ID。用于获取原始信息。

​            b、词频TF。记录该单词在该文档中的出现次数，用于计算相关性得分。

​            c、位置Position。记录单词在文档中的分词位置(多个)，用于词语搜索。

​            d、偏移Offset。记录单词在文档的开始和结束位置，用于高亮显示。

​          B、如上的数据中，“搜索引擎”的Posting List为：

| DocID | TF   | Position | Offset  |
| ----- | ---- | -------- | ------- |
| 1     | 1    | 2        | <18,22> |
| 3     | 1    | 0        | <0,4>   |

#####  4）分词介绍

​        指：将文本转换成一系列单词Term/Token的过程，也可称作文本分析，ES中叫作：**Analysis**。

​        ※分词器(Analyzer)：

​          ES中专门处理分词的组件，**组成**和**执行顺序**如下：

​            A、Character Filters(多个)。针对原始文本进行处理，如：去除html特使标记符</p>等。

​            B、Tokenizer(一个)。将原始文本按一定规则划分为单词。

​            C、Token Filters(多个)。针对Tokenizer处理的单词进行再加工，如：转小写、删除、新增等。

#####   5）分词API

​        ES提供了一个测试分词的API接口，使用endpoint：_analyze。

​        并且：可以指定分词器进行测试，还可以直接指定索引中的字段，甚至自定义分词器进行测试。

​        1）指定分词器：

```json
#指定分词器进行分词测试
POST _analyze
{
    "analyzer":"standard",
    "text":"hello world!"
}
结果：
{
  "tokens": [
    {
      "token": "hello",
      "start_offset": 0,
      "end_offset": 5,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "world",
      "start_offset": 6,
      "end_offset": 11,
      "type": "<ALPHANUM>",
      "position": 1
    }
  ]
}
```

​		

2）直接指定索引中字段：

```json
#直接指定索引中字段  使用username字段的分词方式对text进行分词。
POST test_index/_analyze
{
 "field":"username",
 "text":"hello world!"
}
结果：
{
  "tokens": [
    {
      "token": "hello",
      "start_offset": 0,
      "end_offset": 5,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "world",
      "start_offset": 6,
      "end_offset": 11,
      "type": "<ALPHANUM>",
      "position": 1
    }
  ]
}

```

3）自定义分词器：

```json
#自定义分词器，自定义Tokenizer、filter、等进行分词，举例：
POST _analyze
{
 "tokenizer":"standard",
 "filter":["lowercase"],
 "text":"Hello World!"
}

结果：
{
  "tokens": [
    {
      "token": "hello",
      "start_offset": 0,
      "end_offset": 5,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "world",
      "start_offset": 6,
      "end_offset": 11,
      "type": "<ALPHANUM>",
      "position": 1
    }
  ]
}
```



​    5、ES自带分词器：

| 1）standard(默认分词器) | 按词划分、支持多语言、小写处理。                             |
| ----------------------- | ------------------------------------------------------------ |
| 2）simple               | 按非字母划分、小写处理。                                     |
| 3）whitespace           | 按空格划分。                                                 |
| 4）stop                 | 按非字母划分、小写处理、按StopWord（语气助词：the、an、的、这等）处理。 |
| 5）keyword              | 不分词，作为一个单词输出。                                   |
| 6）pattern              | 通过正则表达式自定义分隔符，默认\w+，即：非字词的符号作为分隔符。 |
| 7）Language             | 另外还有30+常见语言的分词器（如：arabic、armenian、basque等） |

  6、中文分词器：

```
#使用默认分词器
POST _analyze
  {
  "analyzer":"standard",
  "text":"南京市长江大桥欢迎你"
  }
  
  结果：
  {
  "tokens": [
    {
      "token": "南",
      "start_offset": 0,
      "end_offset": 1,
      "type": "<IDEOGRAPHIC>",
      "position": 0
    },
    {
      "token": "京",
      "start_offset": 1,
      "end_offset": 2,
      "type": "<IDEOGRAPHIC>",
      "position": 1
    },
    {
      "token": "市",
      "start_offset": 2,
      "end_offset": 3,
      "type": "<IDEOGRAPHIC>",
      "position": 2
    },
    {
      "token": "长",
      "start_offset": 3,
      "end_offset": 4,
      "type": "<IDEOGRAPHIC>",
      "position": 3
    },
    {
      "token": "江",
      "start_offset": 4,
      "end_offset": 5,
      "type": "<IDEOGRAPHIC>",
      "position": 4
    },
    {
      "token": "大",
      "start_offset": 5,
      "end_offset": 6,
      "type": "<IDEOGRAPHIC>",
      "position": 5
    },
    {
      "token": "桥",
      "start_offset": 6,
      "end_offset": 7,
      "type": "<IDEOGRAPHIC>",
      "position": 6
    },
    {
      "token": "欢",
      "start_offset": 7,
      "end_offset": 8,
      "type": "<IDEOGRAPHIC>",
      "position": 7
    },
    {
      "token": "迎",
      "start_offset": 8,
      "end_offset": 9,
      "type": "<IDEOGRAPHIC>",
      "position": 8
    },
    {
      "token": "你",
      "start_offset": 9,
      "end_offset": 10,
      "type": "<IDEOGRAPHIC>",
      "position": 9
    }
  ]
}
```

​      中文分词的难点在于：汉语中的分词没有一个形式上的分隔符，上下文不同，分词的结果也就不同，比如交叉歧义问题：“**南京市长江大桥欢迎你**”，到底是“**南京市长/江大桥/欢迎你**”，还是“**南京市/长江大桥/欢迎你**”？

​      目前，常用的中文分词器有：

​      1）IK。实现中英文分词，支持多模式，可自定义词库，支持热更新分词词典。

​      2）jieba。python中流行，支持繁体分词、并行分词，可自定义词典、词性标记等。         



  7、自定义分词：

​      通过自定义：Character Filters、Tokenizer、Token Filter实现。格式：

```json
PUT index名

{

  "setting":{

    "analysis":{

      "char_filter":{},

      "tokenizer":{},

      "filter":{},

      "analyzer":{}     

 }

  }

}
```

举例:

```json
PUT sss
{
  "settings": {
    "analysis": {
      "analyzer": {
        "custom_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase"
          ]
        }
      }
    }
  },
  "mappings": {
    "doc": {
      "properties": {
        "my_text": {
          "type": "text",
          "analyzer": "custom_analyzer"
        }
      }
    }
  }
}
```

​                   

## 7、ik分词*

#####   1) ik_max_word：细颗粒度分词

> **ik_max_word: 会将文本做最细粒度的拆分**，比如会将“关注我系统学习ES”拆分为“关注，我，系统学，系统，学习，es”，**会穷尽各种可能的组合**，**适合 Term Query**；  
>
>    

    POST _analyze
    {
        "analyzer": "ik_max_word",
        "text":"南京市长江大桥欢迎你"
    }
    
    结果
    {
      "tokens": [
        {
          "token": "南京市",
          "start_offset": 0,
          "end_offset": 3,
          "type": "CN_WORD",
          "position": 0
        },
        {
          "token": "南京",
          "start_offset": 0,
          "end_offset": 2,
          "type": "CN_WORD",
          "position": 1
        },
        {
          "token": "市长",
          "start_offset": 2,
          "end_offset": 4,
          "type": "CN_WORD",
          "position": 2
        },
        {
          "token": "长江大桥",
          "start_offset": 3,
          "end_offset": 7,
          "type": "CN_WORD",
          "position": 3
        },
        {
          "token": "长江",
          "start_offset": 3,
          "end_offset": 5,
          "type": "CN_WORD",
          "position": 4
        },
        {
          "token": "大桥",
          "start_offset": 5,
          "end_offset": 7,
          "type": "CN_WORD",
          "position": 5
        },
        {
          "token": "欢迎",
          "start_offset": 7,
          "end_offset": 9,
          "type": "CN_WORD",
          "position": 6
        },
        {
          "token": "你",
          "start_offset": 9,
          "end_offset": 10,
          "type": "CN_CHAR",
          "position": 7
        }
      ]
    }

##### 2) ik_smart：粗颗粒度分词

> **ik_smart: 会做最粗粒度的拆分**，**适合 Phrase 查询。**

```json
GET /_analyze
{
  "text": "南京市长江大桥欢迎你",
  "analyzer": "ik_smart"
}

{
  "tokens": [
    {
      "token": "南京市",
      "start_offset": 0,
      "end_offset": 3,
      "type": "CN_WORD",
      "position": 0
    },
    {
      "token": "长江大桥",
      "start_offset": 3,
      "end_offset": 7,
      "type": "CN_WORD",
      "position": 1
    },
    {
      "token": "欢迎",
      "start_offset": 7,
      "end_offset": 9,
      "type": "CN_WORD",
      "position": 2
    },
    {
      "token": "你",
      "start_offset": 9,
      "end_offset": 10,
      "type": "CN_CHAR",
      "position": 3
    }
  ]
}
```

  建议：一般情况下，为了提高搜索的效果，**需要这两种分词器配合使用**。既**建索引时用 ik_max_word 尽可能多的分词**，而**搜索时用 ik_smart 尽可能提高匹配准度**，让用户的搜索尽可能的准确。比如一个常见的场景，**就是搜索"进口红酒"的时候，尽可能的不要出现口红相关商品或者让口红不要排在前面。**

> 在简单学习了解了Ik分词后，我们就可以去学习es的全文查询了。

​       

##### 3) 数据准备

创建index

```json
PUT /hero_index
{
  "settings": {
    "index": {
      "number_of_shards": 1,
      "number_of_replicas": 1
    }
  },
  "mappings": {
    "doc": {
      "dynamic": false,
      "properties": {
        "id": {
          "type": "integer"
        },
        "content": {
          "type": "keyword",
          "fields": {
            "ik_max_analyzer": {
              "type": "text",
              "analyzer": "ik_max_word",
              "search_analyzer": "ik_max_word"
            },
            "ik_smart_analyzer": {
              "type": "text",
              "analyzer": "ik_smart"
            }
          }
        },
        "createAt": {
          "type": "date"
        }
      }
    }
  }
}

结果：
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "hero_index"
}       
```

解释下content字段的映射：【就是**一个字段配置多个分词器**】

```
        "content": {
          "type": "keyword", # 默认为 keyword类型
          "fields": {
            "ik_max_analyzer": { # 创建名为 ik_max_analyzer 的子字段
              "type": "text",
              "analyzer": "ik_max_word", # 字段ik_max_analyzer 的倒排序索引分词器为ik_max_word
              "search_analyzer": "ik_max_word" # 检索关键词的分词器为ik_max_word
            },
            "ik_smart_analyzer": {  # 创建名为 ik_smart_analyzer的子字段
              "type": "text",
              "analyzer": "ik_smart" # 字段ik_smart_analyzer 的倒排序索引分词器为ik_smart
                 # 字段ik_smart_analyzer 的检索关键词的分词器默认为ik_smart
            }
          }
        }
```

> 相比粗暴的用不同的字段去实现配置不同的分词器而言，**一个字段配置多个分词器**在数据的存储和操作上方便许多，**只用储存一个字段，即可得到不同的分词效果。**



##### 4）Full text queries 之 match query（检索关键词会被分词）

match query：用于执行全文查询的标准查询，包括模糊匹配和短语或接近查询。

- 1）批量导入数据

```json
POST _bulk
{ "index" : { "_index" : "hero_index", "_type" : "doc", "_id" : "1" } }
{ "id" : 1,"content":"关注我，系统学编程" }
{ "index" : { "_index" : "hero_index", "_type" : "doc", "_id" : "2" } }
{ "id" : 2,"content":"系统学编程,就关注我" }
{ "index" : { "_index" : "hero_index", "_type" : "doc", "_id" : "3" } }
{ "id" : 3,"content":"系统编程，求关注" }
```

- 2）使用 content 的默认字段检索【keword】

```json
# 1、发现查询不到结果
GET /hero_index/doc/_search
{
  "query":{
    "match":{
      "content":"系统学"
    }
  }
}

结果：
{
  "took": 39,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 0,
    "max_score": null,
    "hits": []
  }
}


# 2、查询到id = 1 的文档
GET /hero_index/doc/_search
{
  "query":{
    "match":{
      "content":"关注我，系统学编程"
    }
  }
}

结果：
{
  "took": 19,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.9808292,
    "hits": [
      {
        "_index": "hero_index",
        "_type": "doc",
        "_id": "1",
        "_score": 0.9808292,
        "_source": {
          "id": 1,
          "content": "关注我，系统学编程"
        }
      }
    ]
  }
}
```

> 分析：【语句1】发现查询不到结果，此时content**是keyword类型，是不会分词的，**所以检索词需要和内容完全一样，【语句2】查询到文档1　type:keyword不会分词， type:text: 会分词



##### 5）使用 content.ik_max_analyzer 字段检索【ik_max_word】

```
# 1、会检索出所有结果
GET /hero_index/doc/_search
{
  "query":{
    "match":{
      "content.ik_max_analyzer":"系统学"
    }
  }
}
{
  "took": 14,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": 1.0735387,
    "hits": [
      {
        "_index": "hero_index",
        "_type": "doc",
        "_id": "1",
        "_score": 1.0735387,
        "_source": {
          "id": 1,
          "content": "关注我，系统学编程"
        }
      },
      {
        "_index": "hero_index",
        "_type": "doc",
        "_id": "2",
        "_score": 1.005015,
        "_source": {
          "id": 2,
          "content": "系统学编程,就关注我"
        }
      },
      {
        "_index": "hero_index",
        "_type": "doc",
        "_id": "3",
        "_score": 0.14330196,
        "_source": {
          "id": 3,
          "content": "系统编程，求关注"
        }
      }
    ]
  }
}


# 2、改变检索分词器为ik_smart,只能检索到 文档1和文档2
GET /hero_index/doc/_search
{
  "query":{
    "match":{
     "content.ik_max_analyzer" : {
                "query" : "系统学",
                "analyzer": "ik_smart"
            }
    }
  }
}

结果：
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 0.47000363,
    "hits": [
      {
        "_index": "hero_index",
        "_type": "doc",
        "_id": "1",
        "_score": 0.47000363,
        "_source": {
          "id": 1,
          "content": "关注我，系统学编程"
        }
      },
      {
        "_index": "hero_index",
        "_type": "doc",
        "_id": "2",
        "_score": 0.44000342,
        "_source": {
          "id": 2,
          "content": "系统学编程,就关注我"
        }
      }
    ]
  }
}
```

> 分析：【语句1】能查询到所有文档，因为**检索词根据ik_max_word分词，得到Token（系统、系统学）**【语句2】的检索词**根据ik_smart分词，只能得到Toke****n（系统学），不能匹配上文档3。



```
GET /_analyze
{
 
  "analyzer": "ik_smart",
  "text":"系统学"
}

{
  "tokens": [
    {
      "token": "系统学",
      "start_offset": 0,
      "end_offset": 3,
      "type": "CN_WORD",
      "position": 0
    }
  ]
}

```

```json
GET /_analyze
{
 
  "analyzer": "ik_max_word",
  "text":"系统学"
}

{
  "tokens": [
    {
      "token": "系统学",
      "start_offset": 0,
      "end_offset": 3,
      "type": "CN_WORD",
      "position": 0
    },
    {
      "token": "系统",
      "start_offset": 0,
      "end_offset": 2,
      "type": "CN_WORD",
      "position": 1
    },
    {
      "token": "学",
      "start_offset": 2,
      "end_offset": 3,
      "type": "CN_CHAR",
      "position": 2
    }
  ]
}


```

##### 6）match的核心参数：**operator ——控制Token之间的逻辑关系，or/and**

```json
# 1、不配置，使用默认值or，得到文档1和文档2
GET /hero_index/doc/_search
{
  "query": {
    "match": {
      "content.ik_smart_analyzer": {
        "query": "系统学es"
      }
    }
  }
}
{
  "took": 13,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 0.48527452,
    "hits": [
      {
        "_index": "hero_index",
        "_type": "doc",
        "_id": "1",
        "_score": 0.48527452,
        "_source": {
          "id": 1,
          "content": "关注我，系统学编程"
        }
      },
      {
        "_index": "hero_index",
        "_type": "doc",
        "_id": "2",
        "_score": 0.44217452,
        "_source": {
          "id": 2,
          "content": "系统学编程,就关注我"
        }
      }
    ]
  }
}

```

```json
# 2、and，查询不到结果

GET /hero_index/doc/_search
{
  "query": {
    "match": {
      "content.ik_smart_analyzer": {
        "query": "系统学es",
        "operator":"and"
      }
    }
  }
}

结果：
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 0,
    "max_score": null,
    "hits": []
  }
}
```

> 分析：检索词“系统学es”被**分词为【系统学、es】两个Token**，【语句1】的operator**默认值为or**，所以文档1和2可以被检索到；【语句2】的operator的值是and，也就是需要**同时包含【系统学、es】这两个Token才行**，所以没有结果。



## 8.ES的mapping设置（表结构）

#####   1）Mapping简介

​        1）类似于数据库中的表结构，主要作用如下：

​            A、定义Index下的Field Name；

​            B、定义Field的类型，如：数值型、字符串型、布尔型等；

​            C、定义倒排索引的相关配置，如：是否有索引，记录position等。

​        2）获取一个mapping，使用endpoint：_mapping，例如：

```bash
GET /test_index/_mapping
```

#####     2）自定义Mapping

​        1）使用mappings进行自定义mapping，示例：

```json
#创建名为my_index的索引，并自定义mapping
PUT my_index
{
 "mappings":{    #1.关键字
  "doc":{    #2.类型名
   "properties":{    #4.字段名称及类型定义
    "title":{
     "type":"text"    #3.字段类型
    },
    "name":{
     "type":"keyword"
    },
    "age":{
     "type":"integer"
    } 
   }                 #4.截止
  }
 }
}
```

   Mapping中的字段类型一旦设定之后，**禁止直接修改**。因为Luence事先的倒排索引生成后不能修改。如果一定要改，可以重新建立新的索引，然后对应修改mapping，之后将之前的数据进行reindex操作，导入新的文档。

  2）自定义mapping时允许新增字段。通过***dynamic参数***进行控制字段的新增，dynamic有三种配置：

​          A、true。默认配置，允许自动新增字段；

​          B、false。不允许自动新增字段，文档可以正常写入，但不能进行查询等操作；

​          C、strict。严格模式。文档不能写入，写入会报错。

```json
#使用dynamic参数控制字段的新增
PUT my_test_index
{
 "mappings":{
  "doc":{
   "dynamic":false,   #设置为false，索引不允许新增字段
   "properties":{
    "title":{
     "type":"text"
    },
    "name":{
     "type":"keyword"
    },
    "age":{
     "type":"integer"
    
   }
  }
 }
}
```

 如果此时，PUT新的字段，就不会成功，虽然不报错，但是没有内容：

#向名为my_test_index的索引增加字段

```
PUT my_test_index/doc/1
{
    "name":"xxx",
    "age":"xx",
    "gender":"xx"
}

GET my_test_index/_search

GET my_test_index/_mapping

GET my_test_index/_search
{
  "query":{
    "match":{
      "gender":"male"
    }
  }
}
```



#####     3）copy_to的使用

功能：将该字段的值复制到目标字段

```
#copy_to的使用
PUT my_index
{
 "mappings":{
  "doc":{
   "properties":{
    "first_name":{
     "type":"text",
     "copy_to":"full_name"
    },
    "last_name":{
     "type":"text",
     "copy_to":"full_name"
    },
    "full_name":{
     "type":"text"
    }
   }
  }
 }
}


#向索引写入数据
PUT my_index/doc/1
{
 "first_name":"John",
 "last_name":"Smith"
}

#查询索引my_index中full_name同时包含John 和 Smith的数据
GET my_index/_search
{
 "query":{
  "match":{
   "full_name":{
    "query":"John Smith",
    "operator":"and"
   }
  }
 }
}
```

##### 4）index参数的使用

  功能：控制当前字段是否为索引，默认true，当设置为false的时候，不进行记录，此时该字段不能被搜索

```json
#index参数的使用
PUT my_index
{
 "mappings":{
  "doc":{
   "properties":{
    "cookie":{
     "type":"text",
     "index":false    #设置为false，该字段不能被搜索
    }
   }
  }
 }
}
```

此时在进行数据写入和查询，不能进行该字段搜索。一般用来进行不想被查询的私密信息设置，如身份证号，电话号码等：

```
#向使用了index参数的字段写入信息
PUT my_index/doc/1
{
 "cookie":"name=alfred"
}
```

##### 5）index_options参数的使用

​        功能：控制倒排索引记录的内容，有如下四种配置：

​          1）**docs**                只记录文档ID

​          2）**freqs**               记录文档ID和词频TF

​          3）**positions**        记录文档ID、词频TF和分词位置 (默认)

​          4）**offsets**            记录文档ID、词频TF、分词位置和偏移

​        其中：text类型默认的配置是positions，其他的比如integer等类型默认为docs，目的是为了节省空间。

```json
#index_options参数的使用
PUT my_index
{
 "mappings":{
  "doc":{
   "properties":{
    "cookie":{
     "type":"text",
     "index_options":"offsets"  #记录文档ID、词频TF、分词位置和偏移
    }
   }
  }
 }
}
```

##### 6）null_value参数的使用

功能：当字段遇到空值null时的处理策略。默认为null，即跳过。此时ES会忽略该值，可通过修改进行默认值的修改：

```json
#使用null_value修改ES遇到null值时的默认返回值
PUT my_index
{
 "mappings":{
  "doc":{
   "properties":{
    "cookie":{
     "type":"keyword",
     "null_value":"NULL"    #当遇到空值null的时候，返回一个字符串形式的NULL
    }
   }
  }
 }
}
```

#####    7）Field字段的数据类型

​        1）***核心数据类型***

​          A、**字符串型**。          text(分词)，keyword(不分词)

​          B、**数值型**。              long、integer、short、byte、double、float、half_float、scaled_float

​          C、**日期类型**。          date

​          D、**布尔类型**。          boolean

​          E、**二进制类型**。       binary

​          F、**范围类型**。           integer_range、float_range、long_range、double_range、date_range

​        2）***复杂数据类型***

​          A、**数组类型**。           array

​          B、**对象类型**。           object

​          C、**嵌套类型**。           nested object

​        3）**地理位置数据类型**

​          A、**点**。                      geo-point

​          B、**形状**。                   geo-shape

​        4）***专用类型***

​          A、**记录ip地址**。         ip

​          B、**实现自动补全**。    completion

​          C、**记录分词数**。        token_count

​          D、**记录字符串hash值**。murmur3

​          E、**perclator**。

​          F、**join**。

​        5）多字段特性：

​          ES允许对同一个字段采用不同的配置，如：**分词**。举例：**对一个人名实现拼音搜索**，只需要在人名字段中新增一个子字段pinyin即可。

```json
#多字段特性
{
 "my_index":{
  "mappings":{
   "doc":{
    "properties":{
     "username":{
      "type":"text",
      "field":{    #增加子字段pinyin
       "pinyin":{
        "type":"text",
        "analyzer":"pinyin"
       }
      }
     }  
    }
   }
  }
 }
}
```

##### 8）ES的自动类型识别

​        1）Dynamic Mapping：

​            ES可以自动识别文档字段类型，从而降低用户使用成本。

```json
#ES的自动类型识别

PUT my_index/doc/1
{
 "username":"alfred",    #username字段自动识别为text类型
 "age":20                #age字段自动识别为long类型
}
```

​        2）ES依靠JSON文档的字段类型实现自动识别字段类型：

| JSON类型 | ElasticSearch类型                                            |
| -------- | ------------------------------------------------------------ |
| null     | 忽略                                                         |
| boolean  | boolean                                                      |
| 浮点类型 | float                                                        |
| 整数类型 | long                                                         |
| object   | object                                                       |
| array    | 由第一个非null的值的类型决定                                 |
| String   | 匹配为日期，则为date类型(默认开启)；匹配为数字，则为long类型/float类型(默认关闭)；都未匹配，则设为text类型，并附带keyword子字段 |

​        3）验证ES的字段类型自动识别：

```json
#验证ES的字段类型自动识别

PUT my_index/doc/1
{
 "username":"alfred",    #字符串类型text
 "age":20,    #整数long
 "bitrh":"1998-10-10",    #默认识别日期date
 "married":false,    #布尔类型boolean
 "year":"18"    #默认不识别数字text
 "tags":["boy","fashion"],    #数组中第一个不为null的元素为字符串类型，所以为text
 "money":100.1    #浮点类型float
}
```

​        再对my_index进行mapping查询，就会获得每个字段的类型：

##### 9）ES中日期类型和数字的自动识别

  ES中可自行配置日期的格式，默认：["strict_date_optional_time","yyyy/MM/dd HH:mm:ss Z|| yyyy/MM/dd z"]

1）使用dynamic_date_formats自定义日期格式

```
#使用dynamic_date_formats自定义日期格式
PUT my_index
{
 "mappings":{
  "doc":{
   "dynamic_date_formats":["MM/dd/yyyy"]
  }
 }
}
 
#写入符合自定义格式的日期数据，可识别为date类型
PUT my_index/doc/1
{
 "create_time":"01/01/2018"    #create_time字段识别为date类型
}
```

​    2）使用date_detection可以关闭自动识别日期格式：

```
#使用date_detection关闭日期的自动识别
PUT my_index
{
 "mappings":{
  "doc":{
   "date_detection":false 
  }
 }
}
 
PUT my_index/doc/1
{
 "create_time":"01/01/2018"    #create_time字段是text类型
}
```

 ES中可配置数字是否识别，默认关闭：

```
#ES配置开启数字的自动识别
PUT my_index
{
 "mappings":{
  "doc":{
   "numeric_detection":true    #开启数字自动识别
  }
 }
}
 
#写入数字数据，ES可以自动识别其类型
PUT mu_index/doc/1
{
 "year":"18",    #year字段自动识别为long类型
 "money":"100.1"    #money字段自动识别为float类型
}
```

#####    10）ES中根据自动识别的数据类型，动态生成字符类型：

​        1）(例)所有字符串类型都设为keyword类型（不分词）

​        2）(例)所有以message开头的字段都设为text类型（分词）

​        3）(例)所有以long_开头的字段都设为long类型

​        4）(例)所有自动匹配为double的类型都设为float类型。（为了节省空间）

​        举例：下面的例子就是讲匹配到的字符串类型都设为keyword类型。

```
#ES根据自动识别的数据类型、字段名等动态设定字符类型
PUT test_index
{
 "mappings":{
  "doc":{
   "dynamic_template":[
    {
     "strings":{
      "match_mapping_type":"string",    #匹配到所有的字符串类型，全部设为keyword类型
      "mapping":{
       "type":"keyword"
      }
     }
    }
   ]
  }
 }
}
```

  匹配规则一般有如下几个参数：

​          1）match_mapping_type                匹配ES自动识别的字段类型，如boolean、long、string等；

​          2）match、unmatch                       匹配字段名，比如"match":"message*" ===>以message开头的数据；

​          3）path_match、path_unmatch    匹配路径



#####  11）一般来说，自定义mapping的操作步骤如下

​          1）**写入一条文档到ES的临时索引中，获取(复制)ES自动生成的mapping；**

​          2）**修改获得的mapping，并在其中自定义相关配置；**

​          3）**使用修改后的mapping创建实际所需索引。**

​          这样不但在开发上简便不少，而且还能防止字段遗漏。



## 9.ElasticSearch的Search API*

#####  0）在ES中，为了实现对存储的数据进行查询分析，使用endpoint：**_search**

​        可以实现对索引的不同查询，如：

​        A、**实现对所有索引的泛查询**：GET /_search

​        B、**实现对一个索引的单独查询**：GET /my_index/_search

​        C、**实现对多个索引的指定查询**：GET /my_index1,my_index2/_search

​        D、**实现对符合指定要求的索引进行查询**：GET /my_*/_search

​        在进行查询的时候，主要有两种方式：

  在进行查询的时候，主要有两种方式：

​        A）URI Search。

​            特点：操作简单，直接通过命令行方便测试，但仅包含部分查询语法；

​            举例：

```bash
#URI Search方式进行查询

GET my_test_index/_search?q=name:moumou
```

​        B）Request Body Search。

​            特点：ES提供的完备查询语法，使用Query DSL(Domain Specific Language)进行查询。

​            举例：

```json
Request Body Search方式进行查询
GET my_test_index/_search
{
  "query":{
    "match":{
      "name":"moumou"
    }
  }
}
```

#####    1）URI Search

   1）通过url query参数实现搜索，常用参数有：

​            A、q                            指定查询的语句，使用query string syntax语法

​            B、df                           q中不指定字段时默认查询的字段    ===>   在不指定的时候默认查询所有字段

​            C、sort                        排序

​            D、timeout                 指定超时时间，默认不超时   

​            E、from，size            用于分页

​     举例：

```bash
#使用url query查询示例：

GET my_index/_search?q=username:alfred&df=username&sort=age:asc&from=4&size=10&timeout=1s
```

​        解释：

​        查询索引**my_index**中**username**字段中包含**alfred**的文档，结果按**age**字段**升序排列**，返回第**5——14**个文档，若超过**1s**未结束，则以超时结束。

#####      2）query string syntax语法

​            前置内容：term————单词，phrase————词语。

​            A、单词与词语语法：

​                单词：alfred way    等价于    alfred **OR** way

​                词语："alfred way"    语句查询，要求先后顺序

​                泛查询：不指定字段，会在所有字段中去匹配其单词

​                指定字段查询：指定字段，在指定字段中匹配单词

​            B、Group分组设定，使用括号指定匹配的规则，举例：

```bash
#分组设定,使用括号进行分组

GET my_index/_search?q=username:(alfred OR way)AND lee
```

#####         3）search api

​     A、泛查询：

```bash
#泛查询
GET my_index/_search?q=alfred
{
 "profile":true    #使用profile参数，可以明确地看到ES如何执行的查询条件
}
```

​    B、指定字段查询：

​          a、查询字段username中包含alfred的文档：

```bash
#查询字段username中包含alfred的文档
GET my_index/_search?q=username:alfred
```

​           b、查询字段username中包含alfred或way的文档

```bash
#查询字段username中包含alfred或way的文档
GET my_index/_search?q=username:alfred way
```

​            c、查询字段username为"alfred way"的文档

```bash
#查询字段username为"alfred way"的文档
GET my_index/_search?q=username:"alfred way"
```

​             d、分组后，查询字段username中包含alfred和包含way的文档

```bash
#分组后，查询字段username中包含alfred，包含way的文档
GET my_index/_search?q=username:(alfred way)
```

​                    这个和b的结果一样，但是区别在于使用分组之后，不进行泛查询。

#####      4）布尔操作符

​            A、AND(&&)、OR(||)、NOT(!)，举例：

```bash
#查询索引my_index中username包含alfred但是不包含way的文档
GET my_index/_search?q=username:(alfred NOT way)
```

​            B、+对应must，-对应must_not，举例：

```bash
#查询索引my_index中一定包含lee，一定不含alfred，可能有way的文档
GET my_index/_search?q=username:(way +lee -alfred)
#或写成
GET my_index/_search?q=username:((lee && !alfred) || (way && lee && !alfred))
```

​            但是要注意一点：

​            在url中，+(加号)会被解析成空格，所以要使用encode后的结果，即：%2B，所以正确的查询语句应该为：

```bash
#查询索引my_index中一定包含lee，一定不包含alfred，可能包含way的文档
GET my_index/_search?q=username:(way %2Blee -alfred)
```

#####         5）范围查询（支持数值和日期）

​            A、区间写法：闭区间使用[ ]，开区间使用{ }

​                a、age:[1 TO 10]                                1 <= age <= 10

​                b、age:[1 TO 10}                                1 <= age < 10

​                c、age:[1 TO ]                                     age >= 1

​                d、age:[* TO 10]                                age <= 10

​            B、算数符号写法：

​                a、age:>= 1                                        

​                b、age:(>=1 && <= 10) / age:(+ >= 1 + <= 10)

​                举例：

```json
#查询索引my_index中username字段包含alfred或年龄大于20的文档

GET my_index/_search?q=username:alfred age>20
#查询索引my_index中username字段包含alfred且年龄大于20的文档
GET my_index/_search?q=username:alfred AND age>20
```

​            C、还可以对日期进行范围查询，注意：年/月是从1月1号/1号开始算的：

```json
#查询索引my_index中birth字段在1985和1990之间的文档
GET my_index/_search?q=birth:(>1985 AND < 1990)
```

#####         6）通配符查询

​            ?代表一个字符，*代表0个或多个字符，如：

​            name:a?lfred         /         name:a*d         /        name:alfred*

​            注意：通配符匹配的执行效率较低，且占用内存较多，不建议使用，如果没有特殊要求，也不要将?或者*放在最前面，因为意味着要匹配所有文档，可能会造成OOM。

​          此外，还支持正则表达式/模糊匹配/近似度查询等

​            A、正则表达式：举例：/[a]?l.*/

​            B、模糊匹配：fuzzy query。举例：

```bash
#模糊匹配。匹配与alfred差一个字符的词，比如：alfreds、alfret等
GET my_index/_search?q=username:alfred~1
```

​            C、近似度查询：proximity search。举例：

```bash
#近似度查询，查询字段username和"alfred way"差n个单词的文档

GET my_index/_search?q=username:"alfred way" ~5
```

​            **使用场景常见于用户输入词的纠错中。**

   2、Request Body Search

​        ES自带的完备查询语句，将查询语句通过http request body发送到ES，主要参数有：

​        ①query    ===>    符合**Query DSL语法**的查询条件

​        ②from，size

​        ③timeout

​        ④sort

#####     7）*Query DSL语法：*

​        **基于JSON定义的查询语言，主要包含两个类型：**

​        **1）字段类查询————如：term，match，range等。只针对一个字段进行查询**

​                A、**全文匹配**

​                    针对text类型的字段进行全文检索，会对查询语句进行“先分词再查询”处理，如：match、match_phrase等

​                a、match query

​                       ①对字段进行全文检索(最基本和最常用的查询类型)，举例：

```bash
#match query

GET my_index/_search
{
  "query":{  
   "match":{    #关键词
    "username":"alfred way"    #字段名和查询语句
   }
  }
}
```

  从结果，可以返回匹配文件总数，返回文档列表，_score相关性得分等。

​       一般的执行流程为：

​            1.对查询语句分词==>2.根据字段的倒排索引列表，进行匹配算分==>3.汇总得分==>4.根据得分排序，返回匹配文档

​           ②使用operator参数，可以控制单词间关系，有and / or：

```bash
#使用operator参数控制单词间关系

GET my_index/_search
{
 "query":{
  "match":{
   "username":"alfred way",
   "operator":"and"    #and，同时包含alfred和way

  }
 }

}
```

​          b、相关性算分，其本质就是一个排序问题

​                        计算文档与待查询语句之间的相关度，一般有四个重要概念：

​                        a、**Term Frequency 词频**(正相关)

​                        b、**Document Frequency 文档频率**(负相关)

​                        c、**Inverse Term Frequency 逆文本频率**(正相关)

​                        d、**Field-length Norm 文档长度**(负相关)

​                        目前ES有两个相关性算分的模型：

​                        ①**TF/IDF模型**。经典模型。

![img](https://img-blog.csdnimg.cn/2018112911263839.png)

​                        在使用kibana进行查询时，使用explain参数，可以查看具体的计算方法。

```json
#使用explain参数，可以查看具体的相关性的得分是如何计算的

GET my_index/_search
{
 "explain":true,    #设置为true
 "query":{
  "match":{
   "username":"alfred"
  }
 }
}
```

​       注意：ES计算相关性得分是根据shard进行的，即分片的分数计算相互独立，所以在使用的时候要注意分片数，可以通过设定分片数为1来避免这个问题，主要是为了观察，不代表之后所有的分片全都设为1。一般放在创建索引后，未加数据之前。

```json
#设定shards数量为1

PUT my_index
{
 "settings":{
  "number_of_shards":"1"
 }
}
```

​       ②**BM25模型**。5.x版本后的默认模型，是对TF/IDF的优化模型。best match，25指：迭代了25次才计算。

![1596765660243](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1596765660243.png)



  **BM25的使用，降低了TF/IDF中因为TF过大导致的负面影响，在BM25中，一个单词的TF一直增长，到一定程度就趋于0变化。**

​     c、match phrase query

​           对字段做全文检索，有顺序要求。(短语匹配)

```
#使用match——phrase查询词语
GET my_index/_search
{
 "query":{
  "match_phrase":{    #关键词
   "job":"java engineer"
  }
 }
}
```

  通过使用**slop参数**，可以控制单词间间隔：

```
#使用slop参数，控制词语中的单词间隔
GET my_index/_search
{
 "query":{
  "match_phrase":{
   "job":{
    "query":"java engineer",
    "slop":"1"    #关键词，设定单词间隔
   }
  }
 }
}
```

​    d、query string query

​         类似于URI Search中的q参数查询，举例：

```
#使用query_string查询
GET my_index/_search
{
 "query":{
  "query_string":{
   "default_field":"username",
   "query":"alfred AND way"
  }
 }
}
#或
GET my_index/_search
{
 "query":{
  "query_string":{
   "fileds":["username","job"],
   "query":"alfred OR (java AND ruby)"
  }
 }
}
```

   e、simple query string query

​        类似于query string，但会忽略错误的查询语法，且仅支持部分查询语句。使用+，|，-分别代替AND，OR，NOT。

​                举例：

```
#使用simple query string query
GET my_index/_search
{
 "query":{
  "simple_query_string":{
   "fields":[username],
   "query":"alfred +way"    #等价于 "query":"alfred AND way"
  }
 }
}
```

   B、**单词匹配**

​        a、term/terms query

​        将待查询语句作为整个单词进行查询，不做分词处理，举例：

```json
#使用term进行单查询
GET my_index/_search
{
 "query":{
  "term":{
   "username":"alfred"
  }
 }
}
#使用terms进行多查询
GET my_index/_search
{
 "query":{
  "terms":{
   "username":["alfred","way"]
  }
 }
}
```

​    此时如果直接使用alfred way作为username查询条件，是不会返回任何文档的。因为在username的倒排索引列表中，存在"alfred"和"way"的索引，但是不存在"alfred way"的索引。

​     b、range query

​          范围查询，主要针对数值类型和日期类型。

​           gt: greater than 大于

​           gte: greate than or equal to 大于等于

​           lt: less than 小于

​           lte: less than or equal to 小于等于

​          ①对数值的查询

```
#range query对数值的查询
GET my_index/_search
{
 "query":{
  "range":{
   "age":{
    "gte":10,
    "lte":20
   }
  }
 }
}
```

​      ②对日期的查询

```
#range query对日期的查询
GET my_index/_search
{
 "query":{
  "range":{
   "birth":{
    "lte":"1988-01-01"   #或者使用  "lte":"now-30y",这种Date Math类型
   }
  }
 }
}
```

​        **Date Math类型**：针对日期提供的一种更友好的计算方式。

​        当前时间用now代替，具体时间的引用，需要使用||间隔。年、月、日、时、分、秒跟date一致：y、M、w、d、h、m、s。举例：

```
#假设当前时间为2018-01-02 12:00:00
now+1h   =>   2018-01-02 13:00:00
now-1h   =>   2018-01-02 11:00:00
now-1h/d =>   2018-01-02 00:00:00
2016-01-01||+1M/d  => 2016-02-01 00:00:00
```

 **2）复合查询————如：bool查询等。包含一个/多个字段类查询/符合查询语句**

​        A、constant_score query

​         将内部的查询结果文档得分全部设定为1或boost的值。返回的相关性得分全部为1或boost

```json
#使用constant_score query
GET my_index/_search
{
 "query":{
  "constant_score":{    #关键词
   "match":{
    "username":"alfred"
   }
  }
 }
}
```

   B、bool query

​         由一个/多个布尔子句组成，主要包含以下四个：

​         a、**filter**     只过滤符合条件的文档，不计算相关性得分，返回的相关性得分全部为0；

​         ES会对filter进行智能缓存，因此执行效率较高，在做简单匹配查询且不考虑得分的时候没推荐使用filter代替query

```json
#使用filter查询
GET my_index/_search
{
 "query":{
  "bool":{
   "filter":{
    "term":{
     "username":"alfred"
    }
   }
  }
 }
}
```

b、**must**     文档必须符合must中的所有条件，影响相关性得分；

```json
#使用must进行查询
GET my_index/_search
{
 "query":{
  "bool":{
   "must":[    
    {
     "match":{
      "username":"alfred"
     }
    },
    {
     "match":{
      "job":"specialist"
     }
    }
   ]
  }
 }
}
```

 c、**must_not**      文档必须排除must_not中的所有条件； 

```json
#使用must_not进行查询
GET my_index/_search
{
 "query":{
  "bool":{
   "must":[
   {
    "match":{
     "job":"java"
    }
   }
   ],
   "must_not":[
   {
    "match":{
     "job":"ruby"
    }
   }
   ]
  }
 }
}
```

 d、**should**       文档可以符合should中的条件，影响相关性得分，分为两种情况：

​       同时配合minimum_should_match控制满足调价你的个数/百分比。

​        ①bool查询中只有should，不包含must的情况

```
#bool查询中只有should的情况
GET my_index/_search
{
 "query":{
  "bool":{
   "should":[
    {
     "term":{"job":"java"}    #条件1
    },
    {
     "term":{"job":"ruby"}    #条件3
    }
    {
     "term":{"job":"specialist"}    #条件3
    }
   ],
   "minimum_should_match":2    #至少需要满足两个条件
  }
 }
}
```

②bool查询中既有should，又包含must的情况，文档不必满足should中的条件，但是如果满足的话则会增加相关性得分。

#bool查询中同时包含should和must

```
GET my_index/_search
{
 "query":{
  "bool":{
   "should":[    #同时包含should
   {
    "term":{"job":"ruby"}
   }
   ],
   "must":[    #同时包含must
   {
    "term":{"usernmae":"alfred"}
   }
   ]
  }
 }
}
```

当一个查询语句位于query或filter上下文的时候，ES的执行结果也不同。

| query  | 查找和查询语句最匹配的文档，并对所有文档计算相关性得分 | query bool中的：must/should                          |
| ------ | ------------------------------------------------------ | ---------------------------------------------------- |
| filter | 查找和查询语句最匹配的文档                             | bool中的：filter/must_not constant_score中的：filter |

```
#query和filter上下文
GET my_index/_search
{
 "query":{
  "bool":{
   "must":[    #query上下文
   {
    "term":{"title":"Search"}
   },
   {
    "term":{"content":"ElasticSearch"}
   }
   ],
   "filter":[    #filter上下文
   {
    "term":{"status":"published"}
   },
   {
    "range":{
     "publish_date":{
      "gte":"2015-01-01"
     }
    }
   }
   ]
  }
 }
}
```

 D、count API

​        获取符合条件的文档书，使用endpoint：_count。

```
#使用_count获取符合条件的文档数
GET my_index/_count    #关键词
{
 "query":{
  "match":{
   "username":"alfred"
  }
 }
}
```

E、Source Filtering

​                过滤返回结果中的_source中的字段，主要由以下两种方式：

​                    a、GET my_index/_search?_source=username        #url参数

​                    b、使用Request Body Search：

```
#不返回_source
GET my_index/_search
{
 "_source":false
}
#返回_source部分字段
GET my_index/_search
{
 "_source":["username","age"]
}
#通配符匹配返回_source部分字段
GET my_index/_search
{
 "_source":{
  "includes":"*I*",
  "encludes":"birth"
 }
}
```



## 10.ElasticSearch的分布式特性*

#####   1、ES支持集群模式，即一个分布式系统。其好处主要有以下2个:

​            A、**可增大系统容量**。比如：内存、磁盘的增加使得ES能够支持PB级别的数据；

​            B、**提高了系统可用性**。即使一部分节点停止服务，集群依然可以正常对外服务。

​        2）ES集群由多个ES实例构成。

​            不同集群通过集群名字来区分，通过配置文件**elasticsearch.yml**中的**cluster.name**可以修改，默认为**elasticsearch**。

​            每个ES实例的本质，其实是一个JVM进程，且有自己的名字，通过配置文件中的**node.name**可以修改。

#####  2、构建ES集群

   1）启动一个节点：

```
#ES启动单节点，配置集群名称，节点名称，文件路径
bin/elasticsearch -Epath.data=node1 -Ecluster.name=my_cluster -Enode.name=node1 -d
```

每个集群包含cluster state，主要记录一下信息：

​                节点信息：如节点名称、连接地址等

​                索引信息：如索引名称、配置等

 2）几个节点的名称：

​            A、**主节点master**：可修改cluster state的节点。一个集群仅一个。

​                cluster state存储于每个节点上，master维护最新版本并向其他从节点同步。

​            B、**选举节点master-eligible**：可以参与选举master的节点。

​                配置为：node.master:true。默认所有节点都可以参与选举。

​            C、**协调节点cordinating**：处理请求的节点。是所有节点的默认角色，且不能取消。

​                路由请求到正确的节点处理，如：创建索引的请求到master节点。

​            D、**从节点data**：存储数据的节点，默认所有节点都是data类型。

​                配置为：node.data:true。

3）如何在本地启动一个集群

​            当一个集群中只有一个节点，意味着这个节点宕机之后，整个集群都停止对外服务功能，所以，为了解决这个办法，可以在本地启动多个节点创建一个本地化集群：

```bash
#创建一个本地化集群my_cluster
bin/elasticsearch -Epath.data=node1 -Ecluster.name=my_cluster -Enode.name=node1 -d
bin/elasticsearch -Ehttp.port=8200 -Epath.data=node2 -Ecluster.name=my_cluster -Enode.name=node2 -d
bin/elasticsearch -Ehttp.port=7200 -Epath.data=node3 -Ecluster.name=my_cluster -Enode.name=node3 -d
```



![1596768666588](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1596768666588.png)

#####  3、副本和分片

​        1）提高系统可用性

​            为了提高系统的可用性，从两个角度进行考虑：

​            A、**服务可用性**：在2个/多个节点的情况下，允许1个/一部分节点停止服务，整个集群依然可以对外提供服务；

​            B、 **数据可用性**：引入**副本(replication)**解决1个/一部分节点停止，节点上数据也同时丢失的情况。此时每个节点都有完备数据。

  2）增大系统容量

​            先思考一个问题：如何将数据分布到所有节点？

​            答案是：引入分片(shard)。

​            A、什么是分片？

​                分片是ES能支持PB级别数据的基石。

​                a、分片存储部分数据，可以分布于任意节点；

​                b、分片数在索引创建时指定，且后续不能更改，默认为5个；

​                c、有主分片和副本分片之分，以实现数据的高可用；

​                d、副本分片由主分片同步数据，可以有多个，从而提高数据吞吐量。

​            B、如何设置分片数和副本数？

​                a、通过在DevTools在创建索引时指定：

#设置分片数和副本数

```
PUT my_test
{
 "settings":{
  "number_of_shards":5,    #设置分片数为3
  "number_of_replicas":1    #设置副本数为1
 }
}
```

![1596769012966](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1596769012966.png)

 B、两个很有意义的问题：

​                假设现在有3个节点，每个节点上有一个主分片和一个副本分片，那么：

​                a、此时增加新的节点，能否提高索引的数据容量？

​                b、此时增加新的副本，能否提高索引读取时的吞吐量？

​                答案是：**不行**。

​                原因：a）三个节点上分布了同一个索引的三个主分片和三个副本分片，已经将所有的数据存到了这三个节点，如果此时增加新的节点，却没有数据给它们利用，造成了资源浪费，且不能提高索引的数据容量；b）新增的副本数，也会增加到这三个节点上，利用的还是同样的资源，所以在三个同样的节点上读取索引数据，吞吐量也不会提高。

​                那么，在这种情况下，如何才能提高这个索引的吞吐量呢？

​                答案是：**既增加新的节点，又增加新的副本**，这样把新的副本放在新的节点上，进行索引数据读取的时候，并且读取，就会提升索引数据读取的吞吐量。

#####  4、ES集群健康状态

 ES的健康状态分为三种：

​           1）Greed，绿色。表示所有主分片和副本分片都正常分配；

​           2）Yellow，黄色。表示所有主分片都正常分配，但有副本分片未分配；

​           3）Red，红色。表示有主分片未分配。

​        集群的状态可以通过api或者插件查看，下面是使用api查看集群状态的命令：

```json
#使用api，查看集群的状态

GET _cluster/health

{
  "cluster_name": "my_cluster",
  "status": "green",
  "timed_out": false,
  "number_of_nodes": 3,
  "number_of_data_nodes": 3,
  "active_primary_shards": 13,
  "active_shards": 26,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 0,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 100
}
```

​        此时，会返回集群名称，集群状态，节点数，活跃分片数等信息。

​        如果此时磁盘空间不够，name在创建新的索引的时候，主副分片都不会再分配，此时的集群状态会直接飙红，但此时依然可以访问集群和索引，也可以正常进行搜索。

​        所以：**ES的集群状态为红色，不一定就不能正常服务**。

#####     5、故障转移 Failover

​        1）什么是故障转移？

​            在一个多节点集群中的master节点突然宕机，此时集群缺少主节点，剩余的节点组成新的集群，并将幸存副本分片转为主分片，同时在剩余节点生成副本分片，并依然对外进行正常服务的情况，称为故障转移。

​        2）如何进行故障转移？

​            当其余节点发现定时ping主节点master无响应的时候，集群状态转为**Red**。此时会发起master选举。称为master的新主节点发现有主分片没有进行分配，会将继续工作的节点上的副本分片升级为主分片，此时集群状态转为**Yellow**。之后，主节点会为对应节点生成未分配的副本分片，此时集群状态转为**Green**。整个故障转移过程结束。

![1596769149805](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1596769149805.png)

#####  6、文档分布式存储

​        Document最终存储到分片上，那么文档的数据是如何选择要存储的分片的呢？

​        此时，就需要**文档到分片的映射算法**。

​        目的：是文档均匀分布到所有分片上，以充分利用资源。之所以不用随机和轮询round-robin算法的原因是：需要维护文档到分片的映射关系，那么在PB级别的数据量的时候，这是一个成本非常大的工程。

​        所以：**直接根据文档值实时计算对应的分片即可**。分片的计算公式：

```bash
shard = hash(routing)%number_of_primary_shards
```

​        hash保证数据均匀分布在分片中，**routing作为关键参数，默认为文档ID**，number_of_primary_shards为主分片数。

​        这也是为什么，主分片数一旦设定，不能更改的原因————**为了保证文档对应的分片不会发生改变**。

​        接下来介绍文档创建和读取的流程，以一个集群三个节点为例：

​        1）文档创建的流程

​            A、Client向node3发送创建文档请求；

​            B、node3通过routing计算该文档存储在shard1上，查询cluster state后，确认主shard1在node2上，然后转发请求到node2；

​            C、node2上的主shard1接收并执行创建文档的请求后，将同样的请求转发到副shard1，查询cluster state后，确认副shard1在node1，向node1发送请求；

​            D、node1上的副shard1接收并执行创建文档的请求后， 通知主shard1结果；

​            E、node2上的主shard1接收到副shard1创建文档成功的结果，通知node3创建成功；

​            F、node3返回结果给Client。

​        2）文档读取的流程

​            A、Client向node3发送读取文档doc1的请求；

​            B、node3通过routing计算该doc1在shard1上，查询cluster state后，确认主shard1和副shard1的位置，然后以**轮询**的机制获取一个shard(比如这次是主shard1，下次就是副shard1)；

​            C、本次是在node2上的主shard1接收到读取文档的请求，执行并返回结果给node3；

​            D、node3返回结果给Client。

​        3）文档批量创建的流程

​            A、Client向node3发送批量创建文档请求(bulk)；

​            B、node3通过routing计算文档对应的所有shard，然后按主shard分配对应执行的操作，同时发送请求到设计到的的主shard；

​            C、每个主shard接收并执行请求后，发送同样的请求到副shard；

​            D、每个副shard接收并执行请求后，返回结果到主shard，再由主shard返回给 node3；

​            E、node3整合所有结果，并返回给Client。

​        4）文档批量读取的流程

​            A、Client向node3发送批量读取文档请求(bulk)；

​            B、node3通过routing计算文档对应的所有shard，再以轮询的机制，按shard构建mget请求，通过发送给设计的shard；

​            C、由shard返回文档结果；

​            D、node3整合后返回结果到Client。

#####     7、脑裂问题

​        1）什么是脑裂问题？

​            在分布式系统中有一个经典的网络问题。

​            当一个集群在运行时，作为master节点的node1的网络突然出现问题，无法和其他节点通信，出现网络隔离情况。那么node1自己会组成一个单节点集群，并更新cluster state；同时作为data节点的node2和node3因为无法和node1通信，则通过选举产生了一个新的master节点node2，也更新了cluster state。那么当node1的网络通信恢复之后，集群无法选择正确的master。

​        2）如何解决脑裂问题？

​           解决方案也很简单：仅在可选举的master-eligible节点数 >= quorum的时候才进行master选举。

​           quorum(至少为2)=master-eligible数量/2   +  1。

​           通过discovery.zen.minimum_master_nodes为quorum即可避免脑裂。

​    8、Shards分片详解

​        1）倒排索引一旦生成，不能更改。

​            A、优点：

​                a、不用考虑并发写文件的问题，杜绝了锁机制带来的性能问题；

​                b、文件不在更改，则可以利用文件系统缓存，只需载入一次，只要内存足够，直接从内存中读取该文件，性能高；

​                c、利于生成缓存数据(且不需更改)；

​                d、利于对文件进行压缩存储，节省磁盘和内存存储空间。

​            B、缺点：

​                在写入新的文档时，必须重构倒排索引文件，然后替换掉老倒排索引文件后，新文档才能被检索到，**导致实时性差**。

​        2）解决文档搜索的实时性问题的方案：

​            新文档直接生成新待排索引文件，查询时同时查询所有倒排索引文件，然后做结果的汇总即可，从而提升了实时性。

​        3）Segment

​            Lucene就采用了上述方案，构建的单个倒排索引称为Segment，多个Segment合在一起称为Index(Lucene中的Index)。在ES中的一个shard分片，对应一个Lucene中的Index。且Lucene有一个专门记录所有Segment信息的文件叫做**Commit Point**。

​            Segment写入磁盘的过程依然很耗时，可以借助文件系统缓存的特性。【先将Segment在内存中创建并开放查询，来进一步提升实时性】，这个过程在ES中被称为：**refresh**。

​            在refresh之前，文档会先存储到一个缓冲队列buffer中，refresh发生时，将buffer中的所有文档清空，并生成Segment。

​            ES默认每1s执行一次refresh操作，因此实时性提升到了1s。这也是ES被称为近实时的原因（Near Real Time）。

​        4）translog文件 binlog

​            那么，如果在节点写入磁盘之前就发生了宕机，这时候内存中的segment丢失，该怎么解决呢？

​            此时，引入了translog机制：当文档写入buffer时，同时会将该操作写入到translog中，这个文件会即时将数据写入磁盘，在6.0版本之后默认每个要求都必须落盘，这个操作叫做**fsync操作**。这个时间也是可以通过配置：index.translog.*进行修改的。比如每五秒进行一次fdync操作，那么风险就是丢失这5s内的数据。

​        5）文档搜索实时性————flush(十分重要)

​            flush的功能，就是：将内存中的Segment写入磁盘，主要做如下工作：

​                A、将translog写入磁盘；

​                B、将index bufffer清空，其中的文档生成一个新的Segment，相当于触发一次refresh；

​                C、更新Commit Point文件并写入磁盘；

​                D、执行fsync落盘操作，将内存中的Segment写入磁盘；

​                E、删除旧的translog文件。

​        6）refresh与flush的发生时机

​            A、refresh：发生时机主要有以下几种情况：

​                a、**间隔时间达到**。

​                    通过index.settings.refresh_interval设置，默认为1s。

​                b、**index.buffer占满时**。

​                    通过indices.memory.index_buffer_size设置，默认JVM heap的10%，且所有shard共享。

​                c、**flush发生时**。会触发一次refresh。

​            B、flush：发生时机主要有以下几种情况：

​                a、**间隔时间达到**。

​                    5.x版本之前，通过index.translog.flush_threshold_period设置，默认30min。

​                    5.x版本之后，ES强制每30min执行一次flush，不能再进行更改。

​                b、**translog占满时**。

​                    通过index.translog.flush_threshold_size设置，默认512m。且每个Index有自己的translog。

​        7）删除和更新文档：

​            A、删除：

​                  Segment一旦生成，就不能更改，删除的时候，Lucene专门维护一个.del文件，记录所有已删除的文档。

​                  .del 文件 上记录的是文档在Lucene中的ID，在查询结果返回之前，会过滤掉.del 文件 中的所有文档。

​            B、更新：

​                   先删除老文档，再创建新文档，两个文档的ID在Lucene中的ID不同，但是在ElasticSearch中ID相同。

​        8）Segment Merging(合并)

​            A、随着Segment的增多，由于每次查询的Segment数量也增多，导致查询速度变慢；

​            B、ES会定时在后台进行Segment merge的操作，减少Segment数量；

​            C、通过force_merge api可以手动强制做Segment的合并操作。

## 11.ElasticSearch中Search的运行机制*

 Search执行的时候，实际分为两个步骤执行：

​        ---> Query阶段：搜索

​        ---> Fetch阶段：获取

​    1、Query—Then—Fetch：

​        假设集群my_cluster中存在三个节点node1、node2、node3，其中master为node1，其余的为data节点。

​        1）**Query阶段**:

​            假设node3接收到Client发送的查询请求之后，先进行query：

​            A、node3在6个主副分片上随机选择3个分片，发送search request；

​            B、被选中的3个分片分别执行查询并排序，返回from+size个文档id和排序值；

​            C、node3整合3个分片返回的from+size个文档id，根据排序值排序选取from到from+size的文档id。

​        2）**Fetch阶段**:

​            node3根据Query阶段获取的文档id列表去对应shard上获取详情数据：

​            A、node3向相关分片发送multi_get请求；

​            B、3个分片返回文档详细数据；

​            C、node3拼接返回的结果返回给Client。

​    2、相关性算分：

​        **相关性算分在shard和shard之间是相互独立的**。也就意味着：同一个单词term在不同的shard上的TDF等值也可能是不同的。得分与shard有关。当文档数量不多是，会导致相关性算分严重不准的情况发生。

​        解决方案：

​            1）设置分片数为1个，从根本上排除问题。（此方案只适用于百万/少千万级的少量数据）

​            2）使用DFS Query-then-Fetch查询方式。

​        DFS Query-then-Fecth：

​            在拿到所有文档后，再重新进行完整的计算一次相关性得分，耗费更多的CPU和内存，执行性能也较低。所以也不推荐。

```json
#使用DLS Query-then-Fetch进行查询：

GET my_index/_search？search_type=dfs_query_then_fetch
{
 "query":{
  "match":{
   ..
  }
 }
}
```

​    3、排序相关：

​        默认采用相关性算分结果进行排序。可通过sort参数自定义排序规则，如：

```bash
#使用sort关键词进行排序

GET my_index/_search

{
 "sort":{    #关键词
  "birth":"desc"
 }
}



#或使用数组形式定义多字段排序规则
GET my_index/_search
{
 "sort":[    #使用数组
  {
   "birth":{
    "order":"asc"
   }
  },
  {
   "age":{
    "order":"desc"
   }
  }

 ]
}
```

​        1）直接按数字/日期排序，如上例中birth。

​        2）按字符串进行排序：字符串排序较特殊，因为在ES中有keyword和text两种：

​            A、针对text类型排序：

```bash
#直接对text类型进行排序

GET my_index/_search
{
 "sort":{
  "username":"desc"    #针对username字段进行倒序排序
 }
}
```

​                返回结果：

![img](https://img-blog.csdnimg.cn/20181203103528719.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0hpd2Vz,size_16,color_FFFFFF,t_70)

​            B、针对keyword类型排序：                

```json
#针对keyword进行排序

GET my_index/_search
{
 "sort":{
  "username.keyword":"desc"    #针对username的子类型keyword类型进行倒叙排序
 }
}
```

​        3）关于fielddata和docvalues:

​            排序的实质是对字段的原始内容排序的过程，此过程中**倒排索引无法发挥作**用，**需要用到正排索引**。即：通过文档ID和字段得到原始内容。ES提供2中实现方式：

​            A、Fielddata。    默认禁用。

​            B、DocValues。    默认启用，除了text类型。

​            Fielddata  对比  DocValues。

| 对比     | Fielddata                                            | DocValues                              |
| -------- | ---------------------------------------------------- | -------------------------------------- |
| 创建时机 | 搜索时即时创建                                       | 创建索引时创建，和倒排索引创建时间一致 |
| 创建位置 | JVM Heap                                             | 磁盘                                   |
| 优点     | 不占用额外磁盘空间                                   | 不占用Heap内存                         |
| 缺点     | 文档较多时，同时创建会花费过多时间，占用过多Heap内存 | 减慢索引的速度，占用额外的磁盘空间     |

​        4）Fielddata的开启:

​            Fielddata默认关闭，可通过如下api进行开启，且在后续使用时随时可以开启/关闭：

```bash
#开启字段的fielddata设置
PUT my_index/_mapping/doc
{
 "properties":{
  "username":{
   "type":"text",
   "fielddata":true    #关键词
  }
 }
}
```

​            使用场景：**一般在对分词做聚合分析的时候开启**。

​        5）Docvalues的关闭:

​            Docvalues默认开启，可在创建索引时关闭，且之后不能再打开，要打开只能做reindex操作。

```bash
#关闭字段的docvalues设置



PUT my_index

{

 "mappings":{
  "doc":{
   "properties":{
    "username":{
     "type":"keyword",
     "doc_values":false    #关键词
    }
   }
  }
 }
}
```

​            使用场景：当明确知道，不会使用这个字段排序或者不做聚合分析的时候，可关闭doc_values，减少磁盘空间的占用。

​    4、分页与遍历：

​        ES提供了三种方式来解决分页和遍历的问题：

​            ***from/size，scroll，search_after***。

​        1）from/size:

​            from：指明开始位置；

​            size：指明获取总数

```json
#使用from——size



GET my_index/_search
{
 "from":1,    #从第2个开始搜索
 "size":2     #获取2个长度
}
```

​            A、此时产生了一个经典的问题，也是分布式文件系统必定面对的问题：**深度分页**。

​                问题：如何在数据分片存储的情况下， 获取前1000个文档？

​                答案：先从每个分片上获取前1000个文档， 然后由处理节点聚合所有分片的结果之后，再排序获取前1000个文档。

​                此时页数越深，处理的文档就越多，占用的内存就越大，耗时就越长。这就是深度分页问题。

​                为了尽量避免深度分页为题，ES通过设定**index.max_result_window**限定最多到10000条数据。

​            B、在设计分页系统时，有一个**分页数**十分重要：

​                total_page=(total + page_size -1) / page_size

​                总分页数= (文档总数+认为设定的文档大小-1)   / 人为设定的文档大小

​                但是在搜索引擎中的意义并不大，因为如果排在前面的结果都不能让用户满意，那么越往后，越不能让用户满意。

​        2）scroll:

​            遍历文档集的API，以**快照**的方式来避免深度分页问题。

​            A、不能用来做实时搜索，因为数据不是实时的；

​            B、尽量不用复杂的sort条件，使用_doc最高效；

​            C、使用比较复杂。

​            步骤：

​                 a、发起一个scroll search：

```bash
#发起一个scroll search

GET my_index/_search?scroll=5m    #该快照的有效时间为5min
{
 "size"1    #指明每次scroll返回的文档数
}
```

​                 会返回后续会用到的_scroll_id：

![img](https://img-blog.csdnimg.cn/20181203110240612.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0hpd2Vz,size_16,color_FFFFFF,t_70)

​            b、调用scroll search 的api，获取文档集合，不断迭代至返回hits数组为空时停止：

```bash
POST _search/scroll

{
 "scroll":"5m",    #指明有效时间
 "scroll_id":"xxxxxx"    #上一步返回的_scroll_id
}
```

​            之后不断返回新的_scroll_id，使用新的_scroll_id进行查询，直到返回数组为空。

​            当不断的进行迭代，会产生很多scroll，导致大量内存被占用，可以通过clear api进行删除：

```bash
#使用clear api对scroll进行删除



DELETE /_search/scroll

{

 "scroll_id":[

   "xxxxxx",    #_scroll_id
   "xxxxxx",    #_scroll_id
......
 ]
}

#删除所有的scroll

DELETE /_search/scroll/_all
```

​        3）search_after:

​            避免深度分页的性能问题，提供实时的下一页文档获取功能。

​            缺点：不能使用from参数，即：不能指定页数。且只能下一页，不能上一页。

​            使用步骤：

​               A、第一步：正常搜索，但是要指定sort值，并保证值唯一：

```bash
#第一步，正常搜索



GET my_index/_search

{

 "size":1,
 "sort":{
  "age":"desc",
  "_id":"desc"
 }
}
```

​               B、第二步：使用上一步最后一个文档的sort值进行查询：

```bash
#第二步，使用sort值进行查询



GET my_index/_search
{
 "size":1,
 "search_after":[28,"2"],    #28,"2"，是上一次搜索返回的sort值
 "sort":{
  "age":"desc",
  "_id":"desc"
 }
}
```

​        4）如何避免深度分页问题:

​            这个问题目前连google都没能解决，所以只能最大程度避免，通过唯一排序值定位每次要处理的文档数都控制在size内：

​            应用场景：

​            A、from/size:需实时获取顶部的部分文档，且需自由翻页（实时）；

​            B、scroll:需全部文档，如：导出所有数据的功能（非实时）；

​            C、search_after:需全部文档，不需自由翻页（实时）。

## 12.ElasticSearch的聚合分析

#####     1.聚合分析简介

​        聚合分析，英文Aggregation，是ES除了搜索功能之外提供的针对ES数据进行统计分析的功能。

​        特点：①功能丰富，可满足大部分分析需求；②实时性高，所有计算结果实时返回。

```bash
#聚合分析格式：

GET my_index/_search
{
 "size":0,
 "aggs":{    #关键词
  "<aggregation_name>":{    #自定义聚合分析名称，一般起的有意义
   "<aggregation_type>":{    #聚合分析类型
    "<aggregation_body>"    #聚合分析主体
   }
  }
  [,"aggs":{[<svb_aggregation>]+}]    #可包含多个子聚合分析
 }
}
```

​        基于分析规则的不同，ES将聚合分析主要划分为以下4种：

​        A、Metric。指标分析类型，如：计算最值，平均值等；

​        B、Bucket。分桶类型，类似于group by语法，根据一定规则划分为若干个桶分类；

​        C、Pipeline。管道分析类型，基于上一级的聚合分析结果进行再分析；

​        D、Matrix。矩阵分析类型。

#####     2.Metric聚合分析

​        主要分为两类：单值分析（输出单个结果）和多值分析（输出多个结果）。

​        1）单值分析

​            A、min。返回数值类型字段的最小值：

```bash
#min关键字


GET my_index/_search
{

 "size": 0,
 "aggs":{
  "min_age":{
   "min":{    #关键字
    "field":"age"    
   }
  }
 }
}
```

​            B、max。返回数值类型字段的最大值。

```bash
#max关键字



GET my_index/_search
{
 "size": 0,
 "aggs":{
  "max_age":{
   "max":{    #关键字
    "field":"age"    
   }
}

 }

}
```

​            C、avg。返回数值类型字段的平均值。

```bash
#avg关键字

GET my_index/_search
{
 "size": 0,
 "aggs":{
  "avg_age":{
   "avg":{    #关键字
    "field":"age"    
   }
  }
 }
}
```

​            D、sum。返回数值类型字段值的总和。

```bash
#sum关键字



GET my_index/_search
{

 "size": 0,
 "aggs":{
  "sum_age":{
   "sum":{    #关键字
    "field":"age"    
   }
  }
 }
}
```

​            E、cardinality。返回字段的基数。

```bash
#cardinality关键字



GET my_index/_search

{
 "size": 0,
 "aggs":{
  "cardinality_age":
   "cardinality":{    #关键字
    "field":"age"    
   }
  }
 }
}
```

​            F、使用多个单值分析关键词，返回多个结果。

```bash
#使用多个单值分析关键词，返回多个分析结果

GET my_index/_search
{

 "size": 0,
 "aggs": {
  "min_age":{
   "min":{    #求最大年龄
    "field":"age"
   }
  },
  "max_age":{
   "max":{    #求最小年龄
    "field":"age"
   }
  },
  "avg_age":{
   "avg":{    #求平均年龄
    "field":"age"
   }
  },
  "sum_age":{
   "sum":{    #求年龄总和
    "field":"age"
   }
  }
 }
}
```

​        2）多值分析

​            A、stats。返回所有单值结果：

```bash
#使用stats关键词



GET my_index/_search
{
 "size": 0,
 "aggs":{
  "stats_age":{
   "stats":{    #关键字
    "field":"age"    
   }
  }
 }
}
```

​            B、extended stats。对stats进行扩展，包含更多，如：方差，标准差，标准差范围等：

```bash
#使用extended_stats关键词
GET my_index/_search

{
 "size": 0,
 "aggs":{
  "stats_age":{
   "extended_stats":{    #关键字
    "field":"age"    
   }

  }

 }
}
```

​            C、Percentile。百分位数统计：

```json
#使用percentiles关键词

GET my_index/_search
{
 "size": 0,
 "aggs":{
  "per_age":{
   "percentiles":{    #关键字
    "field":"age"   
   }
  }
 }

}


```

​            D、Top hits。一般用于分桶之后获取该桶内最匹配的定不稳当列表，即详情数据：

```bash
#使用top_hits关键词


POST _bulk
{ "index":  { "_index": "recipes", "_type": "type"}}
{"name":"清蒸鱼头","rating":1,"type":"湘菜"}
{ "index":  { "_index": "recipes", "_type": "type"}} 
{"name":"剁椒鱼头","rating":2,"type":"湘菜"}
{ "index":  { "_index": "recipes", "_type": "type"}} 
{"name":"红烧鲫鱼","rating":3,"type":"湘菜"}
{ "index":  { "_index": "recipes", "_type": "type"}} 
{"name":"鲫鱼汤（辣）","rating":3,"type":"湘菜"}
{ "index":  { "_index": "recipes", "_type": "type"}} 
{"name":"鲫鱼汤（微辣）","rating":4,"type":"湘菜"}
{ "index":  { "_index": "recipes", "_type": "type"}} 
{"name":"鲫鱼汤（变态辣）","rating":5,"type":"湘菜"}
{ "index":  { "_index": "recipes", "_type": "type"}} 
{"name":"广式鲫鱼汤","rating":5,"type":"粤菜"}
{ "index":  { "_index": "recipes", "_type": "type"}} 
{"name":"鱼香肉丝","rating":2,"type":"川菜"}
{ "index":  { "_index": "recipes", "_type": "type"}} 
{"name":"奶油鲍鱼汤","rating":2,"type":"西菜"}


GET recipes/_search
{
 "size":0,
 "aggs":{
  "jobs":{
   "terms":{
     "field":"name.keyword",
     "size":10
    },
    "aggs":{
     "top_employee":{
      "top_hits":{
       "size":10,
       "sort":[
        {
         "rating":{
          "order":"desc"
         } 
        }
       ]
      }
     }
    }
   }
  }
 }
```

#####     3.Bucket聚合分析

​        Bucket，意为桶。即：按照一定规则，将文档分配到不同的桶中，达分类的目的。常见的有以下五类：

​        1）Terms。直接按term进行分桶，如果是text类型，按分词后的结果分桶：

```bash
#使用terms关键词



GET my_index/_search

{
 "size": 0,
 "aggs":{
  "terms_job":{
   "terms":{    #关键
    "field":"job.keyword",    #按job.keyword进行分桶
    "size":5                  #返回五个文
   }
  }
 }
}
```

​        2）Range。按指定数值范围进行分桶：

```bash
#使用range关键词



GET my_index/_search
{
 "size": 0,
 "aggs":{
  "number_ranges":{
   "range":{    #关键字
    "field":"age",    #按age进行分桶
    "ranges":[
     {
      "key":">=19 && < 25",  #第一个桶：  19<=年龄<25
      "from":19,
      "to":25
     },
     {
      "key":"< 19",    #第二个桶：  年龄<19
      "to":19
     },
     {
      "key":">= 25",    #第三个桶：  年龄>=25
      "from":25
     }
    ]
   }
  }
 }
}
```

​        3）Date Range。按指定日期范围进行分桶：

```bash
#使用date_range关键词



GET my_index/_search

{
 "size": 0,
 "aggs":{
  "date_ranges":{
   "date_range":{    #关键字
    "field":"birth",    #按age进行分桶
    "format":"yyyy",
    "ranges":[
     {
      "key":">=1980 && < 1990",  #第一个桶：  1980<=出生日期<1990
      "from":"1980",
      "to":"1990"
     },

     {

      "key":"< 1980",    #第二个桶：  出生日期<1980
      "to":1980
     },
     {

      "key":">= 1990",    #第三个桶：  出生日期>=199
      "from":1990

     }

    ]
   }
  }
 }
}
```

​        4）Histogram。直方图，按固定数值间隔策略进行数据分割：

```bash
#使用histogram关键词



GET my_index/_search
{
 "size": 0,
 "aggs":{
  "age_hist":{
   "histogram":{     #关键词
    "field":"age",
    "interval":3,    #设定间隔大小为3
    "extended_bounds":{    #设定数据范围
     "min":0,
     "max":30
    }
   }
  }
 }
}
```

​        5）Date Histogram。日期直方图，按固定时间间隔进行数据分割：

```bash
#使用date_histogram关键词



GET my_index/_search
{
 "size": 0,
 "aggs":{
  "birth_hist":{
   "date_histogram":{     #关键词
    "field":"birth",
    "interval":"year",    #设定间隔大小为年year
    "format":"yyyy",
    "extended_bounds":{    #设定数据范围
     "min":"1980",
     "max":"199
    }
   }
  }
 }
}
```

#####     4.Bucket+Metric聚合分析

​         Bucket聚合分析允许通过添加子分析来进一步进行分析，该子分析可以是**Bucket**，也可以是**Metric**。

​         1）分桶之后再分桶（Bucket+Bucket），在数据可视化中一般使用**千层饼图**进行显示。

```json
#分桶之后再分桶——Bucket+Bucket



GET my_index/_search
{
    "size": 0,
    "aggs": {
        "jobs": {
            "terms": {
             
                    "field": "job.keyword",
                    "size": 10
                },
                "aggs": {
                    "age_range": {
                        "range": {
                            "field": "age",
                            "ranges": [
                                {
                                    "to": 20
                                },
                                {
                                    "from": 20,
                                    "to": 30
                                },
                                {
                                    "from": 30
                                }
                            ]
                        }
                    }
                }
            }
        }
    }
```

​         2）分桶之后再数据分析（Bucket+Metric）

```bash
#分桶之后再数据分析——Bucket+Metric

GET my_index/_search
{
    "size": 0,
    "aggs": {
        "jobs": {
            "terms": {
                    "field": "job.keyword",
                    "size": 10
                },
                "aggs": {
                    "stats_age": {
                        "stats": {
                            "field": "age"
                        }
                    }
                }
            }
        }
    }

```

​    5.Pipeline聚合分析

​        针对聚合分析的结果进行再分析，且支持链式调用：

```json
#使用pipeline聚合分析,计算订单月平均销售额。

GET my_index/_search
{
    "size": 0,
    "aggs": {
        "sales_per_month": {
            "date_histogram": {
                "field": "date",
                "interval": "month"
            },
            "aggs": {
                "sales": {
                    "sum": {
                        "field": "price"
                    }
                }
            }
        },
        "avg_monthly_sales": {
            "avg_bucket": {
                "buckets_path": "sales_per_month>sales"
            }
        }
    }
}
```

​        pipeline的分析结果会输出到原结果中，由输出位置不同，分为两类：Parent和Sibling。

​        1）Parent。结果内嵌到现有聚合分析结果中，如：Derivate、Moving Average、Cumulative Sum。

​        2）Sibling。结果与现有聚合分析结果同级，如：Max/Min/Sum/Avg Bucket、Stats/Extended Stats Bucket、Percentiles Bucket。

​        A、Sibling——————min_bucket，其他同理:

```json
#Sibling聚合分析



GET my_index/_search
{
    "size": 0,
    "aggs": {
        "jobs": {
            "terms": {
                "field": "job.keyword",
                "size": 10
            },
            "aggs": {
                "avg_salary": {
                    "avg": {
                        "field": "salary"
                    }
                }
            }
        },
        "min_salary_by_job": {
            "min_bucket": {
                "buckets_path": "jobs>avg_salary"
            }
        }
    }
}
```

​        B、Parent——————Derivate，其他同理：

```bash
#Parent聚合分析

GET my_index/_search
{
    "size": 0,
    "aggs": {
        "bitrh": {
            "date_histogram": {
                "field": "birth",
                "interval": "year",
                "min_doc_count": 0
            },
            "aggs": {
                "avg_salary": {
                    "avg": {
                        "field": "salary"
                    }
                },
                "derivative_avg_salary": {
                    "derivative": {
                        "buckets_path": "avg_salary"
                    }
                }
            }
        }
    }
}
```

#####     6.聚合分析的作用范围

​        ES聚合分析默认作用范围是query的结果集：

```bash
#ES中聚合分析的默认作用范围是query的结果集

GET my_index/_search
{
    "size": 0,
    "query": {
        "match": {
            "username": "alfred"
        }
    },
    "aggs": {
        "jobs": {
            "terms": {
                "match": {
                    "field": "job.keyword",
                    "size": 10
                }
            }
        }
    }
}
```

​        可通过以下方式修改：

​            ①filter。

​            ②post_filter。

​            ③global。

​        1）filter，为某个结合分析设定过滤条件，从而在不改变整体query语句的情况下修改范围。

```bash
#使用filter进行过滤



GET my_index/_search
{
    "size": 0,
    "aggs": {
        "jobs_salary_small": {
            "filter": {
                "range": {
                    "salary": {
                        "to": 10000
                    }
                }
            },
            "aggs": {
                "jobs": {
                    "terms": {
                        "field": "job.keyword"
                    }
                }
            }
        }
    }
}
```

​        2）post_filter，作用于文档过滤，但在聚合分析之后才生效。

```bash
#使用post_filter进行过滤

GET my_index/_search
{
    "size": 0,
    "aggs": {
        "jobs": {
            "terms": {
                "field": "job.keyword"
            }
        }
    },
    "post_filter": {
        "match": {
            "job.keyword": "java engineer"
        }
    }
}
```

​        3）global，无视query条件，基于所有文档进行分析。

```bash
#使用global进行过滤



GET my_index/_search
{
    "query": {
        "match": {
            "job.keyword": "java engineer"
        }
    },
    "aggs": {
        "java_avg_salary": {
            "avg": {
                "field": "salary"
            }
        },
        "all": {
            "global": {
                "aggs": {
                    "avg_salary": {
                        "avg": {
                            "field": "salary"
                        }
                    }
                }
            }
        }
    }
}
```

#####     7.聚合分析中的排序

​        1）可使用自带的关键数据排序，如：

​            _count 文档数、_key按key值

```bash
#使用自带的数据进行排序

GET my_index/_search
{
    "size": 0,
    "aggs": {
        "jobs": {
            "terms": {
                "field": "job.keyword",
                "size": 10,
                "order": [
                    {
                        "_count": "asc"
                    },
                    {
                        "_key": "desc"
                    }
                ]
            }
        }
    }
}
```

​        1）也可使用聚合结果进行排序，如：

```json
#使用聚合结果进行排序

GET my_index/_search
{
    "size": 0,
    "aggs": {
        "salary_hist": {
            "histogram": {},
            "aggs": {
                "age": {
                    "filter": {
                        "range": {
                            "age": {
                                "gte": 10
                            }
                        }
                    },
                    "aggs": {
                        "avg_age": {
                            "field": "age"
                        }
                    }
                }
            }
        }
    }
}
```

#####     8.计算精准度

​        每个Shard上分别计算，由coordinating Node做聚合，多个Shard上分散着若干数据，而coordinating Node无法得知，那么在取数据的时候，比如：top hits的时候，结果有偏差，造成精准度不准确。解决办法有两种：

​        1）直接设置shard数量为1；

​        2）设置shard_size大小，即每次从shard上额外多获取数据，从而提升精准度

```bash
#使用shard_size额外获取数据



GET my_index/_search


{
    "size": 0,
    "aggs": {
        "jobs": {
            "terms": {
                "field": "job.keyword",
                "size": 1,
                "shard_size": 10
            }
        }
    }
}
```

​        terms返回聚合结果中有两个统计值：

​        ![img](https://img-blog.csdnimg.cn/20181204135232346.png)

​        doc_count_error_upper_bound：被一楼的term可能的最大值；

​        sum_other_doc_count：返回结果bucket的term外其他term的文档总数。

![img](https://img-blog.csdnimg.cn/20181204135355472.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0hpd2Vz,size_16,color_FFFFFF,t_70)

​        三者只能取其二。

## 13.ElasticSearch的集群优化*

#####   1.生产环境部署

​        1）遵照官方建议设置所有系统参数。

​            在ES的配置文件中elasticsearch.yml中，尽量只写必备的参数，其他可通过api进行动态设置，随着ES版本的不断升级，很多网上流传的参数，现在已经不再适用，所以不要胡乱复制。

​            建议设置的基本参数有：

​                ①cluster.name

​                ②node.name

​                ③node.master/node.data/node.ingest

​                ④network.host。建议显示指定为服务器的内网ip，切勿直接指定0.0.0.0，很容易直接从外部被修改ES数据。

​                ⑤discovery.zen.ping.unicast.hosts 设置集群其他节点地址，一般设置选举节点即可。

​                ⑥discovery.zen.minimum_master_nodes。一般设置为2，有3个即可。

​                ⑦path.data/path.log

​                ⑧除上述参数外，再根据需要增加其他的静态配置参数，如：refresh优化参数，indices.memory.index_buffer_size。

​        动态设定的参数有transient(短暂的)和persistent(持续的)两种，前者在集群重启后会丢失，后者在集群重启后依然生效。二者都覆盖了yml中的配置，举例：

```bash
#使用transient和persistent动态设置ES集群参数

PUT /_cluster/Settings
{

 "persistent":{    #永久
  "discovery.zen.minimum_master_nodes:2

 },
 "transient":{   #临时
  "indices.store.throttle.max_bytes_per_sec":"50mb"

 }

}
```

​        2）关于JVM内存设定

​            每个节点尽量不要超多31GB。

​            预留一半内存给操作系统，用来做文件缓存。ES的具体内存大小根据node要存储的数据量来估算，为了保证性能：

​            搜索类项目中：内存：数据量   ===>   1：16；

​            日志类项目中：内存：数据量   ===>   1：48/96。

```
假设现有数据1TB，3个node，1个副本，那么：

每个node存储(1+1)*1024 / 3 = 666GB,即700GB左右，做20%预留空间，每个node约存850GB数据。
此时：
如果是搜索类项目，每个node内存约为850/16=53GB，已经超过31GB最大限制；
而：31*16 = 496，意味着每个node最大只能存496GB的数据，则：2024/496=4.08...即至少需要5个节点。
如果是日志类项目，每个node最大能存:31*48=1488GB,则：2024/1488=1.36...，则三个节点已经够了。
```

#####     2.写性能优化

​        在写上面的优化，主要是增大写的吞吐量——EPS(Event Per Second)

​        优化方案：

​            ①Client：多线程写，批量写bulk；

​            ②ES：在高质量数据建模的前提下，主要在refresh、translog和flush之间做文章。

​        1）降低refresh写入内存的频率：

​            A、增大**refresh_interval**，降低实时性，增大每次refresh处理的文件数，默认1s。可以设为-1s，禁止自动refresh。

​            B、增大index buffer大小，参数为：**indices.memory.index_buffer_size**。

​                  此为静态参数，需设定在elasticsea.yml中，默认10%

​        2）降低translog写入磁盘频率，同时会降低容灾能力：

​            A、**index.translog.durability**：设为async；

​                  **index.translog.sync_interval**。设置需要的大小如：120s  =>  每120s才写一次磁盘。

​            B、**index.translog.flush_threshold_size**。默认512m。

​                  即当translog大小超过此值，会触发一次flush，可以调大避免flush过早触发。

​        3）在flush方面，从6.x开始，ES固定每30min执行一次，所以优化点不多，一般都是ES自动完成。

​        4）其他：

​            A、将副本数设置为0，在文档全部写完之后再加副本；

​            B、合理设计shard数，保证shard均匀地分布在所有node上，充分利用node资源：

​                    index.routing.allocation.total_shards_per_node：限定每个索引在每个node上可分配的主副分片数，

​                  如：有5个node，某索引有10个主分片，1个副本(10个副分片)，则：20/5=45,但是实际要设置为5，预防某个node下线后分片迁移失败。

​        写性能优化，主要还是index级别的设置优化。

​        一般在refresh、translog、flush三个方面进行优化；

#####     3.读性能优化

​        主要受以下几方面影响：

​        ①数据模型是否符合业务模型？

​        ②数据规模是否过大？

​        ③索引配置是否优化？

​        ④查询运距是否优化？

​        1）高质量的数据建模

​            A、将需通过cripte脚本动态计算的值，提前计算好作为字段存入文档中；

​            B、尽量使数据模型贴近业务模型

​        2）根据不同数据规模设定不同的SLA(服务等级协议)，万级数据和千万级数据和亿万级数据性能上肯定有差异；

​        3）索引配置优化

​            A、根据数据规模设置合理的分片数，可通过测试得到最适合的分片数；

​            B、分片数并不是越多越好

​        4）查询语句优化

​            A、尽量使用Filter上下文，减少算分场景(Filter有缓存机制，能极大地提升查询性能)；

​            B、尽量不用cript进行字段计算或算分排序等；

​            C、结合profile、explain API分析慢查询语句的症结所在，再去优化数据模型。

#####     4.其他优化点

​        1）如何设定shard数？

​            ES的性能基本是线性扩展的，因此，只需测出一个shard的性能指标，然后根据实际的性能需求就可算出所需的shard数。

​            测试一个shard的流程如下：

​            ①搭建与生产环境相同配置的单节点集群；

​            ②设定一个单分片0副本的索引；

​            ③写入实际生产数据进行测试，获取（写性能指标）；

​            ④针对数据进行查询操作，获取（读性能指标）。

​        2）压力测试工具，可以采用ES自带的esrally，从经验上讲：

​            **如果是搜索引擎场景，单shard大小不超过15GB；**

​            **如果是日志分析场景，单shard大小不超过50GB。**

​            估算索引的总数据大小，除以上述单shard大小，也可得到经验上的分片数。

#####     5.ES集群监控

​        使用官方免费插件X-pack。

​        1）安装与启动：

```bash
#X-pack的安装
cd ~/elasticsearch-6.1.1

bin/elasticsearch-plugin install x-pack
cd ~/kibana-6.1.1
bin/kibana-plugin indtall x-pack

之后重启ES集群即可。
```