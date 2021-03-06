<!-- ---
title: SQL常用笔记
tags: MySQL,Oracle
grammar_cjkRuby: true
--- -->
## where in 查询
Oracle：当前所用版本中，限制in中的参数不能超过 1000个。当超出时会被报错"ORA-01795异常(where in超过1000)的解决"。

MySQL：有人说有限制，有人说没限制。但尽量也是不要太多为好，容易造成全表扫描。
解决方案:
- 使用 inner join 代替 in
- 使用 left in ... where ... is null 代替 not in

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

在某些数据库中， LEFT JOIN 称为 LEFT OUTER JOIN。

基础：
```
SELECT p.LastName, p.FirstName, o.OrderNo
FROM Persons p
LEFT JOIN Orders o
ON p.Id_P=o.Id_P
ORDER BY p.LastName
```
- LEFT JOIN 关键字会从左表 (Persons) 那里返回所有的行，即使在右表 (Orders) 中没有匹配的行。 
- 当右表未匹配时会以null展示

进阶
```
SELECT p2.LastName, p2.FirstName, o2.OrderNo
FROM Persons p2
LEFT JOIN(
	SELECT p.LastName, p.FirstName, o.OrderNo
	FROM Persons p
	INNER JOIN Orders o
	ON p.Id_P=o.Id_P
)o2
WHERE o2.OrderNo is null;
```
- 先用 inner join 查询出有订单的用户，将该查询作为右表o2
- Persons 继续做左表，此时为p2
- p2 LEFT JOIN o2 查询左表所有信息，加上条件 o2.OrderNo is null 将有订单的用户过滤掉
- 最终出来的是，没有下订单的用户，即代替了 not in实现。

参考与扩展：
[SQL INNER JOIN 关键字](http://www.w3school.com.cn/sql/sql_join_inner.asp)
[SQL LEFT JOIN 关键字](http://www.w3school.com.cn/sql/sql_join_left.asp)
[inner join 和where 区别](https://blog.csdn.net/qingtanlang/article/details/2133816)

## 时间判断

### 判断是否为今天是否存在记录
对于当天的判等简单点说就是：mysql 用to_days(now())，Oracle 用trunc(sysdate)
```
//Oracle
(select count(*) from ugc_activity_vote_record uavr where uavr.vote_id = o.vote_id and uavr.option_id = uavo.id  and uavr.created_by = #{user.id} and trunc(uavr.created_date) = trunc(sysdate)  ) is_voted

//MySQL
(select count(*) from ugc_activity_vote_record uavr where uavr.vote_id = o.vote_id and uavr.option_id = uavo.id  and uavr.created_by = #{user.id} and to_days(uavr.created_date) = to_days(now())  ) is_voted,

```
1. [mysql 查询当天、本周，本月，上一个月的数据](https://www.cnblogs.com/benefitworld/p/5832897.html)

2. [oracle sql日期比较](https://www.cnblogs.com/lanblogs/p/3257551.html)

3. [Oracle 查询时间在当天的数据](https://www.cnblogs.com/simeone/p/4189264.html)

### 时间转换

#### Oracle
to_char(sysdate,'yyyy-mm-dd') 做日期格式化;
trunc(sysdate) 做截取；
#### MySQL
to_days()

## 空值补全

```
//Oracle
nvl(uavo.votes,0)  as votes,(
	SELECT
		COUNT (*)
	FROM
		ugc_activity_vote_option vo1
	WHERE
		vo1.votes > nvl(uavo.votes,0)
		and vo1.vote_id = o.vote_id
) + 1 rank
//MySQL
IFNULL(uavo.votes,0) as votes,(
	SELECT
		COUNT (*)
	FROM
		ugc_activity_vote_option vo1
	WHERE
		vo1.votes > IFNULL(uavo.votes,0)
		and vo1.vote_id = o.vote_id
) + 1 rank
```
