---
title: 5-3MySQL分布式架构
tags: MySQL
grammar_cjkRuby: true
description: 网易DBA微专业笔记
---

## 分布式数据库
区别单机，不因单台服务器的性能瓶颈上限
一般由管理节点台机器、计算节点、数据节点组成
数据节点统一由分布式数据库管理节点控制
## 分布式数据的分类
Shared Nothing VS Shared Anyhing

三个计算node节点对应三个数据节点。每个数据节点没有交点，整个分布式数据库的数据的是三个的总和。

三个计算节点同时访问一个数据，即同时访问所有数据。

![enter description here][1]

![enter description here][2]

![enter description here][3]

## MySQL分布式架构

- 分布式事务处理与XA协议
- MySQL分布式事务处理
- MySQL分布式事务的局限与改进

### 分布式事务
Distributed(XA) Transactions

也就是通常说的XA事务

2pc(Two Phase Commit) Transactions
一般依据此协议。

![enter description here][4]

0号节点为管理节点，1到n-1为数据节点，事务进行提交时，0号节点向下面节点询问是否可以提交，此时下面节点返回自己是否可以提交，若所有节点都返回可以提交，则0号向下下发一个具体提交的指令。若一个节点返回自己还未准备好，则整个事务进行等待或回滚的操作。这就是两个阶段提交，首先管理节点询问下面节点是否可以提交，下面节点向他确定自己的状态，若所有节点都可以提交则执行提交。

### MySQL的分布式事务
什么是分布式事务处理
分布式事务一般包括3个部分：
- APP

一般指事务的发起者
- Resource Manager,RM

资源管理器RM，主要是提供外部程序进行资源共享访问。

- Transaction Manager/Coordinator(TM or TC)

TM或者TC是事务的协调者，对于分布式事务，在提交时，可能有部分节点会提交失败，这个时候，需要TC来决定那些事务需要重新提交，哪些事务需要回滚。

## 常见的分布式数据库架构


  [1]: https://assets.windcoder.com/xiaoshujiang/mysql_study_fenbushi01.png "mysql_study_fenbushi01"
  [2]: https://assets.windcoder.com/xiaoshujiang/mysql_study_fenbushi02.png "mysql_study_fenbushi02"
  [3]: https://assets.windcoder.com/xiaoshujiang/mysql_study_fenbushi03.png "mysql_study_fenbushi03"
  [4]: https://assets.windcoder.com/xiaoshujiang/mysql_study_fenbushi04.png "mysql_study_fenbushi04"