---
layout: post
title:  "ElasticSearch 之 RollOver"
date:   2017-01-31
desc: "3 Steps (2 minutes) to Setup Your Personal Website with Jalpc"
keywords: "Jalpc,Jekyll,gh-pages,website,blog,easy"
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


