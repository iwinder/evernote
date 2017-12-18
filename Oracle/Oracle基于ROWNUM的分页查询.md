---
title: Oracle基于ROWNUM的分页查询 
tags: 数据库,Oracle
grammar_cjkRuby: true
---

## 基础
Oracle中，ROWNUM（行号）属于伪列（Pseudocolumn），其只在数据库内部使用，通过DESCRIBE命令查看表结构也是无法看到其描述的。

在查询时ROWNUM自动返回一个数字作为本行的行号，第一行为1，依次类推。

普通查询可以使用如下：
```
SELECT *
  FROM users
  WHERE ROWNUM < 11;
```
## 分页
当用来做分页查询时，用惯mysql的我看着上面的格式，习惯性的写出如下格式：

```
SELECT *
  FROM users
  WHERE ROWNUM > 1 AND ROWNUM < 11;
```
此时奇迹出现啦，本来不添加```ROWNUM > 1```时还能查出数据，添加后一条也不见了。

查找一番，发现Oracle机制是这样的：因为第一条数据行号为1，不符合ROWNUM > 1，故被舍弃，此时第二条数据上位，成为新的第一条，行号随之变为1，如此下去，所有的都不符合条件从而被舍弃了，故最终一条数据都没有。






