# Elasticsearch使用手册

# 一、Elasticsearch 基本命令及示例

Elasticsearch（简称 ES）是一个基于 Lucene 的开源分布式搜索和分析引擎，常用于全文搜索、结构化搜索、分析等场景。以下是一些常用的基本命令及示例：

## 1. 索引使用

### 1.1 索引说明

```json
   {
  "settings": {
    "number_of_shards": "1",
    "number_of_replicas": "2",
    "refresh_interval": "2s",
    "analysis": {
      "filter": {
        "tyt_synonym_filter": {
          "type": "synonym_graph",
          "synonyms_path": "analysis/tyt_synonyms.txt"
        }
      },
      "analyzer": {
        "tyt_synonym": {
          "filter": [
            "lowercase",
            "tyt_synonym_filter"
          ],
          "tokenizer": "ik_smart"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "id": {
        "type": "long"
      },
      "sort": {
        "type": "long"
      },
      "startPoint": {
        "type": "text",
        "fields": {
          "keyword": {
            "ignore_above": 256,
            "type": "keyword"
          }
        }
      },
      "destPoint": {
        "type": "text",
        "fields": {
          "keyword": {
            "ignore_above": 256,
            "type": "keyword"
          }
        }
      },
      "taskContent": {
        "type": "text",
        "fields": {
          "keyword": {
            "ignore_above": 501,
            "type": "keyword"
          },
          "synonym": {
            "type": "text",
            "analyzer": "tyt_synonym"
          }
        }
      },
      "tel": {
        "type": "keyword"
      },
      "pubTime": {
        "type": "keyword"
      },
      "pubQq": {
        "type": "long"
      },
      "nickName": {
        "type": "keyword"
      },
      "decryptNickName": {
        "type": "keyword"
      },
      "userShowName": {
        "type": "keyword"
      },
      "status": {
        "type": "keyword"
      },
      "source": {
        "type": "integer"
      },
      "ctime": {
        "type": "date"
      },
      "mtime": {
        "type": "date"
      },
      "uploadCellphone": {
        "type": "keyword"
      },
      "resend": {
        "type": "integer"
      },
      "startCoord": {
        "type": "keyword"
      },
      "destCoord": {
        "type": "keyword"
      },
      "platId": {
        "type": "integer"
      },
      "verifyFlag": {
        "type": "integer"
      },
      "price": {
        "type": "integer"
      },
      "userId": {
        "type": "long"
      },
      "priceCode": {
        "type": "keyword"
      },
      "startCoordX": {
        "type": "integer"
      },
      "startCoordY": {
        "type": "integer"
      },
      "destCoordX": {
        "type": "integer"
      },
      "destCoordY": {
        "type": "integer"
      },
      "startDetailAdd": {
        "type": "text",
        "fields": {
          "keyword": {
            "ignore_above": 256,
            "type": "keyword"
          }
        }
      },
      "startLongitude": {
        "type": "integer"
      },
      "startLatitude": {
        "type": "integer"
      },
      "destDetailAdd": {
        "type": "text",
        "fields": {
          "keyword": {
            "ignore_above": 256,
            "type": "keyword"
          }
        }
      },
      "destLongitude": {
        "type": "integer"
      },
      "destLatitude": {
        "type": "integer"
      },
      "pubDate": {
        "type": "date"
      },
      "goodsCode": {
        "type": "keyword"
      },
      "weightCode": {
        "type": "keyword"
      },
      "weight": {
        "type": "keyword"
      },
      "length": {
        "type": "keyword"
      },
      "wide": {
        "type": "keyword"
      },
      "high": {
        "type": "keyword"
      },
      "isSuperelevation": {
        "type": "integer"
      },
      "linkman": {
        "type": "keyword"
      },
      "remark": {
        "type": "text",
        "fields": {
          "keyword": {
            "ignore_above": 501,
            "type": "keyword"
          }
        }
      },
      "distance": {
        "type": "integer"
      },
      "pubGoodsTime": {
        "type": "date"
      },
      "tel3": {
        "type": "keyword"
      },
      "tel4": {
        "type": "keyword"
      },
      "displayType": {
        "type": "keyword"
      },
      "hashCode": {
        "type": "keyword"
      },
      "isCar": {
        "type": "keyword"
      },
      "userType": {
        "type": "integer"
      },
      "pcOldContent": {
        "type": "text",
        "fields": {
          "keyword": {
            "ignore_above": 256,
            "type": "keyword"
          }
        }
      },
      "resendCounts": {
        "type": "integer"
      },
      "verifyPhotoSign": {
        "type": "integer"
      },
      "userPart": {
        "type": "integer"
      },
      "startCity": {
        "type": "keyword"
      },
      "startProvinc": {
        "type": "keyword"
      },
      "startArea": {
        "type": "keyword"
      },
      "srcMsgId": {
        "type": "long"
      },
      "destProvinc": {
        "type": "keyword"
      },
      "destCity": {
        "type": "keyword"
      },
      "destArea": {
        "type": "keyword"
      },
      "clientVersion": {
        "type": "keyword"
      },
      "isInfoFee": {
        "type": "keyword"
      },
      "infoStatus": {
        "type": "keyword"
      },
      "tsOrderNo": {
        "type": "keyword"
      },
      "releaseTime": {
        "type": "date"
      },
      "regTime": {
        "type": "date"
      },
      "type": {
        "type": "keyword"
      },
      "brand": {
        "type": "keyword"
      },
      "goodTypeName": {
        "type": "keyword"
      },
      "goodNumber": {
        "type": "integer"
      },
      "isStandard": {
        "type": "integer"
      },
      "matchItemId": {
        "type": "integer"
      },
      "androidDistance": {
        "type": "integer"
      },
      "iosDistance": {
        "type": "keyword"
      },
      "isDisplay": {
        "type": "integer"
      },
      "referLength": {
        "type": "integer"
      },
      "referWidth": {
        "type": "integer"
      },
      "referHeight": {
        "type": "integer"
      },
      "referWeight": {
        "type": "integer"
      },
      "startCoordXValue": {
        "type": "keyword"
      },
      "startCoordYValue": {
        "type": "keyword"
      },
      "destCoordXValue": {
        "type": "keyword"
      },
      "destCoordYValue": {
        "type": "keyword"
      },
      "startLatitudeValue": {
        "type": "keyword"
      },
      "startLongitudeValue": {
        "type": "keyword"
      },
      "destLatitudeValue": {
        "type": "keyword"
      },
      "destLongitudeValue": {
        "type": "keyword"
      },
      "distanceValue": {
        "type": "keyword"
      },
      "changeTime": {
        "type": "date"
      },
      "carLength": {
        "type": "keyword"
      },
      "carType": {
        "type": "keyword"
      },
      "specialRequired": {
        "type": "keyword"
      },
      "similarityCode": {
        "type": "keyword"
      },
      "similarityFirstId": {
        "type": "long"
      },
      "similarityFirstInfo": {
        "type": "keyword"
      },
      "tyreExposedFlag": {
        "type": "keyword"
      },
      "carLengthLabels": {
        "type": "text",
        "fields": {
          "keyword": {
            "ignore_above": 256,
            "type": "keyword"
          }
        }
      },
      "loadingTime": {
        "type": "date"
      },
      "unloadTime": {
        "type": "date"
      },
      "carMinLength": {
        "type": "float"
      },
      "carMaxLength": {
        "type": "float"
      },
      "carStyle": {
        "type": "keyword"
      },
      "workPlaneMinHigh": {
        "type": "float"
      },
      "workPlaneMaxHigh": {
        "type": "float"
      },
      "workPlaneMinLength": {
        "type": "float"
      },
      "workPlaneMaxLength": {
        "type": "float"
      },
      "climb": {
        "type": "keyword"
      },
      "orderNumber": {
        "type": "integer"
      },
      "beginLoadingTime": {
        "type": "date"
      },
      "beginUnloadTime": {
        "type": "date"
      },
      "evaluate": {
        "type": "integer"
      },
      "shuntingQuantity": {
        "type": "integer"
      },
      "firstPublishType": {
        "type": "integer"
      },
      "publishType": {
        "type": "integer"
      },
      "infoFee": {
        "type": "double"
      },
      "isDelete": {
        "type": "integer"
      },
      "exclusiveType": {
        "type": "integer"
      },
      "totalScore": {
        "type": "double"
      },
      "rankLevel": {
        "type": "integer"
      },
      "isShow": {
        "type": "integer"
      },
      "refundFlag": {
        "type": "integer"
      },
      "sourceType": {
        "type": "integer"
      },
      "tradeNum": {
        "type": "integer"
      },
      "authName": {
        "type": "keyword"
      },
      "labelJson": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      },
      "guaranteeGoods": {
        "type": "integer"
      },
      "creditRetop": {
        "type": "integer"
      },
      "sortType": {
        "type": "integer"
      },
      "priorityRecommendExpireTime": {
        "type": "date"
      },
      "excellentGoods": {
        "type": "integer"
      },
      "excellentGoodsTwo": {
        "type": "integer"
      },
      "tecServiceFee": {
        "type": "double"
      },
      "machineRemark": {
        "type": "keyword"
      },
      "driverDriving": {
        "type": "integer"
      },
      "loadCellPhone": {
        "type": "keyword"
      },
      "unloadCellPhone": {
        "type": "keyword"
      },
      "cargoOwnerId": {
        "type": "long"
      },
      "invoiceTransport": {
        "type": "integer"
      },
      "useCarType": {
        "type": "integer"
      },
      "additionalPrice": {
        "type": "keyword"
      },
      "enterpriseTaxRate": {
        "type": "double"
      },
      "suggestMinPrice": {
        "type": "integer"
      },
      "suggestMaxPrice": {
        "type": "integer"
      },
      "goodModelScore": {
        "type": "float"
      },
      "commissionScore": {
        "type": "float"
      },
      "topFlag": {
        "type": "integer"
      },
      "seckillGoods": {
        "type": "integer"
      }
    }
  }
}
```
```json
{
  "settings" : {
    "index" : {
      "number_of_shards" : "1",
      "number_of_replicas" : "1",
      "analysis" : {
        "analyzer" : {
          "tyt_analyzer" : {
            "tokenizer" : "ik_smart",
            "filter" : [
              "tyt_synonym_filter"
            ]
          }
        },
        "filter":{
          "tyt_synonym_filter" : {
            "type" : "synonym_graph",
            "synonyms_path" : "analysis/tyt_synonyms.txt"
          }
        }
      },
      "similarity": {
        "tyt_similarity": {
          "type": "BM25",
          "discount_overlaps": false
        }
      }
    }
  },
  "mappings" : {
    "properties" : {
      "id" : {
        "type" : "long"
      },
      "show_name" : {
        "type" : "text",
        "fields" : {
          "keyword" : {
            "type" : "keyword",
            "ignore_above" : 256
          }
        },
        "index_options" : "docs",
        "analyzer" : "tyt_analyzer",
        "similarity": "tyt_similarity"
      },
      "second_class" : {
        "type" : "keyword"
      },
      "brand" : {
        "type" : "keyword"
      },
      "top_type" : {
        "type" : "keyword"
      },
      "display" : {
        "type" : "integer"
      },
      "ctime" : {
        "type" : "date",
        "format": "yyyy-MM-dd HH:mm:ss"
      },
      "mtime" : {
        "type" : "date",
        "format": "yyyy-MM-dd HH:mm:ss"
      },
      "brand_type" : {
        "type" : "keyword",
        "index" : false
      },
      "height" : {
        "type" : "double",
        "index":false
      },
      "length" : {
        "type" : "double",
        "index":false
      },
      "remarks" : {
        "type" : "text",
        "index":false
      },
      "score" : {
        "type" : "double",
        "index":false
      },
      "second_type" : {
        "type" : "keyword",
        "index" : false
      },
      "top_class" : {
        "type" : "keyword",
        "index" : false
      },
      "weight" : {
        "type" : "double",
        "index":false
      },
      "width" : {
        "type" : "double",
        "index":false
      },
      "general_matches_item" : {
        "type" : "keyword",
        "index":false
      }
    }
  }
}
```

