---
title: Hadoop基础 
tags: Hadoop
grammar_cjkRuby: true
---

Apache Hadoop是一款可靠，可扩展的分布式计算的开源软件。

Apache Hadoop软件库是一个允许使用简单的编程模型跨计算机集群分布式处理大型数据集的框架。它旨在从单个服务器扩展到数千台计算机，每台计算机都提供本地计算和存储。

Hadoop本身不是依靠硬件来提供高可用性，而是设计用于检测和处理应用程序层的故障，从而在计算机集群之上提供高可用性服务，每个计算机都可能容易出现故障。

Hadoop框架包括以下四个模块：

- **Hadoop Common:** 支持其他Hadoop模块的常用实用程序。
- **Hadoop Distributed File System (HDFS™):** 一种分布式文件系统，提供对应用程序数据的高吞吐量访问。
- **Hadoop YARN:** 作业调度和集群资源管理的框架。
- **Hadoop MapReduce:** 基于YARN的用于并行处理大型数据集的系统。

最近在官方首页出现了一个新的模块[Hadoop Ozone](https://hadoop.apache.org/ozone/)，其功能是提供Hadoop的对象存储。目前处于alpha版本，docs文档中暂未提及。

