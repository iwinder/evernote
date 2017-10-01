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
- Distributed(XA) Transactions

也就是通常说的XA事务

- 2pc(Two Phase Commit) Transactions
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
 
 ![enter description here][5]
 
 图中蓝色线是TM，TM协调底层RM是否准备好是否可以提交，RM去向它汇报自己的状态，如果可以提交，TM在让其提交。
 在MySQL中一般MySQL充当RM角色，TM需要单独实现，另外去开发一套程序去实现。
 
 ### XA In MySQL
 Only a Resource Manager
 MySQL中仅充当资源管理器的角色，事务管理器没有做到
 InnoDB Only：innodb_support_xa
 目前一般只有InnoDB引擎支持才支持XA 
 
 执行xa事务实例：
 ```
 # 开启xa事务‘dba’
 xa start 'dba';
 # 执行语句
 insert into dba_test values(100,100);
 # 一般事务可执行commit或rollback，但xa事务非如此
 commit /rollback
 # xa事务需要end之前定义的xa事务‘dba’
 xa end 'dba'
 #
 xa prepare 'dba';
 # 提交记录
 xa commit 'dba''
 ```
 
 
 若回滚xa事务
 ```
 xa start '123';
 insert into dba_test values(200,200);
 xa end '123';
 xa rollback '123';
 ```
 
 **xa prepare 'dba';**
  
 在分布式事务协议中，所有处于 prepare 的事务都是可以提交的事务。
 
 ### MySQL XA 事务的局限性
 - 客户端退出，prepare状态的事务被回滚。
 根据分布式事务的原理，所有prepare成功的事务都应该被提交，有下面一个客户端操作:
 ```
 xa start '111'
 insert into t values(1);
 xa end '111';
 xa prepare '111';
 ```
 若此时客户端退出了，再连上去会发现这个事务会被MySQL回滚掉，按照标准协议本不应该被回滚掉，这是MySQL与协议的冲突。
 
 - Server Crash后重启，提交prepare的事务，造成主从数据不一致。
 
 binlog中会被丢掉，与数据中的不一致。
 
 在
 ```
  xa prepare '123';
 ```
 时，在另一终端执行：
 ```
 kill -9 3025
 ```
 MySQL将重启，执行：
 ```
 xa recover;
 ```
 连到里面，可以看到重启之前的事务‘123’，
 此时执行
 ```
 xa commit '123';
 ```
 提交后，主库没问题可以提交，但不会出现在binlog中，从而导致从库中数据丢失从而造成数据不一致。
 
 #### 为何没人修复
 - 看似简单，其实解决代价很大
 - MySQL的XA事务用的真不多，官方觉得没必要
 MySQL官方最终的修复
 
 之前没有记录到binlog中，修复是将其记录到binlog中。
 
 5.7之前是有风险的，5.7做的修复。

## 开源分布式软件Mycat

### Mycat的由来
- Java 语言编写
- JDK版本要求>1.7版本
- 基于阿里的cobar实现，完全额兼容MySQL协议
- 有专业的维护团队，更新频繁，文档写的比较好
- 功能较为完善
- 分布式事务支持的不够好
- 支持各种数据库：MySQL、Oracle、MongoDB
- 目前不支持在线水平扩容
- 要做MySQL界的Tomcat

### Mycat架构

![enter description here][6]

- 管理层
- MySQL协议层
- SQL解析层
### Mycat功能特点
- 数据拆分，丰富的拆分策略，数据库水平扩容容易
- 读写分离，支持用户友好的读写的分离策略
- 兼容MySQL协议，用户可以以最小的代价接入Mycat

#### 数据拆分（一）
- 枚举算法

如用户信息，想根据省份市级拆分，可设置一个枚举算法，设置某些值落在某些上面。
```
<tableRule name="sharding-by-intfile">
	<rule>
		<columns>user_id</columns>
		<algorithm>hash-int</algorithm>
	</rule>
</tableRule>
<function name="hash-int" class="org.opencloudb.route.function.PartitionByFileMap">
	<property name="nampFile">partition-hash-int.txt</property>
	<property name="type">0</property>
	<property name="defaultName">0</property>
</function>
```
- Range 算法

简单的如时间区间

```
<tableRule name="auto-sharding-long">
	<rule>
		<columns>user_id</columns>
		<algorithm>rang-long</algorithm>
	</rule>
</tableRule>
<function name="rang-long" class="org.opencloudb.route.function.AutoPartitionByLong">
	<property name="nampFile">autopartition-long.txt</property>
	<property name="defaultName">0</property>
</function>
```
- Hash取模算法

