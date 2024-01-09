---
title: ElasticSearch
categories: 数据库
tags: [数据库]
---

## 简介

- 也算是一个数据库，可以从海量数据中检索出想要的数据
- 所有的操作都封装了对应的RestAPI，方便使用

## 定义

| 属性     | 对应   |
| -------- | ------ |
| Index    | 数据库 |
| Type     | 表     |
| Document | 文档   |
| Field    | 字段   |

## 概念

- 全文搜索

  - 相关性
    - 评价查询与结果的相关程度，并进行排序
  - 分析
    - 将文本转换为token，目的是为了创建倒排索引，查询倒排索引

- 倒排索引

  - 根据属性值找记录，记录属性值在记录中出现的频率
  - 先将记录进行分词，记录分词对应的记录
  - 搜索时也会将搜索词进行分词，然后去找对应的记录
  - 找到的记录进行相关性评分，评分高的返回

- 单次搜索

  - 检查字段类型
  - 分析查询字符串
  - 查找匹配文档
  - 为每个文档评分

- 多次搜索

  - `"minimum_should_match" : "40%"`  设置多词之间匹配度
  - `"operator" : "and / or"`  设置多次之间的关系

- 组合搜索

  ```json
  "query": {
      "bool": {
          "must": {
              "match": {
                  "": ""
              }
          },
          "must_not": {
              "match": {
                  "": ""
              }
          },
          "should": {
              "match": {
                  "": ""
              }
          }
      }
  }
  ```

  - 计算每个文档的相关度评分_score，将所有匹配的must和should语句的分数求和，除于must和should语句的条数
  - should中的内容不是必须匹配的，但是语句中没有must的话，那么至少会匹配一个

- 权重

  - should中通过boost来设置权重

- 短语匹配

  - 词和顺序都要匹配
  - slop：跳过n个词进行匹配

## 使用

- 路径 `http://localhost:9200/`

### 基本信息

| 路径          | 请求格式 | 作用           |      |
| ------------- | -------- | -------------- | ---- |
| /_cat/nodes   | Get      | 查看所有节点   |      |
| /_cat/health  | Get      | 查看es健康状态 |      |
| /_cat/master  | Get      | 查看主节点     |      |
| /_cat/indices | Get      | 查看所有索引   |      |

### CRUD

- 注意
  - ES新版本去除了类型type

#### 插入数据

不携带id就新增，携带就是修改

```json
POST esIp:9200/索引名/类型/id
{
    "settings": {
        "index": {
         	"number_of_shards": "2", #分片数
            "number_of_replicas" : "0" #副本数
        }
    }
}
```

#### 修改数据

- 必须携带id
- 通过覆盖实现的更新
- 每次覆盖更新版本会加一，乐观锁机制
  - 发起请求时携带 `?if_seq_np= &if_primary_term= `

```json
PUT url:端口号/索引名/类型/id
{
    "":  
}
```


#### 局部更新

```json
POST/PUT url:端口号/索引名/类型/id/_update
{
    "doc":{
        "": ""
    }  
}
```

- 路径使用_update就必须携带`"doc":{}` 体
- 会对比待更新的数据，一样则不更新，版本号不会自增

#### 删除数据

```json
#删除对应的数据
DELETE url:端口号/索引名/类型/id
#删除suo'yin
DELETE url:端口号/索引名
```

#### 查询数据

```json
GET url:端口号/索引名/类型/id
```

```json
GET url:端口号/索引名/类型/_search
```

- 搜索全部数据，默认返回10条

```json
GET url:端口号/索引名/类型/_search?q=age:20
```

- 添加关键字

## DSL搜索

查询数据时可以携带Json体，请求类型为POST

- 全文搜索

  ```json
  POST url:端口号/索引名/类型/_search
  {
      "query": {
          "match": {
              "name": "张三 里斯"
          }
      }
  }
  ```

- 高亮显示

  ```json
  "highlight": {
      "fields": {
          "name": {}
      }
  }
  ```

- 聚合搜索

  - 桶聚合,类似sql中的分组

  - 可以指定size

    ```json
    POST url:端口号/索引名/类型/_search
    {
        "aggs": {
            "all_interests": {
                 "terms": {
                    "fields": "age"
        		}
            }
        }
    }
    ```
  
  - 度量聚合

    - min、avg、max

- 分页搜索

  - 类似sql中的分页
  - size: 结果数
  - from: 跳过数

- 文档

- 美化

  - 在查询语句后添加`?pretty` 可以美化返回的json

- 指定响应字段

  - GET请求，指定返回字段，会返回元数据

    ```json
    GET url:端口号/索引名/类型/id/?_source=返回的字段
    ```

  - 返回数据，不返回元数据

    ```json
    GET url:端口号/索引名/类型/id/_source //返回全部数据
    ```

    ```json
    GET url:端口号/索引名/类型/id/_source?_source=返回的字段
    ```

- 判断文档是否存在

  ```json
  HEAD url:端口号/索引名/类型/id
  ```

## 批量操作

