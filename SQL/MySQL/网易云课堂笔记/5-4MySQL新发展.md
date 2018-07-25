---
title: 5-4MySQL新发展
tags: MySQL
grammar_cjkRuby: true
description: 网易DBA微专业笔记
---

## MySQL版本时间轴

![enter description here][1]


  
  ## MySQL 5.7简介
  
  - **更强大的功能**：原生JSON支持；新增安全性功能；新增地理位置支持GeoJSON,GeoHash特性
  - **更强大的性能**：某些场景性能较MySQL 5.6提升2-3倍，InnoDB只读事务优化；临时表改进；并行复制优化。
  - **更强大的运维**：Innodb buffer Pool动态修改；sys schema引入；在线修改表结构等

## 功能-JSON
- 原生支持JSON。包括字段的JSON类型和一系列JSON处理函数
- BLOB存储
- 混合存储结构化与非结构化数据
- 事务特性

### 实例

![enter description here][2]
第一个是查询记录第5个元素的值

第二个查询是查询第一个元素为3的记录。

## 功能-安全性
- 初始化数据库：root密码由空变成了随机密码；不在自动创建test数据库
- 设置密码过期策略
```
ALTER USER 'user'@'localhost' PASSWORD EXPIRE INTERVAL 30 DAY;
```
- 锁定用户、解锁用户
```
ALTER USER 'user'@'localhost' ACCOUNT LOCK;
ALTER USER 'user'@'localhost' ACCOUNT UNLOCK;
```
## 功能-虚拟列
```
sum_price decimal(10,2) GENERATED ALWAYS AS(qty * price) STORED
```
不是用户插入的，而是通过实际列动态计算出来的列

**可赋予属性**：
- STORED--实际物理存储在数据库中的
- VIRTURAL--真正的虚拟列，不会真的存储，仅在查询时动态生成。
![enter description here][3]

 ## 性能-只读事务
 - 绝大数业务都是读多写少，优化只读事务意义重大。
 - 只读事务
 ```
 start transaction read only
 autocommit 下的不加锁的select语句
 ```
 - MySQL5.7通过不再为只读事务分配事务ID,不分配回滚段开销，减少锁竞争等多种方式，优化了只读事务的开销。
## 性能-临时表
- 临时表：
不需要严格一致性，重启MySQL或断开连接生命周期结束；
只在当前会话中可见。
- 优化策略：
元数据修改不持久化

DML操作不写redo，关闭change buffer

通过减少不必要的IO操作，提高临时表各种操作的性能

## 性能-并行复制
基于MariaDB并行复制策略的改进，提升提交并行度
基于组提交的并行复制：从库安装主库的事务提交顺序提交，一个组提交的事务都是可以并行回放的。
slave-parallel-type:
DATABASE(默认) 基于库的并行复制方式
LOGICAL_CLOCK --基于组的并行复制
  
![enter description here][4]


## 运维-Innodb Buffer Pool动态修改
```
mysql> SET GLOABL innodb_buffer_pool_siez = 402653184
```
- innodb_buffer_pool_size = N*(innodb_buffer_pool_chunk_size*innodb_buffer_pool_instances)
innodb_buffer_pool_chunk_size：mysql中的buffer pool以chunk为单位增长和收缩的，innodb_buffer_pool_instances值可以多个buffer pool实例。
- 调整buffer pool的过程，用户的新请求都会阻塞
- 查看buffer pool调整状态

SHOW STATUS WHERE Variable_name = 'InnoDB_buffer_pool_resize_status'或查看error log

## 运维-sys schema
- sys schema 是MySQL5.7.7中引入的一个系统库，包含了一系列视图、函数或存储过程
- 包含：系统运行状态；用户会谈信息与资源占用；数据对象信息；SQL执行情况；索引覆盖；锁信息
- 基于performance schema的汇总与统计
- 详细内容查看手册：https://dev.mysql.com/doc/refman/5.7/en/sys-schema-views.html

全表扫描的查询语句
```
select * from statements_with_full_table_scans;
```
用户会话信息与资源占用
```
select * from host_summary;
```
Innodb锁等待情况
```
select * from innodb_lock_waits;
```
## 运维-在线修改表结构
- 在线修改表结构：
是否拷贝表；是否允许DML;是否允许Select；
- MySQL 5.7 版本新增ALTER TABLE RENAME INDEX 无需Copy Table
- 支持较好的操作：新增索引、删除索引、重命名字段
- 支持不好的操作：新增字段、删除字段、修改字段类型

  [1]: https://assets.windcoder.com/xiaoshujiang/mysql_study_xinbanbentexin01.png "mysql_study_xinbanbentexin01"
  [2]: https://assets.windcoder.com/xiaoshujiang/mysql_study_xinbanbentexin02.png "mysql_study_xinbanbentexin02"
  [3]: https://assets.windcoder.com/xiaoshujiang/mysql_study_xinbanbentexin03.png "mysql_study_xinbanbentexin03"
  [4]: https://assets.windcoder.com/xiaoshujiang/mysql_study_xinbanbentexin04.png "mysql_study_xinbanbentexin04"