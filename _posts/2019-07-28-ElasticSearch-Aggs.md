---
layout: post
title:  "ElasticSearch 之 聚合查询"
date:   2017-01-31
desc: "3 Steps (2 minutes) to Setup Your Personal Website with Jalpc"
keywords: "Jalpc,Jekyll,gh-pages,website,blog,easy"
categories: [Elasticsearch]
tags: [ElasticSearch,聚合查询]
icon: icon-html
---
## 求总和
`
GET /impassive/_search
 {
   "size": 1, 
   "aggs": {
     "age_sum": {
       "sum": {
         "field":"age"
       }
     }
   }
 }
`
## 求平均值
`
 GET /impassive/_search
  {
    "size": 1, 
    "aggs": {
      "age_avg": {
        "avg": {
          "field":"age"
        }
      }
    }
  }
`
## 求基数
  基数：互不相同的数据的个数，比如2和3的基数为2，5和5的基数为1<br>
`
GET /impassive/_search
{
  "size": 1, 
  "aggs": {
    "age_avg": {
      "cardinality": {
        "field":"age"
      }
    }
  }
}
`

## 分组
根据字段显示分组信息<br>
`
GET /impassive/_search
 {
   "size": 1, 
   "aggs": {
     "age_avg": {
       "terms": {
         "field":"age"
       }
     }
   }
 }
`
<br>
显示结果<br>
`{
   "took" : 4,
   "timed_out" : false,
   "_shards" : {
     "total" : 1,
     "successful" : 1,
     "skipped" : 0,
     "failed" : 0
   },
   "hits" : {
     "total" : {
       "value" : 6,
       "relation" : "eq"
     },
     "max_score" : 1.0,
     "hits" : [
       {
         "_index" : "impassive",
         "_type" : "_doc",
         "_id" : "6",
         "_score" : 1.0,
         "_source" : {
           "post_date" : "2009-11-15",
           "message" : "Hello",
           "user" : "Impassive",
           "interest" : "eat",
           "age" : 1
         }
       }
     ]
   },
   "aggregations" : {
     "age_avg" : {
       "doc_count_error_upper_bound" : 0,
       "sum_other_doc_count" : 0,
       "buckets" : [
         {
           "key" : 1,
           "doc_count" : 2
         },
         {
           "key" : 4,
           "doc_count" : 1
         },
         {
           "key" : 95,
           "doc_count" : 1
         },
         {
           "key" : 100,
           "doc_count" : 1
         },
         {
           "key" : 481,
           "doc_count" : 1
         }
       ]
     }
   }
 }`
 
### 练习
 根据年龄分组并求平均值，然后根据平均值进行降序排序<br>
 `GET /impassive/_search
  {
    "size": 1, 
    "aggs": {
      "age_group": {
        "terms": {
          "field":"age",
          "order":{
            "age_of_avg":"desc"
          }
        },
        "aggs":{
          "age_of_avg":{
            "avg":{
              "field":"age"
            }
          }
        }
      }
    }
  }`