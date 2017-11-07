---
title: Oracle部分语句用法
tags:Oracle,SQL
grammar_cjkRuby: true
---


##  ROW_NUMBER () OVER (ORDER BY) 

实例：
```
row_number() over(partition by a order by b)
```