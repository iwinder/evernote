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

CBO优化模式下，Oracle可以将外层的查询条件推到内层查询中，以提高内层查询的执行效率。

对于第2个查询语句，第二层的查询条件```WHERE ROWNUM <= 11```就可以被Oracle推入到内层查询中，这样Oracle查询的结果一旦超过了ROWNUM限制条件，就终止查询将结果返回了。 

而第1个查询语句，由于查询条件```WHERE RN > 1 AND RN < 11``` 是存在于查询的第三层，而Oracle无法将第三层的查询条件推到最内层（即使推到最内层也没有意义，因为最内层查询不知道RN代表什么）。

因此，对于第1个查询语句，Oracle最内层返回给中间层的是所有满足条件的数据，而中间层返回给最外层的也是所有数据。数据的过滤在最外层完成，显然这个效率要比第一个查询低得多。

#### 注意
- rownum不能以任何基表的名称作为前缀。 　
- 子查询中的rownum必须要有别名，否则还是不会查出记录来，这是因为rownum不是某个表的列，如果不起别名的话，无法知道rownum是子查询的列还是主查询的列。
- 查询rownum在某区间的数据，rownum对小于某值的查询条件为true，rownum对于大于某值的查询条件直接认为是false的，但是可以间接的让它转为认为是true的。那就必须使用子查询。

## 排序
分页基本完成，但很多时候不只需要分页，还需要按照某条件排序。很简单啦，如此有了这样的写法：
```
SELECT *
  FROM USERS
  WHERE ROWNUM < 11
  ORDER BY last_name;
```
然而执行后，发现好像有点不对啊，结果好像和上面执行的不太一样啊，ROWNUM顺序也是乱的。
原来：如果在同一查询中一个 ```ORDER BY```语句跟随在ROWNUM 之后，那么结果行将会被```ORDER BY```从句重新排列。查询结果会依赖访问行的方式而不同。例如，如果ORDER BY导致Oracle使用索引访问数据，可能以不同的顺序返回数据行。因此，上面的语句与上一个语句将会返回不同的结果。

如果在子查询中嵌入ORDER BY子句并且将ROWNUM条件放在顶层查询中，可以使得ROWNUM在排序之后应用。例如，以下查询返回具有最小用户编号的10个用户。这种查询有时被称为top-N报表：
```
SELECT *
  FROM (SELECT * FROM USERS ORDER BY username)
  WHERE ROWNUM < 11;
```
再略微改下，还可以使用ROW_NUMBER，方法二：
```
SELECT *
  FROM (SELECT *,ROW_NUMBER() OVER(ORDER BY username) RN FROM USERS )
  WHERE ROWNUM < 11;
```


## 参考资料
[3.9 ROWNUM Pseudocolumn](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sqlrf/ROWNUM-Pseudocolumn.html#GUID-2E40EC12-3FCF-4A4F-B5F2-6BC669021726)

[7.186 ROW_NUMBER]()https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sqlrf/ROW_NUMBER.html#GUID-D5A157F8-0F53-45BD-BF8C-AE79B1DB8C41






