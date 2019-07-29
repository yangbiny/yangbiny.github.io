---
layout: post
title:  "ElasticSearch 之 RollOver"
date:   2019-07-29
desc: "ElasticSearch 之 Rollover的使用"
keywords: "ElasticSearch,RollOver"
categories: [Elasticsearch,Distributed]
tags: [ElasticSearch,RollOver]
icon: icon-html
---
# RollOver
## 作用
当某个别名指向的实际索引过大时，自动将别名指向下一个索引
## 参数
1. max-size：分片的最大大小
2. max-docs：最大文档数
3. max-age：索引的最大存活时间

## 使用

### 1. 创建一个策略

`
PUT _ilm/policy/datastream_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_age": "7m",
            "max_docs": 5,
            "max_size": "5m"
          }
        }
      },
      "delete": {
        "min_age": "1d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
`

<br>

### 注意
1. datastream_policy为策略的名称
2. 定义rollover action<br>
`
"rollover": {
            "max_age": "7m",
            "max_docs": 5,
            "max_size": "5m"
}`

### 2.创建一个模板
`PUT _template/datastream_template
{
  "index_patterns": ["datastream-*"],                 
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1,
    "index.lifecycle.name": "datastream_policy",      
    "index.lifecycle.rollover_alias": "datastream"    
  }
}`

### 3.创建索引
`PUT datastream-000001
{
  "aliases": {
    "datastream": {
      "is_write_index": true
    }
  }
}`
现在已经完成创建<br>
注意：
1. 如果最大文档数是3，当插入6条数据的时候，并不会直接创建一个，而是在当前索引上直接创建
6个文档，如果需要创建出一个新的文档，需要手动执行，才会完成新的索引的创建
`POST /datastream/_rollover
{
  "conditions": {
    "max_docs": 3
  }
}`

