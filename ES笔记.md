











# ES入门

### hadoop与mpp

| 比较项目           | MPP                                                       | Hadoop                                                       |
| ------------------ | --------------------------------------------------------- | ------------------------------------------------------------ |
| 平台开放           | 封闭和专有，对于某些技术,甚至不能为非客户下载文档。       | 通过互联网免费提供供应商和社区资源的完全开源                 |
| 可扩展性           | 平均数十个节点，最大100-200个节点                         | 平均100个节点，可扩展至几千个节点                            |
| 可处理数据         | 平均10TB,最大1PB                                          | 平均100TB, 最大PB                                            |
| 延迟               | 10-20毫秒                                                 | 10-20秒                                                      |
| 平均查询时间       | 5-7 秒                                                    | 10-15 分钟                                                   |
| 最大查询时间       | 1-2小时                                                   | 1-2周                                                        |
| 查询优化           | 拥有复杂的企业级优化器                                    | 没有优化器或优化器功能非常有限，有时甚至优化不是基于成本的   |
| 查询调试和分析     | 便于查看的查询执行计划、查询执行统计信息、说明性错误信息  | OOM问题和Java堆转储分析，GC在群集组件上暂停，每个任务的单独日志给你很多有趣的时间 |
| 技术价格           | 每个节点数十万美元                                        | 每个节点免费或高达数千美元                                   |
| 易用性             | 简单友好的SQL界面和简单可编译的数据库内功的数据库内置函数 | SQL不完全符合ANSI标准，用户应该关心执行逻辑，底层数据布局。函数通常需要用Java编写，编译并放在集群上 |
| 目标用户           | 业务分析师                                                | Java开发人员和经验丰富的DBA                                  |
| 单作业冗余         | 低，当MPP节点发生故障时作业失败                           | 高，作业只有当节点管理作业执行失败时才会失败                 |
| 最小数据集         | 任意                                                      | GB                                                           |
| 最大并发性         | 十到数百个查询                                            | 多达10-20个job                                               |
| 技术可扩展性       | 仅使用供应商提供的工具                                    | 混搭                                                         |
| DBA技能等级要求    | 普通DBA                                                   | 需要懂Java编程的高级RDBMSDBA                                 |
| 解决方案实施复杂性 | 一般                                                      | 复杂                                                         |

>  通过这些信息，您可以总结为什么Hadoop不能用作传统企业数据仓库的完全替代品，
>
> 但它可以用作以分布式方式处理大量数据并从数据中获得重要见解的引擎。 Facebook
> 拥有一个300PB的Hadoop，并且仍然使用一个小型的50TB Vertica集群，LinkedIn
> 拥有一个巨大的Hadoop集群，并且仍然使用Aster Data集群（由Teradata购买的MPP），
> 您可以继续使用此列表。

#### 基本术语

 - 在Elasticsearch中存储数据的行为就叫做**索引(indexing)**

 - 在Elasticsearch中，文档归属于一种**类型(type)**,而这些类型存在于**索引(index)**中，我们可以画一些简单的对比图来类比传统关系型数据库

   ```
   Relational DB -> Databases -> Tables -> Rows   -> Columns
   Elasticsearch -> Indices   -> Types  -> Documents -> Fields
   ```

- Elasticsearch集群可以包含多个**索引(indices)**（数据库）

- 每一个**索引**可以包含多个**类型(types)**（表）

- 每一个**类型**包含多个**文档(documents)**（行）

- 每个**文档**包含多个**字段(Fields)**（列）

#### 索引的区分

- 索引（名词） 如上文所述，一个**索引(index)**就像是传统关系数据库中的**数据库**，它是相关文档存储的地方，index的复数是**indices** 或**indexes**
- 索引（动词） **「索引一个文档」**表示把一个文档存储到**索引（名词）**里，以便它可以被检索或者查询。这很像SQL中的`INSERT`关键字，差别是，如果文档已经存在，新的文档将覆盖旧的文档
- 倒排索引 传统数据库为特定列增加一个索引，例如B-Tree索引来加速检索。Elasticsearch和Lucene使用一种叫做**倒排索引(inverted index)**的数据结构来达到相同目的。

