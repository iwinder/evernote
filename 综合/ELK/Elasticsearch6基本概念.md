---
title: Elasticsearch6基本概念
tags: Elasticsearch
grammar_cjkRuby: true
---

### 近实时（Near Realtime, NRT）
Elasticsearch是一个近乎实时的搜索平台。这意味着从索引文档到可搜索文档的时间有一点延迟（通常是一秒）。
### 集群（Cluster ）

集群是一个或多个节点（服务器）的集合，它们共同保存您的整个数据，并提供跨所有节点的联合索引和搜索功能。群集由唯一名称标识，默认情况下为“elasticsearch”。
此名称很重要，因为如果节点设置为按名称加入群集，则该节点只能是群集的一部分。

### 节点（Node）