#### 1.1.1 settings：定义索引的基本属性和分析配置

##### （1）number\_of\_shards：索引的主分片数量。主分片是存储数据的基本单元，单个节点环境下通常设为 1

##### （2）number\_of\_replicas：每个主分片的副本数量，用于提高查询性能和数据冗余

##### （3）refresh\_interval：索引刷新间隔。刷新操作将内存中的数据写入磁盘，影响搜索实时性和性能

##### （4）analyzer：分析器在索引和搜索中关键作用是将文本转为结构化信息，通过分词、小写化等处理提高搜索准确性和性能，ES 内置多种分析器，如标准、简单、空格分析器。

       **char\_filter：**对输入的文本字符进行第一步处理，如去除html标签(html\_strip)，将表情字符转换成英文单词（mapping）等

       **tokenizer：**分词器。有ik\_smart，ik\_max\_word等。 

       常用分词器介绍：[https://blog.csdn.net/qq\_31960623/article/details/119811859](https://blog.csdn.net/qq_31960623/article/details/119811859)

        **filter :** 对一个token集合的元素做过滤和转换(修改)，删除等操作。例如停用过滤词，使用同义词（synonym\_graph）等

##### （5）similarity：相似度算法。计算每项得分的一种算法。

#### 1.1.2 mappings：定义索引的文档结构和字段类型

##### （1）properties：包含所有字段的定义，常用字段类型有

*   text：用于全文搜索的文本字段，会进行分词处理
    
*   keyword：用于精确匹配的字符串，不分词（如标签、ID）
    
*   date：日期类型，支持多种格式（如 "2024-01-01"）
    
*   integer/long/float：数值类型
    
*   boolean：布尔类型
    
*   nested：嵌套对象类型，用于处理数组中的对象
    
*   geo\_point：地理位置坐标（经纬度）
    

*   index：是否索引该字段（true/false）
    
*   store：是否单独存储该字段（默认只存储在\_source 中）
    
*   format：日期格式（如 "yyyy-MM-dd HH:mm:ss"）
    
*   fields：多字段映射，用于同一字段的不同处理方式
    

### 1.2 索引操作：

#### 1.2.1 创建索引

使用 PUT 请求创建一个名为my\_index的索引，并定义其映射（Mapping）。映射用于定义文档的字段及其数据类型。

```json
  
PUT /my_index
{
  "settings" : {
    "index" : {
      "number_of_shards" : "1",
      "number_of_replicas" : "1"
    }
  },
  "mappings" : {
    "properties" : {
      "id" : {
        "type" : "long"
      },
      "name" : {
        "type" : "text",
        "fields" : {
          "keyword" : {
            "type" : "keyword",
            "ignore_above" : 256
          }
        }
      },
      "age" : {
        "type" : "integer"
      },
      "tall" : {
        "type" : "long",
        "index" : false
      },
      "weight" : {
        "type" : "double"
      },
      "level" : {
        "type" : "keyword"
      },
      "remark" : {
        "type" : "text"
      },
      "time" : {
        "type" : "date",
        "format": "yyyy-MM-dd HH:mm:ss"
      }
    }
  }
}

```

#### 1.2.2 查看索引

使用 GET 请求查看指定索引的详细信息，包括映射和设置

```json
GET /my_index

```

#### 1.2.3 删除索引：

使用 DELETE 请求删除指定索引。

```json
DELETE /my_index
```

#### 1.2.4 查询所有索引：

查询所有索引。可以看见每个索引的文档数量，以及索引大小。

```json
GET /_cat/indices?v
GET /_cat/indices/my*?v
```

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/Q35O851GbJR4Jl9V/img/d611eb6e-d44c-4796-8a26-20a7f19fb23a.png)