#### 索引(建库)示例

- 所以为了创建员工目录，我们将进行如下操作：
  - 为每个员工的**文档(document)**建立索引，每个文档包含了相应员工的所有信息。
  - 每个文档的类型为`employee`。
  - `employee`类型归属于索引`megacorp`。
  - `megacorp`索引存储在Elasticsearch集群中。

```Javascript
PUT /megacorp/employee/1
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
PUT /megacorp/employee/2
{
    "first_name" :  "Jane",
    "last_name" :   "Smith",
    "age" :         32,
    "about" :       "I like to collect rock albums",
    "interests":  [ "music" ]
}

PUT /megacorp/employee/3
{
    "first_name" :  "Douglas",
    "last_name" :   "Fir",
    "age" :         35,
    "about":        "I like to build cabinets",
    "interests":  [ "forestry" ]
}
```

**path:/megacorp/employee/1分析:**

| 名字     | 说明                   |
| -------- | ---------------------- |
| megacorp | 索引名（数据库）       |
| employee | 类型名（表）           |
| 1        | 员工id（文档，列，id） |

#### 地理位置格式

```json
PUT my_index
{
  "mappings": {
    "_doc": {
      "properties": {
        "location": {
          "type": "geo_point"
        }
      }
    }
  }
}

PUT my_index/_doc/1
{
  "text": "Geo-point as an object",
  "location": { 
    "lat": 41.12,
    "lon": -71.34
  }
}

PUT my_index/_doc/2
{
  "text": "Geo-point as a string",
  "location": "41.12,-71.34" 
}
```



#### 简单搜索

**检索单个员工的信息：**

 - 执行HTTP GET请求并指出文档的“地址”——索引、类型和ID既可

   ```Jacscript
   GET /megacorp/employee/1
   ```

- 响应的内容中包含一些文档的元信息，John Smith的原始JSON文档包含在

  **_source**字段中

  ```Javascript
  {
    "_index" :   "megacorp",
    "_type" :    "employee",
    "_id" :      "1",
    "_version" : 1,
    "found" :    true,
    "_source" :  {
        "first_name" :  "John",
        "last_name" :   "Smith",
        "age" :         25,
        "about" :       "I love to go rock climbing",
        "interests":  [ "sports", "music" ]
    }
  }
  ```

  > 我们通过HTTP方法`GET`来检索文档，
  >
  > 我们可以使用`DELETE`方法删除文档，
  >
  > 使用`HEAD`方法检查某文档是否存在。
  >
  > 如果想更新已存在的文档，我们只需再`PUT`一次。

- **搜索全部信息**

​	`GET /megacorp/employee/_search`

- **搜索姓氏中包含“Smith”的员工:**

​	`GET /megacorp/employee/_search?q=last_name:Smith`

#### 使用DSL语句查询

​	Elasticsearch提供丰富且灵活的查询语言叫做**DSL查询(Query DSL)**,它允许你构建更加`复杂`、`强大`的查询。

##### 映射

###### 空值

```json
{
  "mappings": {
    "_doc": {
      "properties": {
        "status_code": {
          "type":       "keyword",
          "null_value": "NULL" 
        }
      }
    }
  }
}
```



##### DSL语法

**DSL(Domain Specific Language特定领域语言)**以JSON请求体的形式出现。我们可以这样表示之前关于“Smith”的查询:

```Javascript
`GET /megacorp/employee/_search`

{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
```

> 这个请求体使用JSON表示，其中使用了`match`语句（查询类型之一）。



