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
 
  


  [1]: https://assets.windcoder.com/xiaoshujiang/mysql_study_xinbanbentexin01.png "mysql_study_xinbanbentexin01"
  [2]: https://assets.windcoder.com/xiaoshujiang/mysql_study_xinbanbentexin02.png "mysql_study_xinbanbentexin02"
  [3]: https://assets.windcoder.com/xiaoshujiang/mysql_study_xinbanbentexin03.png "mysql_study_xinbanbentexin03"