#### 1.2.5 修改setting数据：

使用 PUT 修改setting数据。修改副本数或者刷新时间时可直接修改，无需重新索引。修改分片数据或分析器等数据时会重新索引数据，期间服务不可用，需谨慎操作。

```json
PUT /my_index/_settings
{
  "index.number_of_replicas": 2 # 可动态设置
  "index.refresh_interval": 3  # 可动态设置
  "index.number_of_shards": 5  # 无法直接修改，需重建
}

```

#### 1.2.6 修改mapping数据：

使用 PUT 修改mapping数据。新增properties，修改index、store、similarity时可动态修改。type、analyzer、fields等属性需要重新索引，不可动态修改。

例：新增字段只会影响后续写入的文档，现有文档不会自动包含该字段，其值默认为null，及时生效，无需重新索引现有数据。

```json
PUT /my_index/_mapping
{
  "properties": {
    "tags": {
      "type": "keyword"
    }
  }
}
```

如果需要修改字段类型或者分析器，可重建索引

```json
# 重建索引并修改字段类型
POST _reindex?conflicts=proceed #防止文档冲突，不设置变更的文档可能被跳过
{
  "source": {
    "index": "old_index",
    "size": 1000,  # 每次处理1000条文档
    "query": {
      "range": {
        "timestamp": {
          "gte": "2023-01-01"  // 只迁移2023年之后的文档
        }
      }
  },
  "dest": {
    "index": "new_index",
    "mapping": {
      "properties": {
        "age": {
          "type": "integer"  # 原字段为 "long"
        }
      }
    }
  }
}
```

#### 1.2.7 分词分析：

```json
POST /_analyze
{
  "analyzer": "standard",  # 指定分析器类型
  "text": "Hello World!"
}
```
```json
POST /my_index/_analyze
{
  "field": "remark", 
  "text": "helian"
}
```

## 2. 文档操作

#### 2.1.1 创建文档：

指定id：使用PUT方式。适用场景如将数据库中的数据同步到es中，并且di保持一致，使用该种方式时如果 id 已存在则会更新文档。

自动生成id：使用es生成id时用POST方式。

create方式：id不存在时才创建文档，否则返回错误

```json
PUT /my_index/_doc/1  #指定自定义id
{

  "name": "liudehua",
  "age": 18,
  "time": "2024-06-18 12:00:00"
}

POST /my_index/_doc   #使用es生成的id
{

  "name": "liuyifei",
  "age": 18,
  "time": "2024-06-18 12:00:00"
}

PUT /my_index/_create/{id}  #只有id不存在时才创建文档，否则返回错误
{
  "name": "liuyifei",
  "age": 18,
  "time": "2024-06-18 12:00:00"
}
```

#### 2.1.2 批量创建文档：

create：文档不存在，进行插入，文档存在，返回409 Conflict 错误

index：文档不存在，进行插入，文档存在，进行更新

```json
POST /_bulk
{ "create" : { "_index" : "my_index","_id": "1" } }
{   "name": "helian","age":18,"tall":180,"weight":68,"level":"练气","remark":"第一次中大奖500万元","time": "2024-06-18 12:00:00" }
{ "create" : { "_index" : "my_index","_id": "1" } }
{ "name": "gaozongzheng","age":18,"tall":170,"weight":65,"level":"筑基","remark":"好人","time": "2024-06-18 12:00:00" }
{ "create" : { "_index" : "my_index","_id": "1"} }
{ "name": "xuyuyuan","age":18,"tall":170,"weight":67,"level":"元婴","remark":"第18个十八岁生日","time": "2024-06-18 12:00:00" }
{ "create" : { "_index" : "my_index","_id": "1" } }
{ "name": "zhanglubin","age":15,"tall":180,"weight":60,"level":"化神","remark":"特运通第一美男","time": "2024-06-18 12:00:00" }
{ "create" : { "_index" : "my_index","_id": "1"} }
{ "name": "kangcong","age":20,"tall":175,"weight":65,"level":"炼虚","remark":"天王盖地虎，宝塔镇河妖","time": "2024-06-18 12:00:00" }
{ "create" : { "_index" : "my_index","_id": "1"} }
{ "name": "liugang","age":30,"tall":175,"weight":75,"level":"合体","remark":"清风拂山岗，明月照大江","time": "2024-06-18 12:00:00" }
POST /_bulk
{ "index" : { "_index" : "my_index","_id": "1" } }
{   "name": "helian","age":18,"tall":180,"weight":68,"level":"练气","remark":"第一次中大奖500万元","time": "2024-06-18 12:00:00" }
{ "index" : { "_index" : "my_index","_id": "1"} }
{ "name": "gaozongzheng","age":18,"tall":170,"weight":65,"level":"筑基","remark":"收到第18张好人卡","time": "2024-06-18 12:00:00" }
{ "index" : { "_index" : "my_index","_id": "1"} }
{ "name": "xuyuyuan","age":18,"tall":170,"weight":67,"level":"元婴","remark":"第18个十八岁生日","time": "2024-06-18 12:00:00" }
{ "index" : { "_index" : "my_index","_id": "1"} }
{ "name": "zhanglubin","age":15,"tall":180,"weight":60,"level":"化神","remark":"特运通第一美男","time": "2024-06-18 12:00:00" }
{ "index" : { "_index" : "my_index","_id": "1"} }
{ "name": "kangcong","age":20,"tall":175,"weight":65,"level":"炼虚","remark":"天王盖地虎，宝塔镇河妖","time": "2024-06-18 12:00:00" }
{ "index" : { "_index" : "my_index","_id": "1"} }
{ "name": "liugang","age":30,"tall":175,"weight":75,"level":"合体","remark":"清风拂山岗，明月照大江","time": "2024-06-18 12:00:00" }

```