- **我们依旧想要找到姓氏为“Smith”的员工，年龄大于30岁的员工**。我们的语句将添加**过滤器(filter)**,它使得我们高效率的执行一个结构化搜索：

  ```javascript
  `GET /megacorp/employee/_search`
  {
      "query" : {
          "filtered" : {
              "filter" : {
                  "range" : {
                      "age" : { "gt" : 30 } <1>
                  }
              },
              "query" : {
                  "match" : {
                      "last_name" : "smith" <2>
                  }
              }
          }
      }
  }
  ```

  > - <1> 这部分查询属于**区间过滤器(range filter)**,它用于查找所有年龄大于30岁的数据——`gt`为"greater than"的缩写。
  > - <2> 这部分查询与之前的`match`**语句(query)**一致。

##### **全文搜索匹配度：**

```Javascript
{
     "last_name":   "John Smith",
     "about":       "I love to go rock climbing",
},
{
     "last_name":   "Jane Smith",
     "about":       "I like to collect rock albums",
}
```

默认情况下，Elasticsearch根据结果相关性评分来对结果集进行排序，所谓的「结果相关性评分」就是文档与查询条件的匹配程度。很显然，排名第一的`John Smith`的`about`字段明确的写到**“rock climbing”**。

但是为什么`Jane Smith`也会出现在结果里呢？原因是**“rock”**在她的`abuot`字段中被提及了。因为只有**“rock”**被提及而**“climbing”**没有，所以她的`_score`要低于John。

这个例子很好的解释了Elasticsearch如何在各种文本字段中进行全文搜索，并且返回相关性最大的结果集。**相关性(relevance)**的概念在Elasticsearch中非常重要，而这个概念在传统关系型数据库中是不可想象的，因为传统数据库对记录的查询只有匹配或者不匹配。

---

##### 删除

语法：indexname/type/_delete_by_query

请求方式：post

dsl:

```json
{
  "query": {
    "match_all": {}
  }
}
```

示例：`basisdata_usr_login_info-201905/information/_delete_by_query`

---

##### 短语搜索 

目前我们可以在字段中搜索单独的一个词，这挺好的，但是有时候你想要确切的匹配若干个单词或者**短语(phrases)**。例如我们想要查询同时包含"rock"和"climbing"（并且是相邻的）的员工记录。

要做到这个，我们只要将`match`查询变更为`match_phrase`查询即可:

---

##### 高亮搜索

从每个搜索结果中**高亮(highlight)**匹配到的关键字，这样用户可以知道为什么这些文档和查询相匹配。在Elasticsearch中高亮片段是非常容易的。

```Javascript
`GET /megacorp/employee/_search`
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    },
    "highlight": {
        "fields" : {
            "about" : {}
        }
    }
}
```

当我们运行这个语句时，会命中与之前相同的结果，但是在返回结果中会有一个新的部分叫做`highlight`，这里包含了来自`about`字段中的文本，并且用`<em></em>`来标识匹配到的单词。

```Javascript
{
   ...
   "hits": {
      "total":      1,
      "max_score":  0.23013961,
      "hits": [
         {
            ...
            "_score":         0.23013961,
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            },
            "highlight": {
               "about": [
                  "I love to go <em>rock</em> <em>climbing</em>" <1>
               ]
            }
         }
      ]
   }
}
```

> - <1> 原有文本中高亮的片段

---

##### 聚合

允许管理者在职员目录中进行一些分析。 Elasticsearch有一个功能叫做**聚合(aggregations)**，它允许你在数据上生成复杂的分析统计。它很像SQL中的`GROUP BY`但是功能更强大。

```Javascript
`GET /megacorp/employee/_search`
{
  "aggs": {
    "all_interests": {
      "terms": { "field": "interests" }
    }
  }
}
```

暂时先忽略语法只看查询结果：

```Javascript
{
   ...
   "hits": { ... },
   "aggregations": {
      "all_interests": {
         "buckets": [
            {
               "key":       "music",
               "doc_count": 2
            },
            {
               "key":       "forestry",
               "doc_count": 1
            },
            {
               "key":       "sports",
               "doc_count": 1
            }
         ]
      }
   }
}
```

