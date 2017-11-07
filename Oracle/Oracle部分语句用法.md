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

将查询结果按照a字段分组（partition），

然后组内按照b字段排序，至于asc还是desc，可自行选择，

然后为每行记录返回一个rownumber用于标记顺序