### 批量查询

```json
POST url:端口号/索引名/类型/_mget
{
    "ids": ["",""]
}
```

### 批量增删改

- 格式
  - `{"操作":{元数据}}`
- 每条操作记录都是独立的，失败的不会影响到其他操作的执行

```json
POST url:端口号/_bulk
{
    //插入
    {"create": {"_index":"索引名", "_type":"类型","_id":id}}\n
	{"id":1,"name":"name1","age":18,"sex":"男"}\n
	//删除
	{"delete":{"_index":"索引名", "_type":"类型","_id":id}}
}
```

### 映射

- String 新版不支持了，需要有全文搜索用text，不需要就用keyword

- 查看映射

  ```json
  GET url:端口号/索引名/_mapping
  ```

## 结构化查询

- term

  ```json
  POST url:端口号/索引名/类型/_search
  {
      "query": {
       	"term": {
              "age": 20
          }   
      }
  }
  ```

  - 用于精确匹配

- terms

  ```json
  POST url:端口号/索引名/类型/_search
  {
      "query": {
       	"terms": {
              "tag": ["","",""]
          }   
      }
  }
  ```

  - 类似sql中的in

- range

  ```json
  POST url:端口号/索引名/类型/_search
  {
      "query": {
       	"range": {
              "age": {
                  "gte": 20,
                  "lt": 30
              }
          }   
      }
  }
  ```

  gt、gte、lt、lte

- exists

  ```json
  POST url:端口号/索引名/类型/_search
  {
      "query": {
       	"exists": {
              "field": "title"
          }   
      }
  }
  ```

- match

  ```json
  POST url:端口号/索引名/类型/_search
  {
      "query": {
       	"match": {
              "tweet": "About Search"
          }   
      }
  }
  ```

- bool

  ```json
  POST url:端口号/索引名/类型/_search
  {
      "query": {
       	"bool": {
              "must": {"term": {}}      //相当于and
              "must_not": {"term": {}}  //相当于not
              "should": {"term": {}}    //相当于or
          }   
      }
  }
  ```

- 过滤查询

  ```json
  POST url:端口号/索引名/类型/_search
  {
      "query": {
       	"bool": {
              "filter": {
              	"term": {
                      "age": 20
                  }
              }
          }   
      }
  }
  ```

  - 查询和过滤的对比
    - 过滤会询问每个文档的字段值是否包含特定值
    - 查询户询问每个文档的字段值与特定值的匹配程度
    - 过滤会缓存结果，高效
    - 查询语句比过滤语句耗时

## 分词

- 内置分词

  ```json
  POST url:端口号/_analyze
  {
      "analyzer": "分词器",
      "field": "字段",
      "text": ""
  }
  ```

  - 内置分词器有
    - Standard，按单词划分，会转化为小写
    - Simple，按照非单词划分，转小写
    - Whitespace，按空格切分
    - Stop，去除语气词后划分
    - Keyword，不做分词处理

- 中文分词
  - IK分词器



## 分布式文档

计算存储节点位置：`shard = hash(routing) % number_of_primary_shards`

- 文档搜索
  - 查询
    - 客户端发送搜索请求，节点收到后，创建一个from+size大小的空优先队列，然后将请求转发到其他分片
    - 每个分片在本地执行这个请求，将结果存入优先队列
    - 每个分片返回id和队列中的排序值给一开始接收请求的分片，这个分片合并这些值到自己的优先队列中后返回
  - 取回
    - 协调节点辨别出哪个Document需要取回，并且向相关分片发出GET请求
    - 每个分片加载document并且根据需要丰富它们，然后再将document返回协调节点
    - 一旦所有的document都被取回，协调节点会将结果返回给客户端

## 调优

## 集群

### 节点

- master节点
  - 配置文件中node.master设置为true，则有资格被选为master节点
  - 创建/删除索引，管理非master
- data节点
  - 配置文件中node.data设置为true，就有资格
  - CRUD
- 客户端节点
  - node.master和node.data都为false
  - 响应客户的请求，转化请求
- 部落节点
  - 配置tribe.*，则是一个部落节点
  - 可以连接多个集群，在所有集群上进行搜索等操作

### 分片和副本

- 一个分片是一个最小级别的工作单元，他只是保存了索引中所有数据的一部分
- 我们需要知道分片就是一个Lucene实例，并且它本身就是一个完整的搜索引擎，应用程序不会和它直接通信
- 分片可以是主分片或者是复制分片
- 索引中的每个文档属于一个单独的主分片，所以主分片的数量决定了所应最多能存储多少数据
- 复制分片只是主分片的一个副本，它可以防止硬件故障导致的数据丢失，同时可以提供读请求，比如搜索或者从别的分片取回文档
- 当索引创建完成的时候，主分片的数量就固定了，但是复制分片的数量可以随时调整

### 脑裂

- master发生宕机，然后集群重新选举master，宕机的master恢复后，集群中出现了俩个master，会分成俩个集群
- 设置选举master数为：（N/2）+ 1 可解决