where条件+group by 形式查询

```Javascript
`GET /megacorp/employee/_search`
{
  "query": {
    "match": {
      "last_name": "smith"
    }
  },
  "aggs": {
    "all_interests": {
      "terms": {
        "field": "interests"
      }
    }
  }
}
```

结果：

```Javascript
"all_interests": {
     "buckets": [
        {
           "key": "music",
           "doc_count": 2
        },
        {
           "key": "sports",
           "doc_count": 1
        }
     ]
  }
```

聚合也允许分级汇总。例如，让我们统计每种兴趣下职员的平均年龄：

```Javascript
`GET /megacorp/employee/_search`
{
    "aggs" : {
        "all_interests" : {
            "terms" : { "field" : "interests" },
            "aggs" : {
                "avg_age" : {
                    "avg" : { "field" : "age" }
                }
            }
        }
    }
}
```

虽然这次返回的聚合结果有些复杂，但任然很容易理解：

```Javascript
"all_interests": {
     "buckets": [
        {
           "key": "music",
           "doc_count": 2,
           "avg_age": {
              "value": 28.5
           }
        },
        {
           "key": "forestry",
           "doc_count": 1,
           "avg_age": {
              "value": 35
           }
        },
        {
           "key": "sports",
           "doc_count": 1,
           "avg_age": {
              "value": 25
           }
        }
     ]
  }
```

##### 分页

和SQL使用`LIMIT`关键字返回只有一页的结果一样，Elasticsearch接受`from`和`size`参数：

```javascript
size: 结果数，默认`10`
from: 跳过开始的结果数，默认`0`

GET /_search?size=5
GET /_search?size=5&from=5
GET /_search?size=5&from=10
```

#### 最重要的查询过滤语句

Elasticsearch 提供了丰富的查询过滤语句，而有一些是我们较常用到的。 我们将会在后续的《深入搜索》中展开讨论，现在我们快速的介绍一下 这些最常用到的查询过滤语句。

##### 组合过滤条件查询DSL

`bool` 过滤可以用来合并多个过滤条件查询结果的布尔逻辑，它包含一下操作符：

`must` :: 多个查询条件的完全匹配,相当于 `and`。

`must_not` :: 多个查询条件的相反匹配，相当于 `not`。

`should` :: 至少有一个查询条件匹配, 相当于 `or`。

这些参数可以分别继承一个过滤条件或者一个过滤条件的数组：

```json
#多条件组合
{
    "bool": {
        "must":     { "term": { "folder": "inbox" }},
        "must_not": { "term": { "tag":    "spam"  }},
        "should": [
                    { "term": { "starred": true   }},
                    { "term": { "unread":  true   }}
        ]
    }
}
#例如es6.5版本
{
  "from": 1, 
  "size": 2, 
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "DevName" : "输入IO2"
          }
        },
        {
          "range": {
            "LogTime": {
              "gte": "2019-06-15",
              "lte": "2019-06-25"
            }
          }
        }
      ]
    }
  }
}
#例如es2.1版本
{
	"query": {
		"bool": {
			"filter": [
				{
					"bool": {
						"must": [
							{
								"term": {
									"PLATENUMBER": "粤S625BG"
								}
							},
							{
								"term": {
									"PLATETYPE": "02"
								}
							},
							{
								"range": {
									"DATEPART": {
										"from": 20190601,
										"to": 20190708
									}
								}
							}
						]
					}
				}
			]
		}
	},
	"aggs": {
		"group": {
			"terms": {
				"field": "PASSPORTID",
				"size": 10   #只查前10个
			}
		}
	},
	"size": 0
}
```

##### match查询

`match`查询是一个标准查询，不管你需要全文本查询还是精确查询基本上都要用到它。