#### 2.1.3 查询文档：

```json
GET /my_index/_search
```
```json
GET /my_index/_doc/1
```
```json
GET /my_index/_search
{
  "query": {
    "query_string": {
      "query": "name:helian AND age:{17 TO *}"
    }
  }
}

GET /my_index/_search
{
  "query": {
    "simple_query_string": {
      "query": "18",
      "fields": ["remark"]
    }
  }
}
```
```json
GET /my_index/_search
{
  "query": {
    "bool": {
      "must": [
        {"term": {"name.keyword": "helian"}},  
        {"range": {"age": {"gte": 18}}}       
      ]
    }
  }
}
GET /my_index/_search
{
  "query": {
    "bool": {
      "must": [{"match": {"name": "helian"}}],
      "filter": [{"term": {"age": 18}}],
      "must_not": [{"range": {"age": {"gt": 30}}}],
      "should": [{"terms": {"name.keyword": ["helian", "gaozongzheng"]}}]
    }
  }
}
```
```json

GET /my_index/_search
{
  "query": {
    "range": {
      "age": {
        "gte": 19,  
        "lt": 50   
      }
    }
  }
}
```
```json

GET /my_index/_search
{
  "query": {
    "match": {
      "remark": "第一"
    }
  }
}

GET /my_index/_search
{
  "query": {
    "multi_match": {
      "query": "18",
      "fields": ["age", "remark"]  
    }
  }
}

GET /my_index/_search
{
  "query": {
    "match_phrase": {
      "remark":{
        "query": "清风明月",
         "slop": 3 
      } 
    }
  }
}
```
```json
GET /my_index/_search
{
  "query": {
    "term": {
      "level": "练气"
    }
  }
}
```

|  **特性**  |  **term 查询**  |  **match 查询**  |
| --- | --- | --- |
|  分词处理  |  不进行分词，直接匹配原始值  |  先对查询文本分词，再匹配分词结果  |
|  适用字段类型  |  keyword、date等精确值类型  |  text类型（全文搜索）  |
|  匹配逻辑  |  完全精确匹配（包括大小写、空格）  |  模糊匹配（只要包含任意分词即可）  |
|  典型场景  |  ID、标签、状态等精确匹配  |  文章内容、标题、描述等全文搜索  |

```json
GET /my_index/_search
{
  "query": {
    "script": {
      "script": {
        "lang": "painless",
        "source": "doc['age'].value > params.min_age && doc['age'].value < params.max_age",
        "params": {
          "min_age": 19,
          "max_age": 29
        }
      }
    }
  }
}
```

#### 2.1.4 查询评分

 评分查询是 Elasticsearch 中一种强大的复合查询，通过function\_score实现，它允许你在查询结果的基础上，通过自定义函数来修改文档的评分（\_score），从而实现更精准的排序控制。Elasticsearch 默认使用 BM25 算法计算文档相关性评分，但在某些场景下，需要根据业务逻辑调整评分，例如：

*   近期文档权重更高（时间衰减）
    
*   热门内容优先展示（基于浏览量、点赞数）
    
*   地理位置近的结果优先（距离衰减）
    

通过function\_score，可以：

*   对基础查询结果应用一个或多个评分函数
    
*   组合多个函数的结果（相乘、相加等）
    
*   控制评分的上限（避免异常值影响排序）
    

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/Q35O851GbJR4Jl9V/img/d39a42cb-145a-4064-aa27-1b8cf5da7dff.png)

**常用的评分函数（functions）：**

weight ：为所有匹配文档设置固定权重

field\_value\_factor ：基于文档字段值调整评分（如浏览量、拨打量）

gauss /linear/exp ：基于距离、时间等进行衰减评分

*   linear : 线性函数是条直线，一旦直线与横轴0相交，所有其他值的评分0。
    
*   exp: 指数函数是先剧烈衰减然后变缓。
    
*   gauss（最常用） : 高斯函数则是`钟形`的，他的衰减速率是先缓慢，然后变快，最后又放缓。
    

script\_score：通过自定义脚本计算评分（灵活性最高），但脚本复杂时影响性能

**函数结果合并方式（****score\_mode****）：**

*   multiply（默认）：函数结果相乘
    
*   sum：函数结果相加
    
*   avg：函数结果求平均
    
*   first：使用第一个匹配的函数结果
    
*   max/min：取最大值 / 最小值
    

**最终评分与原始评分的合并方式(boost\_mode)**

*   multiply（默认）：函数结果相乘
    
*   sum：函数结果相加
    
*   avg：函数结果求平均
    
*   first：使用第一个匹配的函数结果
    
*   max/min：取最大值 / 最小值
    

```json

GET /my_index/_search
{
  "query": {
    "function_score": {
      "query": {
        "term": {
          "age": 18
        }
      },
      "functions": [
        {
          "gauss": {
            "time": {
              "origin": "now",
              "scale": "30d",
              "offset": "8d",
              "decay": 0.5
            }
          },
          "weight": 1
        }
      ],
      "score_mode": "min",
      "boost_mode": "min",
      "max_boost": 10,
      "min_score": 0
    }
  }
}
```

#### 2.1.5 查询排序

数值 / 日期 / 字符串字段排序，适用字段类型：不分词的字符串，如keyword，date，integer，double等

