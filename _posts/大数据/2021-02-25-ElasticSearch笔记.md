---
layout:     post
title:      ElasticSearch笔记
subtitle:   ElasticSearch笔记
date:       2021-02-25
author:     张三
header-img: img/ElasticSearch.jpg
catalog: true
mathjax: true
tags:
    - ElasticSearch

---

> [【狂神说Java】ElasticSearch7.6.x最新完整教程通俗易懂](https://www.bilibili.com/video/BV17a4y1x7zq?p=4)

# 简介

ElasticSearch 是一个开源高扩展性的分布式全文检索引擎，可以近乎实时的存储和检索数据。其本身扩展性良好，能够处理PB级别的数据。ElasticSearch使用Java开发，并使用Lucene作为核心来实现索引和搜索功能，通过简单的 RESTful API 来隐藏Lucene的复杂性。

ElasticSearch 可用于全文搜索、结构化搜索、分析以及三者的混合使用。

Lucene 是一个成熟的免费开源工具，是当前最受欢迎的免费Java信息检索程序库。

**Solr** 是Apache下一个顶级的开源项目，使用Java开发，是基于Lucene的全文搜索服务器，提供了比Lucene更丰富的查询语言，同时实现了可配置可扩展，对索引和搜索性能进行了优化。对外提供类似于Web-service的API接口，可通过HTTP请求向搜索引擎服务器提交一定格式的文件生成索引，也可以通过提交查询请求获得返回结果。

### ElasticSearch和Solr比较

- 性能比较
    + 对已有数据进行搜索时，Solr更快
    + 建立索引时，Solr会产生IO阻塞，查询性能较差，ElasticSearch具有明显优势
    + 随着数据量增加，Solr效率变低，ElasticSearch没有明显变化
- ElasticSearch解压即可用，Solr需要安装
- Solr使用Zookeeper进行分布式管理，ElasticSearch自带分布式功能
- Solr支持更多格式的数据，ElasticSearch只支持json
- Solr功能更多，ElasticSearch更专注于核心功能，高级功能通过第三方插件提供，例如通过kibana提供图形化界面
- Solr比较成熟，ElasticSearch更新较快

# ElasticSearch安装

环境要求：jdk1.8

Windows下 解压即可启动使用

访问测试
![ElasticSearch启动成功](/img/大数据/ElasticSearch启动成功.png)

**目录**

- bin 启动文件
- config 配置文件
    + log4j2 日志配置文件
    + jvm.options jvm相关配置
    + elasticsearch.yml ElasticSearch的配置文件 默认端口9200
- lib 相关jar包
- modules 功能模块
- plugins 插件 
- logs 日志文件

## 安装可视化插件 elasticsearch-head

- git clone git://github.com/mobz/elasticsearch-head.git
- cd elasticsearch-head
- npm install
- npm run start
- 访问 http://ip:9100/

**配置跨域** 解决连接问题

elasticsearch.yml
```yaml
http.cors.enabled: true
http.cors.allow-origin: "*"
```

# ELK

ELK是ElasticSearch、Logstash、Kibana 三大开源框架的简称。ElasticSearch是搜索平台框架，Logstash是中央数据流引擎，Kibana可以将ElasticSearch的数据流进行展示，帮助实时分析

# 安装Kibana

官网：[https://www.elastic.co/cn/kibana](https://www.elastic.co/cn/kibana)

- 下载解压
- 修改config下的kibana.yml
```yaml
#服务端口
server.port: 5601
#开放公网访问
server.host: "0.0.0.0"
#elasticsearch地址
elasticsearch.hosts: ["http://ip:9200"]
```
- 执行 ./bin/kibana 启动

![Kibana首页](/img/大数据/Kibana首页.png)

**汉化：**

- config/kibana.yml 
```yaml
i18n.locate: "zh-CN"
```

# ElasticSearch 核心概念

ElasticSearch是面向文档的。在ElasticSearch中，一切都是json

ElasticSearch与关系型数据库的结构对应关系：

| Relational Database | ElasticSearch |
| --- | --- |
| 数据库 | 索引 |
| 表 | 类型 types（过时） |
| 行 | 文档 documents |
| 字段 | fields |

ElasticSearch在后台默认把索引划分成多个分片，每一个分片可以在集群中的不同机器上进行迁移。

ElasticSearch的逻辑设计：索引 --> 类型 --> 文档

### 文档

ElasticSearch索引和搜索数据的最小单位是文档，文档有以下特性：

- 一个文档中需要包含key和value
- 可以是层次型的，文档中包含文档（json对象）
- 结构灵活，不需要提前定义好字段

### 类型

类型是文档的逻辑容器，相当于表

### 索引 

索引时类型的容器，相当于数据库

### 倒排索引（哪个傻逼翻译出来的名字）

指的是将单词或记录作为索引，将文档ID作为记录，这样便可以方便地通过单词或记录查找到其所在的文档。

# IK分词器

**安装**

[ik 7.11.1 压缩包](/img/downloads/elasticsearch-analysis-ik-7.11.1.zip)

- 解压到ElasticSearch 的 plugins下
- 重启ElasticSearch

分词方法：
- ik_smart 最少切分
- ik_max_word 最细粒度划分

```json
GET _analyze
{
  "analyzer": "ik_max_word",
  "text": "法外狂徒张三"
}

{
  "tokens" : [
    {
      "token" : "法外",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "狂徒",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 1
    },
    {
      "token" : "张三",
      "start_offset" : 4,
      "end_offset" : 6,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "三",
      "start_offset" : 5,
      "end_offset" : 6,
      "type" : "TYPE_CNUM",
      "position" : 3
    }
  ]
}
```
```json
GET _analyze
{
  "analyzer": "ik_smart",
  "text": "法外狂徒张三"
}

{
  "tokens" : [
    {
      "token" : "法外",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "狂徒",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 1
    },
    {
      "token" : "张三",
      "start_offset" : 4,
      "end_offset" : 6,
      "type" : "CN_WORD",
      "position" : 2
    }
  ]
}
```

**给ik分词器增加自己的配置**

在 config 下 IKAnalyzer.cfg.xml 文件中配置字典文件

字典文件参考dic文件编写


# Rest风格

提供了一组设计原则和约束条件，主要用于客户端与服务器交互类的软件，让设计的软件更简洁，更有层次。

- Rest命令：

| method | url | description |
| --- | --- | --- |
| PUT | localhost:9200/索引名称/类型名称/文档id | 创建文档（指定文档id） |
| POST | localhost:9200/索引名称/类型名称 | 创建文档(随机文档id) |
| POST | localhost:9200/索引名称/类型名称/文档id/_update | 修改文档 |
| DELETE | localhost:9200/索引名称/类型名称/文档id | 删除文档 |
| GET | localhost:9200/索引名称/类型名称/文档id | 通过id查询文档 |
| POST | localhost:9200/索引名称/类型名称/_search | 查询所有数据 |

# 关于索引的基本操作

>> PUT GET POST DELETE

- 创建索引（添加数据）

```json
PUT /index1/type1/document1
{
  "name": "zhangsan",
  "age": 18
}

// 返回
#! Deprecation: [types removal] Specifying types in document index requests is deprecated, use the typeless endpoints instead (/{index}/_doc/{id}, /{index}/_doc, or /{index}/_create/{id}).
{
  "_index" : "index1",
  "_type" : "type1",
  "_id" : "document1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

- 创建索引的规则：

```json
PUT /index2

{
  "mappings": {
    "properties": {
      "name": {
        "type": "text"
      },
      "age": {
        "type": "int"
      },
      "birthday": {
        "type": "date"
      }
    }
  }
}

// 返回
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "index2"
}

```

- 获得索引规则

```json
GET /index2

// 返回
{
  "index2" : {
    "aliases" : { },
    "mappings" : { },
    "settings" : {
      "index" : {
        "routing" : {
          "allocation" : {
            "include" : {
              "_tier_preference" : "data_content"
            }
          }
        },
        "number_of_shards" : "1",
        "provided_name" : "index2",
        "creation_date" : "1614303072049",
        "number_of_replicas" : "1",
        "uuid" : "vOm8CjtCSeaIpLHX3rLjAg",
        "version" : {
          "created" : "7110199"
        }
      }
    }
  }
}
```

- 如果没有指定文档的字段类型，ES会默认配置字段类型
- GET _cat 命令：获取一些基本信息

```json
GET _cat/health

// 返回
1614303665 01:41:05 my-application yellow 1 1 13 13 0 0 7 0 - 65.0%
```

- 修改：可以直接使用PUT命令进行覆盖

```json
PUT /index1/type1/document1
{
  "name": "zhangsan",
  "age": 5
}

// 返回
#! Deprecation: [types removal] Specifying types in document index requests is deprecated, use the typeless endpoints instead (/{index}/_doc/{id}, /{index}/_doc, or /{index}/_create/{id}).
{
  "_index" : "index1",
  "_type" : "type1",
  "_id" : "document1",
  "_version" : 2,   // 版本号增加
  "result" : "updated", // 类型变成update
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}
```

- 修改： 通过POST命令，如果没设置某些值，原来的值不会丢失

```json
POST /index1/type1/document1/_update
{
  "doc": { // 固定写法
    "age": 44
  }
}

// 返回
#! Deprecation: [types removal] Specifying types in document index requests is deprecated, use the typeless endpoints instead (/{index}/_doc/{id}, /{index}/_doc, or /{index}/_create/{id}).
{
  "_index" : "index1",
  "_type" : "type1",
  "_id" : "document1",
  "_version" : 3,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 2,
  "_primary_term" : 1
}
```

- 删除： DELETE

```json
DELETE /index1

// 返回
{
  "acknowledged" : true
}
```

# 关于文档的基本操作

- 条件搜索_search
```json
GET /index1/type1/document1/_search?q=name:zhangsan
```

## 复杂查询

> select 排序、分页、高亮、模糊查询、精准查询

结构：命令+json文档

```json
GET /index1/type1/document1/_search
{
    "query": {
        "match": {
            "name": "zhangsan"
        }
    }
}
```

- 过滤结果，展示列表中的某些字段： 

```json
{
    "query": {
        ...
    }
    "_source": ["name", "age"]

}
```

- 结果排序

```json
{
    "query": {
        ...
    }
    "sort": {
    "age": {
        "order": "asc"
    }
}
}

```

- 分页查询

```json
{
    "query": {
        ...
    }
    "from": 0,
    "size": 10
}
```

### 多条件查询

- 布尔值查询

```json
{
    "query": {
        "bool": {
            // and or not 
            "must": // and
            // "must_not": // not
            // "should": // or
            [
                {
                    "match": {
                        ...
                    }
                },
                {
                    "match": {
                        ...
                    }
                }
            ]
        }
    }
}
```

- 二元逻辑
    + must 与 对应and 
    + must_not 非 对应not
    + should 或 对应or

- 过滤器 filter

```json
{
    "query": {
        "bool": {
            "must": 
            [
                {
                    "match": {
                        ...
                    }
                }
            ],
            "filter": {
                "age": {
                    "gt": 10
                }
            }
        }
    }
}
```

- 条件区间
    + gt: great than 
    + gte: great than equal
    + lt: little than
    + lte: little than equal

- 匹配多条件: 空格隔开
    + 查询结果中会给出score表示匹配度

```json
{
    "query": {
        "match": {
            "tags": "男 技术 暖"
        }
    }
}
```

- 精确查询

term查询直接通过倒排索引指定的词条进行精确查询

- 关于分词：
    + term是直接精确查询
    + match会使用分词器进行解析

- 两个类型：text和keyword
    + text类型可以被分词解析
    + keyword类型不能被分词解析

```json
GET _analyze
{
    "analyzer": "keyword", // 不会被分词
    "text": "你的答案"
}
```

- 精确查询多个值： 可使用should

### 高亮

```json
{
    "quern": {
        ...
    }
    "highlight": {
        "fields": { // 设置高亮字段，会在结果上增加<em>标签
            "name": {}
        }
    }
}
```

- 可以自定义高亮样式

```json
{
    "quern": {
        ...
    }
    "highlight": {
        "pre_tags": "<p class='key' style='color:red'>",
        "post_tags": "</p>",
        "fields": { // 设置高亮字段，会在结果上增加<em>标签
            "name": {}
        }
    }
}
```