如果你使用 `match` 查询一个全文本字段，它会在真正查询之前用分析器先分析`match`一下查询字符：

```Javascript
{
    "match": {
        "tweet": "About Search"
    }
}
```

如果用`match`下指定了一个确切值，在遇到数字，日期，布尔值或者`not_analyzed` 的字符串时，它将为你搜索你给定的值：

```Javascript
{ "match": { "age":    26           }}
{ "match": { "date":   "2014-09-01" }}
{ "match": { "public": true         }}
{ "match": { "tag":    "full_text"  }}
```

> **提示**： 做精确匹配搜索时，你最好用过滤语句，因为过滤语句可以缓存数据。

### DSL

##### 过滤日期(搜索)

- 查询时间区间的数据

- ```json
  //GET log_alarmevent_topic/information/_search
  {   
      "size": 100,   
      "query": {  
          "range": {
              "LogTime": {
                  "gte": "2018-07-04",
                  "lt": "2019-07-05"
              }
           }
       }
  }
  ```

##### 建设日期Mapping

- ```json
  //POST indexname/indextype/_mapping
  {
      "properties": {
          "UserIds": {
  			"type": "keyword"
  		},
          "CreateTime": {
  			"format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis",
  			"type": "date"
  		}
      }
  }
  ```

##### 更换索引reindex

- ```json
  #POST _reindex
  {
      "conflicts": "proceed", // 将冲突进行类似于continue的操作
      "source": {
          "remote": {
              "host": "http://otherhost:9200", // 远程es的ip和port列表
              "socket_timeout": "1m",
              "connect_timeout": "10s"  // 超时时间设置
          },
          "index": "my_index_name", // 源索引名称
          "query": {         // 满足条件的数据
              "match": {
                  "test": "data"
              }
          }
      },
    	"dest": {
      	"index": "index_to",
          "op_type": "create" //只会对发生不同的document进行reindex
    	}
  }
  ```

  > 重新建设mapping,数据会自动同步，类型也可以转换
  >
  > 具体使用：https://blog.csdn.net/ctwy291314/article/details/82734667

##### 建设别名

- 为索引secisland添加别名secisland_alias：

  ```json
  #POST _aliases
  {
    "actions": [
      {
        "add": {
          "index": "indexName",
          "alias": "indexName_alias"
        }
      }
    ]
  }
  ```

- 删除别名：

  ```json
  #POST _aliases
  {
    "actions": [
      {
        "remove": {
          "index": "indexName",
          "alias": "indexName_alias"
        }
      }
    ]
  }
  ```

##### 一个别名关联多个索引：

- ```json
  #POST _aliases
  {
    "actions": [
      {
        "remove": {
          "index": "secisland",
          "alias": "secisland_alias"
        }
      },
      {
        "add": {
          "index": "secisland",
          "alias": "secisland_alias"
        }
      },
      {
        "add": {
          "index": "secisland2",
          "alias": "secisland_alias"
        }
      }
    ]
  }
  ```

##### 过滤索引别名：

- 通过过滤索引来指定别名提供了对索引查看的不同视图，该过滤器可以使用查询DSL来定义适用于所有的搜索，计算，查询，删除等，以及更多类似这样的与此别名类似的操作。

- 如以下索引：

  ```json
  # 现有如下索引的映射
  {
      "index3":{
          "mappings":{
              "properties":{
                  "city":{
                      "type":"keyword"
                  },
                  "createTime":{
                      "type":"date"
                  },
                  "message":{
                      "type":"text"
                  }
                  
              }
          }
      }
  }
  
  # post _aliases --创建过滤索引
  {
      "actions":[
          {
              "add":{
                  "index":"index3",
                  "alias":"index3_alias_beijing",
                  "filter":{
                      "term":{
                          "city":"beijing"
                      }
                  }
              }
          }
      ]
  }
  ```