```json

GET /my_index/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    { "age": { "order": "desc" } },     
    { "time": { "order": "asc" } }  
  ]
}
```
```json
GET /my_index/_search
{
  "query": {
    "function_score": {
      "query": {
        "term": {
          "age": 18
        }
      },
      "functions": [
        {
          "gauss": {
            "time": {
              "origin": "now",
              "scale": "30d",
              "offset": "8d",
              "decay": 0.5
            }
          },
          "weight": 1
        }
      ],
      "score_mode": "min",
      "boost_mode": "min",
      "max_boost": 10,
      "min_score": 0
    }
  },
  "sort": { "_score": { "order": "asc" } }
}
```
```json

GET /my_index/_search
{
  "query": {
    "script": {
      "script": {
        "lang": "painless",
        "source": "doc['age'].value > params.min_age && doc['age'].value < params.max_age",
        "params": {
          "min_age": 15,
          "max_age": 29
        }
      }
    }
  },
  "sort": [
    { "age": { "order": "desc" } }  
  ]
}
```
```json
GET /my_index/_search
{
  "query": {
    "script": {
      "script": {
        "lang": "painless",
        "source": "doc['age'].value > params.min_age && doc['age'].value < params.max_age",
        "params": {
          "min_age": 15,
          "max_age": 35
        }
      }
    }
  },
  "sort": [
    {
      "_script": {
        "type": "number",
        "script": {
          "lang": "painless",
          "source": "Math.abs(doc['age'].value - 25)"  
        },
        "order": "asc"
      }
    }
  ]
}
```

#### 2.1.6 分页查询

##### 2.1.6.1基础分页：

**优点**：简单直观，适合小数据量（前几千条）。

**缺点**：深度分页（如 from=10000）性能极差，因为每个分片都要扫描并排序全量数据。

**限制**：默认 from + size 不能超过 index.max\_result\_window（默认 10000）。

```json
GET /my_index/_search
{
  "query": {
    "match_all": {}
  },
  "from": 1,  
  "size": 10  
}
```

##### 2.1.6.2滚动分页：

初始化滚动会话，返回结果包含 `_scroll_id`，用于后续请求

```json
GET /my_index/_search?scroll=1m  
{
  "query": {
    "match_all": {}
  },
  "size": 3  
}

```
```json
POST /_search/scroll
{
  "scroll": "1m",         
  "scroll_id": "DXF1ZXJ5QW5kRmV0Y2gB..."  
}

```

*   **优点：**适合大数据量导出（如百万级数据），性能稳定。
    
*   **缺点：**
    
    *   结果是查询时刻的快照，不反映实时变化。
        
    *   需要维护会话，占用资源。
        
    *   不支持跳页，只能顺序获取**。**
        

##### 2.1.6.3 Search After深度分页

用第一次请求的sort值作为后续请求的入参，请求必须包含sort字段

```json
GET /my_index/_search
{
  "query": {
    "match_all": {}
  },
  "size": 10,
  "sort": [
    { "time": { "order": "desc" } },  
    { "_id": { "order": "desc" } }          
  ]
}
```
```json
GET /my_index/_search
{
  "query": {
    "match_all": {}
  },
  "size": 2,
  "sort": [
    { "time": { "order": "desc" } },
    { "_id": { "order": "desc" } }
  ],
  "search_after": [1718366400000,
          "fcyOg5cB4TnJJxlruknG"]  
}
```

*   **优点**：
    
    *   性能最优，无需扫描全量数据。
        
    *   支持实时数据变化。
        
    *   适合海量数据分页（如千万级）。
        
*   **缺点**：
    
    *   不支持后退（只能向前翻页）。
        
    *   实现较复杂，需客户端维护 `search_after` 值
        

##### 2.1.6.4 对比

|  **分页方式**  |  **深度分页性能**  |  **实时性**  |  **资源消耗**  |  **适用场景**  |
| --- | --- | --- | --- | --- |
|  From/Size  |  差  |  是  |  高  |  小数据量浅分页  |
|  Scroll API  |  好  |  否  |  中等  |  大数据量批量导出  |
|  Search After  |  好  |  是  |  低  |  大数据量实时分页  |

#### 2.1.7 更新文档

全量替换更新：

*   会完全替换原文档，未指定的字段将被删除
    
*   操作简单，但可能导致数据丢失
    
*   版本号会递增
    

```json
PUT /my_index/_doc/文档ID
{
  "age": 30,
  "level": "大乘"
}
```

部分更新：

*   只更新指定字段，不影响其他字段
    
*   支持脚本更新（Scripted Updates）
    
*   内部使用乐观锁保证并发安全
    

```json
POST /my_index/_update/文档ID
{
  "age": 30,
  "level": "大乘"
}
```

脚本更新：

*   适合执行复杂操作（如计数器、条件更新）
    
*   支持 Groovy/Painless 脚本语言（推荐使用 Painless）
    
*   原子性操作，避免并发冲突
    

```json
POST /my_index/_update/文档ID
{
  "script": {
    "source": "if (ctx._source.age > 0) { ctx._source.age++ }",
    "lang": "painless"
  }
}
```

Upsert（存在更新，不存在创建）

*   如果文档存在，执行更新
    
*   如果文档不存在，使用内容创建新文档
    
*   避免先查询再更新的额外开销
    

```json
POST /my_index/_update/文档ID
{
  "doc": { "age": 50 },
  "upsert": {
    "name": "小明",
    "age": 30,
    "tall": 180
  }
}
```

批量更新（\_bulk API）

*   高效处理大量更新请求
    
*   减少网络开销，提升吞吐量
    
*   原子性保证：要么全部成功，要么全部失败
    

```json
POST /_bulk
{ "update": { "_index": "my_index", "_id": "1" } }
{ "doc": { "age": 200 } }
{ "update": { "_index": "my_index", "_id": "2" } }
{ "doc": { "age": 50 } }
```

#### 2.1.8 删除文档

```json
DELETE /my_index/_doc/文档ID
```

注意事项：可使用 version 参数确保删除操作的原子性

```json
DELETE /my_index/_doc/文档ID?version=5
```
```json
POST /my_index/_delete_by_query
{
  "query": {
    "age": 500
  }
}
```

注意：可能会产生大量段碎片，建议在低峰期执行并随后进行段合并（\_forcemerge）

*   max\_num\_segments：指定合并后每个分片的最大段数量（默认值为 1）。
    
*   only\_expunge\_deletes：仅删除标记为已删除的文档，不进行段合并（默认值为false）。
    
*   flush：合并后是否执行刷新操作（默认值为true）
    

```json
POST /my_index/_forcemerge?参数=值
```
```json
POST /_bulk
{ "delete": { "_index": "my_index", "_id": "1" } }
{ "delete": { "_index": "my_index", "_id": "2" } }
{ "delete": { "_index": "my_index", "_id": "3" } }
```

# 二、Spring Boot 集成 Elasticsearch 

### 1. 添加依赖

