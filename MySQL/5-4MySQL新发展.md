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


  [1]: https://assets.windcoder.com/xiaoshujiang/mysql_study_xinbanbentexin01.png "mysql_study_xinbanbentexin01"
  [2]: https://assets.windcoder.com/xiaoshujiang/mysql_study_xinbanbentexin02.png "mysql_study_xinbanbentexin02"
  
  