最主流也是用的最多的数据拆分算法。根据某个字段取模，并均散到各节点上。

```
<tableRule name="mod-long">
	<rule>
		<columns>user_id</columns>
		<algorithm>mod-long</algorithm>
	</rule>
</tableRule>
<function name="mod-long" class="org.opencloudb.route.function.PartitionByMod">
	<! --how many data nodes-->
	<property name="count">3</property>
</function>
```
### Mycat里的重要概念

#### 全局ID

MySQL:auto_increment
Mycat：global sequnce number
全局序列ID。
##### 实现方式：
conf/server.xml中
```
<system><property name="sequnceHandlerType">0</property></system>
```

- 基于文件
 实现简单、安全性较差
 
 趋势是递增，不保证是递增。
 ```
 cat sequence_conf.properties
 #default gloabl swquence
 GLOABL.HISIDS=    //基本没有，为空即可
 GLOABL.MINID=10001 //最小分配的id是多少，如本处为10001
 GLOABL.MAXID=20000 //最大ID是多少，如此处为20000，超出会报错
 GLOABL.CURID = 10000 //每次分配时分配1w，批量分配。每次Mycat分配ID时都需要更新此文件，更新代来开销，分配速度可能受更新速度影响。
 ```
- 基于本地时间戳
不依赖任何服务
ID分配一般比较大
![enter description here][7]

- 基于数据库
比直接存文件要安全多
但DB要支持高可用

![enter description here][8]

实际是server直接向DB发送请求进行申请，然后更新ID。

#### DataHost
#### DataBase Node(DBN)
#### 分区规则
#### 分区键
需要根据拆分的字段，如想根据实际拆分，则分区键是时间。

### Mycat的部署与安装
```
# 查看
ls - lh 
```
txt后缀
一般代表用户可以编辑的拆分算法相关的配置。
properties
一般用处不会很大，一般不会去动,
xml后缀
一般代表Mycat的一些配置信息，部署安装时会用到。
router.xml可以不用特别关心，阿里cobar遗留下来的配置
主要是rule.xml和schema.xml这些。 

### 启动Mycat

![enter description here][9]


### schema.xml
```
//vi 中语法高亮
:syntax on
```
dataHost配置Mycat下的一个mysql节点
writeTpe = 0
name 名称，
dbDriver="native" Mycat自己实现的，无需管理。
writeHost--可写节点
readHost--从节点，做一些读写分离操作。

dataNode，即DataBase Node，可理解为mysql中的一个schema或DBN,一般很少与dataHost打交道，常用dataNode打交道。

### server.xml
相关的全局server的配置文件。更改后需重启使Mycat感知更改。
两个端口
serverPort:8066 ----面向用户的端口，普通用户-》开发人员，增删改查
managerPort:9066---面向管理员的端口，面向管理员-》DBA,可做监控、表变更等更高级操作。

<user>设置用户名密码

![enter description here][10]

readOlny 为true即为只读权限。

### 使用
![enter description here][11]


  [1]: https://assets.windcoder.com/xiaoshujiang/mysql_study_fenbushi01.png "mysql_study_fenbushi01"
  [2]: https://assets.windcoder.com/xiaoshujiang/mysql_study_fenbushi02.png "mysql_study_fenbushi02"
  [3]: https://assets.windcoder.com/xiaoshujiang/mysql_study_fenbushi03.png "mysql_study_fenbushi03"
  [4]: https://assets.windcoder.com/xiaoshujiang/mysql_study_fenbushi04.png "mysql_study_fenbushi04"
  [5]: https://assets.windcoder.com/xiaoshujiang/mysql_study_fenbushi05.png "mysql_study_fenbushi05"
  [6]: https://assets.windcoder.com/xiaoshujiang/mysql_study_fenbushi06.png "mysql_study_fenbushi06"
  [7]: https://assets.windcoder.com/xiaoshujiang/mysql_study_fenbushi07.png "mysql_study_fenbushi07"
  [8]: https://assets.windcoder.com/xiaoshujiang/mysql_study_fenbushi08.png "mysql_study_fenbushi08"
  [9]: https://assets.windcoder.com/xiaoshujiang/mysql_study_fenbushi09.png "mysql_study_fenbushi09"
  [10]: https://assets.windcoder.com/xiaoshujiang/mysql_study_fenbushi010.png "mysql_study_fenbushi010"
  [11]: https://assets.windcoder.com/xiaoshujiang/mysql_study_fenbushi011.png "mysql_study_fenbushi011"