在pom.xml文件中添加 Spring Data Elasticsearch 依赖：

```json
<dependency>

    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>

```
```json
            <dependency>
                <groupId>org.elasticsearch.client</groupId>
                <artifactId>elasticsearch-rest-high-level-client</artifactId>
                <version>${elasticsearch.version}</version>
            </dependency>
            <dependency>
                <groupId>org.elasticsearch.client</groupId>
                <artifactId>elasticsearch-rest-client</artifactId>
                <version>${elasticsearch.version}</version>
            </dependency>
            <dependency>
                <groupId>org.elasticsearch</groupId>
                <artifactId>elasticsearch</artifactId>
                <version>${elasticsearch.version}</version>
            </dependency>

            <dependency>
                <groupId>org.dromara.easy-es</groupId>
                <artifactId>easy-es-boot-starter</artifactId>
                <version>${easy-elasticsearch.version}</version>
            </dependency>

```

### 2. 配置 Elasticsearch 连接

```java
easy-es:
  address: 123.57.17.53:9200 # es的连接地址,必须含端口 若为集群,则可以用逗号隔开 例如:127.0.0.1:9200,127.0.0.2:9200
 # username: elastic #如果无账号密码则可不配置此行
  #password: Es#Tyt@20210524 #如果无账号密码则可不配置此行
  keep-alive-millis: 300000 # 心跳策略时间 单位:ms
  connect-timeout: 10000 # 连接超时时间 单位:ms
  socket-timeout: 20000 # 通信超时时间 单位:ms
  connection-request-timeout: 5000 # 连接请求超时时间 单位:ms
  max-conn-total: 1024 # 最大连接数 单位:个
  max-conn-per-route: 500 # 最大连接路由数 单位:个
  global-config:
    db-config:
      index-prefix: dev_
    map-underscore-to-camel-case: true


```

### 3. 代码使用

#### 3.1 使用spring集成es

使用RestHighLevelClient进行操作

##### 3.1.1 查询组装

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/Q35O851GbJR4Jl9V/img/88d091fe-425e-4f94-b877-ba3d3cae05a9.png)

