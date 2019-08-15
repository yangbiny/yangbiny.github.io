---
layout: post
title:  "ElasticSearch 之 master选举"
date:   2019-08-11
desc: "ElasticSearch master选举"
keywords: "ElasticSearch,master"
categories: [Search]
tags: [集群,分布式,ElasticSearch]
icon: icon-html
---
## 什么是master
作用类似与JVM中的同步关键字synchronized。因为在集群中多节点血条工作时必须保证数据
一致性，但是不同的节点分布在不同的服务器上，节点操作集群中的资源的额时候就需要通过master。
