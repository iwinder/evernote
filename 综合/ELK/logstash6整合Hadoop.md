---
title: logstash6整合Hadoop 
tags: Logstash,Hadoop 
grammar_cjkRuby: true
---

## 前提
本文是之前elk的后续，故默认已搭建好logstash等elk相关环境。侧重点是Hadoop安装以及其与logstash的Output插件的整合。

假设存在两台服务器并处于同一局域网中，分别是192.168.0.79和192.168.0.80，系统均是CentOS7，且均已安装Java 8。ELK系统已部署在192.168.0.79，Hadoop将部署于192.168.0.80。

原则Hadoop集群需要ssh免密登陆配置，以便操作节点的启动与停止，这里暂不涉及此方面需求，故舍去。相关的可在 [官方文档-Hadoop: Setting up a Single Node Cluster.](https://hadoop.apache.org/docs/r2.8.5/hadoop-project-dist/hadoop-common/SingleCluster.html) 查看。

## 下载Hadoop 
可从官方[ Apache Download Mirrors](http://www.apache.org/dyn/closer.cgi/hadoop/common/)获取链接下载。本文下载的为Hadoop-2.8.5：
```
wget http://mirrors.hust.edu.cn/apache/hadoop/common/hadoop-2.8.5/hadoop-2.8.5.tar.gz
```

192.168.0.80的Java目录为home/parim//apps/jdk1.8.0_181