```java

    /**
     * 组装查询条件
     *
     * @param searchBean
     * @return
     */
    private BoolQueryBuilder buildBaseQuery(BaseTransportSearchDTO searchBean) {
        BoolQueryBuilder query = QueryBuilders.boolQuery();
        query.must(QueryBuilders.termQuery("status", 1)).must(QueryBuilders.termQuery("displayType", 1)).must(QueryBuilders.termQuery("isDisplay", 1)).must(QueryBuilders.termQuery("isShow", 1));

        BoolQueryBuilder startBoolQuery = QueryBuilders.boolQuery();
        BoolQueryBuilder startAddressQuery = QueryBuilders.boolQuery();
        if (StringUtils.isNotBlank(searchBean.getStartProvinc())) {
            startAddressQuery.filter(QueryBuilders.termQuery("startProvinc", searchBean.getStartProvinc()));
        }
        if (StringUtils.isNotBlank(searchBean.getStartCity())) {
            startAddressQuery.filter(QueryBuilders.termQuery("startCity", searchBean.getStartCity()));
        }
        if (StringUtils.isNotBlank(searchBean.getStartArea())) {
            startAddressQuery.filter(QueryBuilders.termQuery("startArea", searchBean.getStartArea()));
        }
        BoolQueryBuilder startCoordQuery = QueryBuilders.boolQuery();
        if (searchBean.getStartCoordX() != null && searchBean.getStartCoordY() != null) {
            startCoordQuery.must(QueryBuilders.rangeQuery("startCoordX").from(searchBean.getStartCoordX() - searchBean.getStartRange()).to(searchBean.getStartCoordX() + searchBean.getStartRange()));
            startCoordQuery.must(QueryBuilders.rangeQuery("startCoordY").from(searchBean.getStartCoordY() - searchBean.getStartRange()).to(searchBean.getStartCoordY() + searchBean.getStartRange()));
        }
        // 检查 addressQuery 是否有子句（即是否所有字段都被填充了）
        if (startAddressQuery.hasClauses()) {
            startBoolQuery.should(startAddressQuery);
        }
        // 检查 coordQuery 是否有子句（即 destCoordX 和 destCoordY 是否都被填充了）
        if (startCoordQuery.hasClauses()) {
            startBoolQuery.should(startCoordQuery);
        }
        if (startBoolQuery.hasClauses()) {
            query.must(startBoolQuery);
        }

        BoolQueryBuilder destBoolQuery = QueryBuilders.boolQuery();
        BoolQueryBuilder destAddressQuery = QueryBuilders.boolQuery();
        if (StringUtils.isNotBlank(searchBean.getDestProvinc())) {
            destAddressQuery.filter(QueryBuilders.termQuery("destProvinc", searchBean.getDestProvinc()));
        }
        if (StringUtils.isNotBlank(searchBean.getDestCity())) {
            destAddressQuery.filter(QueryBuilders.termQuery("destCity", searchBean.getDestCity()));
        }
        if (StringUtils.isNotBlank(searchBean.getDestArea())) {
            destAddressQuery.filter(QueryBuilders.termQuery("destArea", searchBean.getDestArea()));
        }
        BoolQueryBuilder destCoordQuery = QueryBuilders.boolQuery();
        if (searchBean.getDestCoordX() != null && searchBean.getDestCoordY() != null) {
            destCoordQuery.must(QueryBuilders.rangeQuery("destCoordX").from(searchBean.getDestCoordX() - searchBean.getDestRange()).to(searchBean.getDestCoordX() + searchBean.getDestRange()));
            destCoordQuery.must(QueryBuilders.rangeQuery("destCoordY").from(searchBean.getDestCoordY() - searchBean.getDestRange()).to(searchBean.getDestCoordY() + searchBean.getDestRange()));
        }

        // 检查 addressQuery 是否有子句（即是否所有字段都被填充了）
        if (destAddressQuery.hasClauses()) {
            destBoolQuery.should(destAddressQuery);
        }
        // 检查 coordQuery 是否有子句（即 destCoordX 和 destCoordY 是否都被填充了）
        if (destCoordQuery.hasClauses()) {
            destBoolQuery.should(destCoordQuery);
        }
        if (destBoolQuery.hasClauses()) {
            query.must(destBoolQuery);
        }

        if (searchBean.getQueryType() != null) {
            if (Objects.equals(searchBean.getQueryType(), 2)) {
                if (searchBean.getQuerySign() != null) {
                    query.must(QueryBuilders.rangeQuery("id").lt(searchBean.getQuerySign()));
                }
                if (searchBean.getMinTsId() != null) {
                    query.must(QueryBuilders.rangeQuery("id").lt(searchBean.getMinTsId()));
                }
            } else {
                if (searchBean.getQuerySign() != null && searchBean.getQuerySign() > 0) {
                    query.must(QueryBuilders.rangeQuery("id").gt(searchBean.getQuerySign()));
                }
            }
        }
        if (searchBean.getStartWeight() != null) {
            query.must(QueryBuilders.rangeQuery("weight").gt(0));
            query.must(QueryBuilders.rangeQuery("referWeight").gte(searchBean.getStartWeight()));
        }

        if (searchBean.getEndWeight() != null) {
            query.must(QueryBuilders.rangeQuery("weight").gt(0));
            query.must(QueryBuilders.rangeQuery("referWeight").lte(searchBean.getEndWeight()));
        }

        if (searchBean.getMinLength() != null) {
            query.must(QueryBuilders.rangeQuery("length").gt(0));
            query.must(QueryBuilders.rangeQuery("referLength").gte(searchBean.getMinLength()));
        }

        if (searchBean.getMaxLength() != null) {
            query.must(QueryBuilders.rangeQuery("length").gt(0));
            query.must(QueryBuilders.rangeQuery("referLength").lte(searchBean.getMaxLength()));
        }

        if (searchBean.getMinWidth() != null) {
            query.must(QueryBuilders.rangeQuery("wide").gt(0));
            query.must(QueryBuilders.rangeQuery("referWidth").gte(searchBean.getMinWidth()));
        }

        if (searchBean.getMaxWidth() != null) {
            query.must(QueryBuilders.rangeQuery("wide").gt(0));
            query.must(QueryBuilders.rangeQuery("referWidth").lte(searchBean.getMaxWidth()));
        }

        if (searchBean.getMinHeight() != null) {
            query.must(QueryBuilders.rangeQuery("high").gt(0));
            query.must(QueryBuilders.rangeQuery("referHeight").gte(searchBean.getMinHeight()));
        }

        if (searchBean.getMaxHeight() != null) {
            query.must(QueryBuilders.rangeQuery("high").gt(0));
            query.must(QueryBuilders.rangeQuery("referHeight").lte(searchBean.getMaxHeight()));
        }

        if (searchBean.getMinPrice() != null) {
            query.must(QueryBuilders.rangeQuery("price").gte(searchBean.getMinPrice()));
        }

        if (searchBean.getMaxPrice() != null) {
            query.must(QueryBuilders.rangeQuery("price").lte(searchBean.getMaxPrice()));
        }

        if (searchBean.getPublishType() != null) {
            query.must(QueryBuilders.termQuery("publishType", searchBean.getPublishType()));
        }

        if (searchBean.getRefundFlag() != null) {
            query.must(QueryBuilders.termQuery("refundFlag", searchBean.getRefundFlag()));
        }
        if (searchBean.getExcellentGoods() != null) {
            query.must(QueryBuilders.termQuery("excellentGoods", searchBean.getExcellentGoods()));
        }

        Date startLoadingTime = searchBean.getStartLoadingTime() == null ? null : new Date(searchBean.getStartLoadingTime());
        Date endLoadingTime = searchBean.getEndLoadingTime() == null ? null : new Date(searchBean.getEndLoadingTime());
        Boolean queryNullLoadingTime = searchBean.getQueryNullLoadingTime();
        if (startLoadingTime == null && endLoadingTime == null) {
            if (Boolean.TRUE.equals(queryNullLoadingTime)) {
                query.must(QueryBuilders.boolQuery().mustNot(QueryBuilders.existsQuery("loadingTime")));
            }
        } else {
            if (startLoadingTime != null) {
                BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
                boolQuery.should(QueryBuilders.rangeQuery("loadingTime").gte(startLoadingTime));

                if (Boolean.TRUE.equals(queryNullLoadingTime)) {
                    boolQuery.should(QueryBuilders.boolQuery().mustNot(QueryBuilders.existsQuery("loadingTime")));
                }
                query.must(boolQuery);
            }

            if (endLoadingTime != null) {
                BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
                boolQuery.should(QueryBuilders.rangeQuery("loadingTime").lte(endLoadingTime));

                if (Boolean.TRUE.equals(queryNullLoadingTime)) {
                    boolQuery.should(QueryBuilders.boolQuery().mustNot(QueryBuilders.existsQuery("loadingTime")));
                }
                query.must(boolQuery);
            }
        }

        if (searchBean.getPriceFlag() != null) {
            if (searchBean.getPriceFlag() == 0) {
                //无价
                BoolQueryBuilder priceBoolQuery = QueryBuilders.boolQuery();
                priceBoolQuery.should(QueryBuilders.boolQuery().mustNot(QueryBuilders.existsQuery("price")));
                priceBoolQuery.should(QueryBuilders.rangeQuery("price").lte(0));
                query.must(priceBoolQuery);
            } else {
                //有价
                query.must(QueryBuilders.rangeQuery("price").gt(0));
            }
        }
        String goodsNames = searchBean.getGoodsName();
        if (StringUtils.isNotBlank(goodsNames)) {
            List<String> goodsNameList = Arrays.stream(goodsNames.split(COMMA)).filter(StringUtils::isNotBlank).toList();
            //商品名称 or 查询
            BoolQueryBuilder goodsNameBool = QueryBuilders.boolQuery();

            for (String oneName : goodsNameList) {
                if (StringUtils.isNotBlank(oneName)) {
                    goodsNameBool.should(QueryBuilders.matchPhraseQuery("taskContent", oneName));
                    //添加同义词
                    goodsNameBool.should(QueryBuilders.matchPhraseQuery("taskContent.synonym", oneName));
                }
            }
            query.must(goodsNameBool);
        }

        if (CollectionUtils.isNotEmpty(searchBean.getExcludeSrcMsgIds())) {
            query.mustNot(QueryBuilders.termsQuery("srcMsgId", searchBean.getExcludeSrcMsgIds()));
        }

        if (CollectionUtils.isNotEmpty(searchBean.getUseSrcMsgIds())) {
            query.must(QueryBuilders.termsQuery("srcMsgId", searchBean.getUseSrcMsgIds()));
        }

        if (CollectionUtils.isNotEmpty(searchBean.getExcludeUserIds())) {
            query.mustNot(QueryBuilders.termsQuery("userId", searchBean.getExcludeUserIds()));
        }

        if (Objects.nonNull(searchBean.getUseCarType())) {
            query.must(QueryBuilders.termQuery("useCarType", searchBean.getUseCarType()));
        }

        // 秒抢货源
        if (searchBean.getSeckillGoods() != null) {
            query.must(QueryBuilders.termQuery("seckillGoods", searchBean.getSeckillGoods()));
        }

        return query;
    }


```

##### 3.1.2 评分查询

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/Q35O851GbJR4Jl9V/img/f99c7fb0-2a2f-49bf-b02e-225f5897bf60.png)

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/Q35O851GbJR4Jl9V/img/300667f2-3b90-4c44-9f18-2ceb6563e54d.png)

