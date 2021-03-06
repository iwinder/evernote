---
title: Hadoop基础 
tags: Hadoop
grammar_cjkRuby: true
---

Apache Hadoop是一款用于可靠，可扩展的分布式计算的开源软件。

Apache Hadoop软件库是一个允许使用简单的编程模型跨计算机集群分布式处理大型数据集的框架。它旨在从单个服务器扩展到数千台计算机，每台计算机都提供本地计算和存储。

Hadoop本身不是依靠硬件来提供高可用性，而是设计用于检测和处理应用程序层的故障，从而在计算机集群之上提供高可用性服务，每个计算机都可能容易出现故障。

Hadoop框架包括以下四个模块：

- **Hadoop Common:** 支持其他Hadoop模块的常用实用程序。
- **Hadoop Distributed File System (HDFS™):** 一种分布式文件系统，提供对应用程序数据的高吞吐量访问。
- **Hadoop YARN:** 作业调度和集群资源管理的框架。
- **Hadoop MapReduce:** 基于YARN的用于并行处理大型数据集的系统。

最近在官方首页出现了一个新的模块[Hadoop Ozone](https://hadoop.apache.org/ozone/)，其功能是提供Hadoop的对象存储。目前处于alpha版本，docs文档中暂未提及。

### 优点
- 高可靠性：Hadoop 按位存储和处理数据的能力值得人们信赖。
- 高可扩展性： Hadoop 是在可用的计算机集簇间分配数据并完成计算任务的，这些集簇可以方便地扩展到数以干计的节点中。
- 高效性： Hadoop能够在节点之间动态地移动数据，并保证各个节点的动态平衡，因此处理速度非常快。
- 高容错性： Hadoop能够自动保存数据的多个副本，并且能够自动将失败的任务重新分。
- 低成本：与一体机、商用数据仓库以及 QlikView、 Yonghong Z- Suites 等数据集市相比，Hadoop 是开源的，项目的软件成本因此会大大降低。

## Hadoop HDFS

HDFS是一种设计用于在通用硬件(commodity hardware)上运行的分布式文件系统。最初是作为Apache Nutch网络搜索引擎项目的基础设施而构建的。

- HDFS具有高度容错能力，旨在部署在低成本硬件上。

- HDFS提供对应用程序数据的高吞吐量访问，适用于具有大型数据集的应用程序。

- HDFS放宽了一些POSIX要求，以实现对文件系统数据的流式访问。

### HDFS架构

![图1-HDFS架构](./images/1539076382731.png)

HDFS采用master/slave架构，并具有以下元素：

#### NameNode

NameNode是一个中心服务器，负责：

- 管理文件系统的名字空间(namespace)

- 客户端对文件的访问

Namenode执行文件系统的名字空间操作，比如打开、关闭、重命名文件或目录。它也负责确定数据块到具体Datanode节点的映射。

#### DataNode

集群中的Datanode一般是一个节点一个，负责管理它所在节点上的存储:

- Datanode负责处理文件系统客户端的读写请求。

- 在Namenode的统一调度下进行数据块的创建、删除和复制。

#### Block

用户的数据以文件的形式存储在HDFS的文件系统中。  	从内部看，一个文件其实被分成一个或多个数据块，这些块存储在一组DataNode上，每个块尽可能地存储于不同的DataNode中。之前1.x默认大小为64M，2.8.5的默认大小已经是128M。

#### Rack
可简单理解为存放服务器的支架。

HDFS采用一种称为机架感知(rack-aware)的策略来改进数据的可靠性、可用性和网络带宽的利用率。

HDFS中的文件是一次写入的（除了追加和截断），并且在任何时候都有一个写入器，亦即一次写入多次读取。

NameNode它定期从群集中的每个DataNode接收Heartbeat和Blockreport。收到Heartbeat意味着DataNode正常运行。Blockreport包含DataNode上所有块的列表。

## Hadoop YARN

YARN 的基本思想是将资源管理和作业调度/监控的功能分解为单独的守护进程。
![enter description here](./images/1539165718129.png)

- **ResourceManager（RM）**：全局资源管理器，负责整个系统的资源管理和分配。它主要由两个组件构成：调度器（Scheduler）和应用程序管理器（ApplicationManager，AM）。
- **ApplicationMaster（AM）**：实际上是一个特定于框架的库，其任务是协调来自ResourceManager的资源，并与NodeManager一起执行和监控任务。
- **NodeManager**：负责启动和管理节点上的容器（Containers ）。 
- **Scheduler**：负责给各个正在运行的拥有相似的约束如容量，队列等的应用分配资源。
- **Containers**：执行AppMaster指定的任务。

ResourceManager和NodeManager构成了数据计算框架（ data-computation framework）。

## Hadoop MapReduce

### 概述
MapReduce作业（job）通常将输入数据集拆分为独立的块，这些块由map任务（map tasks）以完全并行的方式处理。框架对maps的输出（outputs）排序，然后输入到reduce 任务（reduce tasks）。通常，作业的输入和输出都存储在文件系统中。该框架负责调度任务，监控它们并重新执行失败的任务。

通常，计算节点和存储节点是相同的，即MapReduce框架和Hadoop分布式文件系统（请参阅HDFS体系结构指南）在同一组节点上运行。此配置允许框架有效地在已存在数据的节点上调度任务，从而在集群中产生非常高的聚合带宽。

MapReduce框架由一个单独的主（master）ResourceManager，每个集群节点（cluster-node）一个从(slave ) NodeManager和每个应用程序(application)的MRAppMaster组成（参见YARN体系结构指南）。


最低限度，应用程序指明输入/输出位置，并通过实现适当的接口和/或抽象类来提供map和reduce方法。再加上其他作业的参数，就构成了作业配置（job configuration）。然后,Hadoop的 job client 提交作业（jar包/可执行程序等）和配置信息给ResourceManager，后者负责将软件/配置分发给slave，调度任务并监控它们，向作业客户端（ job-client）提供状态和诊断信息。

### 输入与输出
Map/Reduce框架运转在<key, value> 键值对上，也就是说， 框架把作业的输入看为是一组<key, value> 键值对，同样也产出一组 <key, value> 键值对做为作业的输出，这两组键值对的类型可能不同。

框架需要对key和value的类(classes)进行序列化操作， 因此，这些类需要实现 Writable接口。 另外，为了方便框架执行排序操作，key类必须实现 WritableComparable接口。

一个Map/Reduce 作业的输入和输出类型如下所示：

(input) <k1, v1> -> map -> <k2, v2> -> combine -> <k2, v2> -> reduce -> <k3, v3> (output)

### Mapper 
将输入键值对(key/value pair)映射到一组中间格式的键值对集合。即将键值对通过函数映射成一组新的键值对。

### Reducer

Reducer将与一个key关联的一组中间数值集归约（reduce）为一个更小的数值集。即将键值对动key相同的值整合，通过函数映射成新的键值对。

Reducer有3个主要阶段：shuffle、sort和reduce。

##### Shuffle

Reducer的输入就是Mapper已经排好序的输出。在这个阶段，框架通过HTTP为每个Reducer获得所有Mapper输出中与之相关的分块。
#### Sort

框架在此阶段按keys（因为不同的映射器可能输出相同的键）对Reducer输入进行分组。

Shuffle和Sort阶段同时发生;

在获取map-outputs时，它们被合并。

#### Secondary Sort
如果需要中间过程对key的分组规则和reduce前对key的分组规则不同，那么可以通过 JobConf.setOutputValueGroupingComparator(Class)来指定一个Comparator。再加上 JobConf.setOutputKeyComparatorClass(Class)可用于控制中间过程的key如何被分组，所以结合两者可以实现按值的二次排序。

#### Reduce

如果需要中间过程对key的分组规则和reduce前对key的分组规则不同，那么可以通过 JobConf.setOutputValueGroupingComparator(Class)来指定一个Comparator。再加上 JobConf.setOutputKeyComparatorClass(Class)可用于控制中间过程的key如何被分组，所以结合两者可以实现按值的二次排序。

[Hadoop文档-0.18](http://hadoop.apache.org/docs/r1.0.4/cn/index.html)

[Apache Hadoop 2.8.5](http://hadoop.apache.org/docs/r2.8.5/index.html)

[Hadoop 系列（一）基本概念](https://www.cnblogs.com/binarylei/p/8903601.html)

[Hadoop MapReduce](https://www.w3cschool.cn/hadoop/flxy1hdd.html)

[Hadoop官方文档翻译——YARN Architecture（2.7.3）](https://www.linuxidc.com/Linux/2016-12/138028.htm)