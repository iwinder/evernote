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

原因找到了，解决方案就不远了，将其略微修改下就好。格式一：

```
SELECT
	*
FROM
	(
		SELECT
			ur.*,
			ROWNUM RN
		FROM
			(SELECT * FROM USERS) ur
	)
WHERE
	RN > 1
AND RN < 11;
```
先最内层查询，然后第二层子查询同时ROWNUM成为固定列并被赋值别名RN，第三层对RN进行条件查询。

现在是内层将结果全查了一遍后做的分页，现在再略微改下。格式二：
```
SELECT
	*
FROM
	(
		SELECT
			ur.*, ROWNUM RN
		FROM
			(SELECT * FROM USERS) ur
		WHERE
			ROWNUM < 11
	)
WHERE
	RN > 1
```
### 效率对比：

声明：此为网上的普遍分析结论。

CBO优化模式下，Oracle可以将外层的查询条件推到内层查询中，以提高内层查询的执行效率。对于第2个查询语句，第二层的查询条件```WHERE ROWNUM <= 11```就可以被Oracle推入到内层查询中，这样Oracle查询的结果一旦超过了ROWNUM限制条件，就终止查询将结果返回了。 而第1个查询语句，由于查询条件```WHERE RN > 1 AND RN < 11``` 是存在于查询的第三层，而Oracle无法将第三层的查询条件推到最内层（即使推到最内层也没有意义，因为最内层查询不知道RN代表什么）。因此，对于第1个查询语句，Oracle最内层返回给中间层的是所有满足条件的数据，而中间层返回给最外层的也是所有数据。数据的过滤在最外层完成，显然这个效率要比第一个查询低得多。