- ## 删除别名

  DELETE secisland3/_alias/secisland3_alias_beijing

  或者：

  DELETE secisland3/_alias/*

##### painless脚本

```json
{
  "query": {
    "range": {
      "PASSTIME": {
        "gte": "2020-03-14 15:04:08",
        "lte": "2020-03-18 15:04:08"
      }
    }
  },
  "script_fields": {
    "rangepasstime": {
      "script": {
        "lang": "painless",
        "source":"doc['ID'].value * params.factor",
        "params": {
          "factor":1
        }
      }
    }
  }
}
```

### es优化

https://www.cnblogs.com/technologykai/articles/10899582.html

https://www.jianshu.com/p/93f25c1f138c

#### 分片设置多少

forcemerge 

**注1**：小的分片会造成小的分段，从而会增加开销。我们的目的是将平均分片大小控制在几 GB 到几十 GB 之间。对于基于时间的数据的使用场景来说，通常将分片大小控制在 20GB 到 40GB 之间。

 **注2**：由于每个分片的开销取决于分段的数量和大小，因此通过 **forcemerge** 操作强制将较小的分段合并为较大的分段，这样可以减少开销并提高查询性能。 理想情况下，一旦不再向索引写入数据，就应该这样做。 请注意，这是一项比较耗费性能和开销的操作，因此[应该在非高峰时段]()执行。

 **注3**：我们可以在[节点上保留的分片数量]()与可用的[堆内存]()成正比，但 Elasticsearch 没有强制的固定限制。 一个好的经验法则是确保每个节点的分片数量低于每GB堆内存配置20到25个分片。 因此，具有30GB堆内存的节点应该具有最多600-750个分片，但是低于该限制可以使其保持更好。 这通常有助于集群保持健康。

**注4**：如果担心数据的快速增长, 建议根据这条限制: [ElasticSearch推荐的最大JVM堆空间](https://www.elastic.co/guide/en/elasticsearch/guide/current/relevance-intro.html) 是 30~32G, 所以把分片最大容量限制为 30GB, 然后再对分片数量做合理估算。例如, 如果的数据能达到 200GB, 则最多分配7到8个分片。

数据分片也是要有相应资源消耗,并且需要持续投入。当索引拥有较多分片时, 为了组装查询结果, ES 必须单独查询每个分片(当然并行的方式)并对结果进行合并。所以高性能 IO 设备(SSDs)和多核处理器无疑对分片性能会有巨大帮助。尽管如此, 还是要多关心数据本身的大小,更新频率以及未来的状态。在分片分配上并没有绝对的答案。

#### 集群数目(jvm分配)

https://www.jianshu.com/p/93f25c1f138c

1. jvm heap分配  （官方jvm堆：30G）

2. 将机器上少于一半的内存分配给es

3. 不要给jvm分配超过32G内存

   > ***\*一旦超过了这个神奇的30-32GB的边界，指针会切换会普通对象指针（oop）\****所以也正是因为32G的限制，一般来说，都是建议说，如果你的es要处理的数据量上亿的话，几亿，或者十亿以内的规模的话，建议，就是用64G的内存的机器比较合适，有个5台，差不多也够了。给jvm heap分配32G，留下32G给os cache。

4. 在32G以内的话具体应该设置heap为多大？

5. 对于有1TB内存的超大内存机器该如何分配？

   > 如果我们的机器是一台超级服务器，内存资源甚至达到了1TB，或者512G，128G，该怎么办？首先es官方是建议避免用这种超级服务器来部署es集群的，但是如果我们只有这种机器可以用的话，我们要考虑以下几点：
   >
   > 1. 我们是否在做大量的全文检索？考虑一下分配4~32G的内存给es进程，同时给lucene留下其余所有的内存用来做os filesystem cache。所有的剩余的内存都会用来cache segment file，而且可以提供非常高性能的搜索，几乎所有的数据都是可以在内存中缓存的，es集群的性能会非常高
   > 2. 是否在做大量的排序或者聚合操作？聚合操作是不是针对数字、日期或者[未分词]()的string？如果是的化，那么还是给es 4~32G的内存即可，其他的留给es filesystem cache，可以将聚合好用的正排索引，doc values放在os cache中
   > 3. 如果在[针对分词的string]()做大量的排序或聚合操作？如果是的化，那么就需要使用fielddata，这就得给jvm heap分配更大的内存空间。此时[不建议运行一个节点在机器上]()，而是运行多个节点在一台机器上，那么如果我们的服务器有128G的内存，可以运行两个es节点，然后每个节点分配32G的内存，剩下64G留给os cache。

6. swapping

   > swap（交换区）是性能终结者
   >
   > https://blog.csdn.net/u013063153/article/details/60876897
   >
   > 如果两种方法都不可用，你应该在ElasticSearch的配置中启用mlockall.file。这允许JVM锁定其使用的内存，而避免被放入操作系统交换区。
   >
   > 在elasticsearch.yml中，做如下设置：
   >
   > ```
   > bootstrap.mlockall: true 
   > ```
   >
   > 

机器性能：内存64G

单个节点：32G -> 3 * 32G = 96G

#### scroll

- **Scroll-Scan**：Scroll 是先做一次初始化搜索把所有符合搜索条件的结果[缓存起来生成一个快照]()，然后持续地、批量地[从快照里拉取数据]()直到没有数据剩下。

  而这时对索引数据的插入、删除、更新都不会影响遍历结果，因此 Scroll 并[不适合用来做实时搜索]()。

- 其思路和使用方式与 Scroll 非常相似，但是 Scroll-Scan 关闭了 Scroll 中最耗时的[文本相似度计算和排序]()，使得性能更加高效

>  注意：Elasticsearch 2.1.0 版本之后移除了 search_type=scan，使用 "sort": [ "_doc"] 进行代替。

**Scroll** 和 **Scroll-Scan** 有一些差别，如下所示：

- Scroll-Scan不进行文本相似度计算，不排序，按照索引中的数据顺序返回。
- Scroll-Scan 不支持聚合操作。
- Scroll-Scan 的参数 Size 代表着每个分片上的请求的结果数量，每次返回 n×size 条数据。而 Scroll 每次返回 size 条数据。

#### **角色隔离和脑裂**

ES 集群中的数据节点负责对数据进行增、删、改、查和聚合等操作，所以对 CPU、内存和 I/O 的消耗很大。

在搭建 ES 集群时，我们应该对 ES 集群中的节点进行角色划分和隔离。 

- **候选主节点：**

```
node.master=true
node.data=false
```

- **数据节点：**

```
node.master=false
node.data=true
```

- **避免脑裂**

网络异常可能会导致集群中节点划分出多个区域，区域发现没有 Master 节点的时候，会选举出了自己区域内 Maste 节点 r，导致一个集群被分裂为多个集群，使集群之间的数据无法同步，我们称这种现象为脑裂。

为了防止脑裂，我们需要在 Master 节点的配置文件中添加如下参数：

```properties
discovery.zen.minimum_master_nodes=（master_eligible_nodes/2）+1 //默认值为1
```

其中 master_eligible_nodes 为 Master 集群中的节点数。这样做可以避免脑裂的现象都出现，最大限度地提升集群的高可用性。

只要不少于 discovery.zen.minimum_master_nodes 个候选节点存活，选举工作就可以顺利进行。

#### **ES 配置说明**

下面我们列出一些比较重要的配置信息：

- **cluster.name：elasticsearch：**配置 ES 的集群名称，默认值是 ES，建议改成与所存数据相关的名称，ES 会自动发现在同一网段下的集群名称相同的节点。

- **node.nam： "node1"：**集群中的节点名，在同一个集群中不能重复。节点的名称一旦设置，就不能再改变了。当然，也可以设置成服务器的主机名称，例如 node.name:${HOSTNAME}。

- **noed.master：true：**指定该节点是否有资格被选举成为 Master 节点，默认是 True，如果被设置为 True，则只是有资格成为 Master 节点，具体能否成为 Master 节点，需要通过选举产生。

- **node.data：true：**指定该节点是否存储索引数据，默认为 True。数据的增、删、改、查都是在 Data 节点完成的。

- **index.number_of_shards：5：**设置都索引分片个数，默认是 5 片。也可以在创建索引时设置该值，具体设置为多大都值要根据数据量的大小来定。如果数据量不大，则设置成 1 时效率最高。

- **index.number_of_replicas：1：**设置默认的索引副本个数，默认为 1 个。副本数越多，集群的可用性越好，但是写索引时需要同步的数据越多。

- **path.conf：/path/to/conf：**设置配置文件的存储路径，默认是 ES 目录下的 Conf 文件夹。建议使用默认值。

- **path.data：/path/to/data1,/path/to/data2：**设置索引数据多存储路径，默认是 ES 根目录下的 Data 文件夹。切记不要使用默认值，因为若 ES 进行了升级，则有可能数据全部丢失。

  可以用半角逗号隔开设置的多个存储路径，在多硬盘的服务器上设置多个存储路径是很有必要的。

- **path.logs：/path/to/logs：**设置日志文件的存储路径，默认是 ES 根目录下的 Logs，建议修改到其他地方。

- **path.plugins：/path/to/plugins：**设置第三方插件的存放路径，默认是 ES 根目录下的 Plugins 文件夹。

- **bootstrap.mlockall：true：**设置为 True 时可锁住内存。因为当 JVM 开始 Swap 时，ES 的效率会降低，所以要保证它不 Swap。

- **network.bind_host：192.168.0.1：**设置本节点绑定的 IP 地址，IP 地址类型是 IPv4 或 IPv6，默认为 0.0.0.0。

- **network.publish_host：192.168.0.1：**设置其他节点和该节点交互的 IP 地址，如果不设置，则会进行自我判断。

- **network.host：192.168.0.1：**用于同时设置 bind_host 和 publish_host 这两个参数。

- **http.port：9200：**设置对外服务的 HTTP 端口，默认为 9200。ES 的节点需要配置两个端口号，一个对外提供服务的端口号，一个是集群内部使用的端口号。

  http.port 设置的是对外提供服务的端口号。注意，如果在一个服务器上配置多个节点，则切记对端口号进行区分。

- **transport.tcp.port：9300：**设置集群内部的节点间交互的 TCP 端口，默认是 9300。注意，如果在一个服务器配置多个节点，则切记对端口号进行区分。

- **transport.tcp.compress：true：**设置在节点间传输数据时是否压缩，默认为 False，不压缩。

- **discovery.zen.minimum_master_nodes：1：**设置在选举 Master 节点时需要参与的最少的候选主节点数，默认为 1。如果使用默认值，则当网络不稳定时有可能会出现脑裂。

  合理的数值为(master_eligible_nodes/2)+1，其中 master_eligible_nodes 表示集群中的候选主节点数。

- **discovery.zen.ping.timeout：3s：**设置在集群中自动发现其他节点时 Ping 连接的超时时间，默认为 3 秒。

  在较差的网络环境下需要设置得大一点，防止因误判该节点的存活状态而导致分片的转移。

- **action.destructive_requires_name: true：**进行全部索引删除是很危险的，我们可以通过在配置文件中添加下面的配置信息，来关闭使用 _all 和使用通配符删除索引的接口，使用删除索引职能通过索引的全称进行。

#### ElasticSearch分片不均匀，集群负载不均衡(并发大)

https://blog.csdn.net/qq_20545159/article/details/80549335

#### xpack代码

https://www.cnblogs.com/WSPJJ/articles/11121138.html