```java

    /**
     * 组装评分函数
     *
     * @param intelligenceSortDTO
     * @return
     */
    private FunctionScoreQueryBuilder.FilterFunctionBuilder[] buildScoreFunction(IntelligenceSortDTO intelligenceSortDTO) {

        List<IntelligenceSortRuleDO> ruleList = intelligenceSortRuleMapper.selectAll();
        Map<Integer, List<IntelligenceSortRuleDO>> collect = ruleList.stream().collect(Collectors.groupingBy(IntelligenceSortRuleDO::getRuleType));
        IntelligenceSortRuleDO addressRule = collect.get(IntelligentSortRuleTypeEnum.ADDRESS.getCode()).get(0);
        IntelligenceSortRuleDO ageingsRule = collect.get(IntelligentSortRuleTypeEnum.AGEING.getCode()).get(0);
        IntelligenceSortRuleDO infoRule = collect.get(IntelligentSortRuleTypeEnum.INFO.getCode()).get(0);
        IntelligenceSortRuleDO modelRule = collect.get(IntelligentSortRuleTypeEnum.MODEL.getCode()).get(0);
        IntelligenceSortRuleDO weightRule = collect.get(IntelligentSortRuleTypeEnum.WEIGHT.getCode()).get(0);
        IntelligenceSortRuleDO refreshRule = collect.get(IntelligentSortRuleTypeEnum.REFRESH.getCode()).get(0);

        // 地址评分
        Script addressScript = buildAddressScript(intelligenceSortDTO, addressRule);
        // 时效性评分
        Script timeScript = buildTimeScript(intelligenceSortDTO, ageingsRule);
        // 货源信息完整度
        Script integrityScript = buildIntegrityScript(infoRule);
        // 质量分评分
        Script modelScript = buildModelScript(modelRule);
        // 车货吨位匹配
        Script weightScript = buildWeightScript(intelligenceSortDTO, weightRule);
        // 刷新规则评分
        Script refreshScript = buildRefreshScript(intelligenceSortDTO, refreshRule);

        FunctionScoreQueryBuilder.FilterFunctionBuilder addressFilterFunctionBuilder = new FunctionScoreQueryBuilder.FilterFunctionBuilder(new ScriptScoreFunctionBuilder(addressScript).setWeight(addressRule.getWeight().floatValue()));
        FunctionScoreQueryBuilder.FilterFunctionBuilder timeFilterFunctionBuilder = new FunctionScoreQueryBuilder.FilterFunctionBuilder(new ScriptScoreFunctionBuilder(timeScript).setWeight(ageingsRule.getWeight().floatValue()));
        FunctionScoreQueryBuilder.FilterFunctionBuilder integrityFilterFunctionBuilder = new FunctionScoreQueryBuilder.FilterFunctionBuilder(new ScriptScoreFunctionBuilder(integrityScript).setWeight(infoRule.getWeight().floatValue()));
        FunctionScoreQueryBuilder.FilterFunctionBuilder modelFilterFunctionBuilder = new FunctionScoreQueryBuilder.FilterFunctionBuilder(new ScriptScoreFunctionBuilder(modelScript).setWeight(modelRule.getWeight().floatValue()));
        FunctionScoreQueryBuilder.FilterFunctionBuilder refreshFilterFunctionBuilder = new FunctionScoreQueryBuilder.FilterFunctionBuilder(new ScriptScoreFunctionBuilder(refreshScript).setWeight(refreshRule.getWeight().floatValue()));
        if (weightScript != null) {
            FunctionScoreQueryBuilder.FilterFunctionBuilder weightFilterFunctionBuilder = new FunctionScoreQueryBuilder.FilterFunctionBuilder(new ScriptScoreFunctionBuilder(weightScript).setWeight(weightRule.getWeight().floatValue()));
            return new FunctionScoreQueryBuilder.FilterFunctionBuilder[]{
                    addressFilterFunctionBuilder,
                    timeFilterFunctionBuilder,
                    integrityFilterFunctionBuilder,
                    modelFilterFunctionBuilder,
                    refreshFilterFunctionBuilder,
                    weightFilterFunctionBuilder};
        }
        return new FunctionScoreQueryBuilder.FilterFunctionBuilder[]{
                addressFilterFunctionBuilder,
                timeFilterFunctionBuilder,
                integrityFilterFunctionBuilder,
                modelFilterFunctionBuilder,
                refreshFilterFunctionBuilder};
    }


```

#### 3.2 使用easy-es

RestHighLevelClient的封装包，与mybatis-plus操作类似，可直接操作

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/Q35O851GbJR4Jl9V/img/7d5675db-9f1d-4e61-9731-6a1ecae31b90.png)

```java
    private final GoodsMatchEsMapper goodsMatchEsMapper;

    private final RestHighLevelClient restHighLevelClient;


    @Override
    public List<GoodsMatchEsDO> searchGoodsMatch(GoodsMatchRpcDto goodsMatchRpcDto) {

        LambdaEsQueryWrapper<GoodsMatchEsDO> wrapper = new LambdaEsQueryWrapper<>();
        wrapper.filter().eq(GoodsMatchEsDO::getDisplay, YesOrNoEnum.YES.getId());
        wrapper.match(GoodsMatchEsDO::getShowName, goodsMatchRpcDto.getKeywords());
        wrapper.from((goodsMatchRpcDto.getPageNum() - 1) * goodsMatchRpcDto.getPageSize());
        wrapper.size(goodsMatchRpcDto.getPageSize());


        return goodsMatchEsMapper.selectList(wrapper);

    }
```

# 三、使用 Elasticsearch 要注意的问题

### 1. 索引设计

*   合理规划索引的映射，避免频繁修改映射结构，因为修改映射可能导致索引重建，影响性能和可用性。
    
*   根据数据量和查询需求，考虑使用别名来管理索引，方便索引的滚动和切换。
    

### 2. 数据写入

*   批量写入数据可以提高写入性能，减少与 Elasticsearch 的交互次数。可以使用Bulk API进行批量操作。
    
*   控制写入速度，避免因写入过快导致集群负载过高，影响查询性能和集群稳定性。
    

### 3. 查询性能

*   避免使用通配符查询（如\*开头的查询），因为这类查询会扫描整个索引，性能较差。尽量使用前缀查询或精确查询。
    
*   对经常用于查询的字段进行适当的分词和索引设置，以提高查询效率。
    

### 4. 集群管理

*   定期监控集群的健康状态、磁盘使用情况、CPU 和内存使用率等指标，及时发现和解决潜在问题。
    
*   做好数据备份和恢复策略，防止数据丢失。可以使用 Elasticsearch 的快照和恢复功能进行数据备份。