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
  WHERE ROWNUM > 10 AND ROWNUM < 21;
```





