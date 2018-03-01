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

[参考1](https://www.2cto.com/database/201303/193063.html)

## 部分字段
```
sysdate 当前日期
hibernate_sequence.nextval  hibernate中的自增id生成
```

## 错误记录

### ora-01789:query block has incorrect number of result columns

这种问题一般出现在用UNION将若干个select连接到一起，由于select后的列的个数不相等而造成的。
比如
```
select userid from usercustom 
union
select userid,loginname from usercustom
```
