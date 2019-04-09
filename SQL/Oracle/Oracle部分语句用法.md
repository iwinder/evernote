<!-- ---
title: Oracle部分语句用法
tags:Oracle,SQL
grammar_cjkRuby: true
--- -->


##  ROW_NUMBER () OVER (ORDER BY) 

实例：
```sql
row_number() over(partition by a order by b)
```

将查询结果按照a字段分组（partition），

然后组内按照b字段排序，至于asc还是desc，可自行选择，

然后为每行记录返回一个rownumber用于标记顺序

[参考1](https://www.2cto.com/database/201303/193063.html)

## 部分字段
```sql
sysdate 当前日期
hibernate_sequence.nextval  hibernate中的自增id生成
```

## 错误记录

### ora-01789:query block has incorrect number of result columns

这种问题一般出现在用UNION将若干个select连接到一起，由于select后的列的个数不相等而造成的。
比如
```sql
select userid from usercustom 
union
select userid,loginname from usercustom
```
[ora-01789:query block has incorrect number of result columns](http://blog.csdn.net/wyzxg/article/details/4261014)

### SQL Error: 1461, SQLState: 72000 can bind a LONG value only

即：SQL Error: 1461, SQLState: 72000 ORA-01461: 仅能绑定要插入 LONG 列的 LONG 值

#### mysql

可以直接修改为longtext:

```sql
-- 修改用户行为错误记录中信息字段类型为longtext
alter table sys_user_behavior_err_info modify column info longtext
```

#### oracle方案
```sql
-- 修改行为错误记录表info的字段类型方案（Oracle本身无法直接从VARCHAR2转为clob）
-- 若有DBMS_REDEFINITION（在线重定义表）权限，亦可考虑通过DBMS_REDEFINITION更改字段
-- 1.修改用户行为错误记录中信息字段类型为clob
ALTER TABLE SYS_USER_BEHAVIOR_ERR_INFO add info_new clob;
update  SYS_USER_BEHAVIOR_ERR_INFO  set info_new=info,info=null; 
commit; 
ALTER TABLE SYS_USER_BEHAVIOR_ERR_INFO  modify info long; 
ALTER TABLE SYS_USER_BEHAVIOR_ERR_INFO  modify info clob; 
update SYS_USER_BEHAVIOR_ERR_INFO  set info=info_new,info_new=null; 
commit;
ALTER TABLE  SYS_USER_BEHAVIOR_ERR_INFO  drop column info_new; 
-- 2.查询到表中的索引，如：数据库SPARKDEV中的SYS_C0051930，每个数据库中的索引名称可能名称不同。
select * from user_indexes  where table_name ='SYS_USER_BEHAVIOR_ERR_INFO'
-- 3.对2中查询到的index_type为NORMAL的索引执行重建，一般只有1条，若查询为空则无需执行。此处仅是示例，以实际情况为准。当存在时若不执行2和3可能会导致存储时报错：ORA-01502: 索引 'SPARKDEV.SYS_C0051930' 或这类索引的分区处于不可用状态
alter index SPARKDEV.SYS_C0051930 rebuild online;
```

#### 扩展
[（Clob的写入和读取-java）更新数据库报错：SQL Error: 1461, SQLState: 72000 ORA-01461: 仅能绑定要插入 LONG 列的 LONG 值](https://www.cnblogs.com/yingsong/p/5685790.html)
[Oracle中表列由VARCHAR2类型改成CLOB](https://blog.csdn.net/jssg_tzw/article/details/40829867)