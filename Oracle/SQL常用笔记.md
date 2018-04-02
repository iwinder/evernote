---
title: SQL常用笔记
tags: MySQL,Oracle
grammar_cjkRuby: true
---
## where in 查询
Oracle：当前所用版本中，限制in中的参数不能超过 1000个。
MySQL：有人说有限制，有人说没限制。但尽量也是不要太多为好，容易造成全表扫描。
解决方案:
- 使用 inner join 代替 in
- 使用 left in ... where ... is null 到代替 not in

例：

1、INNER JOIN（内连接）

- 在表中存在至少一个匹配时，INNER JOIN 关键字返回行。
- INNER JOIN 与 JOIN 是相同的。

```
--查询在Orders存在的Persons，即有订购记录的用户
SELECTp.LastName, p.FirstName, o.OrderNo
FROM Persons p
INNER JOIN Orders o
ON p.Id_P=o.Id_P
ORDER BY p.LastName
```
INNER JOIN 关键字在表中存在至少一个匹配时返回行。如果 "Persons" 中的行在 "Orders" 中没有匹配，就不会列出这些行。

2、LEFT JOIN（左连接）

LEFT JOIN 关键字会从左表 (table_name1) 那里返回所有的行，即使在右表 (table_name2) 中没有匹配的行。


