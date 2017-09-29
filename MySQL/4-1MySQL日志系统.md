## 什么是日志
日志（logo）是一种顺序记录时间流水的文件。
记录计算机程序运行过程中发生了什么
多种多样的用途
帮助分析程序问题
分析服请求的特征，流量等
判断工作是否成功执行
等等。。


## MySQL日志的分类
### 服务器日志
记录进程启动运行过程中的特殊事件，帮助分析MySQL服务遇到的问题
根据需求抓取特定的SQL语句，追踪性能可能存在的问题的业务SQL
包含：服务错误日志、慢查询日志、综合查询日志

### 事务日志
记录应用程序对数据的所有更改
可用于数据恢复
可用于实例间数据同步
包含：存储引擎事务日志、二进制日志

### 服务错误日志
记录实例启动运行过程中重要消息
配置参数--定义输出目录，无法关闭或定义输出内容，在启动时就应该指定一个日志输出目标
log_error = /data/mysql_data/node-1/mysqld.log
内容并非全是错误消息
如果mysqld进程无法正常启动首先查看错误日志

查看错误日志文件所在目录
show global variables like 'log_error'; 
查看日志文件内容：
tail -f SAM-LIT.err
note、warning信息是启动过程中的重要记录，错误时注意看error

### 慢查询日志
记录执行时间超过一定阈值的sql语句
配置参数
slow_query_log = 1 ----是否打开
slow_query_log_file = /data/mysql_data/node-1/mysql-show.log   ---输出文件位置
long_query_time = 5  ----时间阈值，执行时间超过m秒的sql
用于分析系统中可能存在性能问题的SQL
查看相关参数：
show global variables like "%slow%"
slow_query_log不设置下默认是关闭的。
查看设置的阈值：
show global variables like "%long%"
 
### 综合查询日志
如果开启将会记录系统中所有SQL语句
配置参数
general_log = 1
general_log_file = /data/mysql_data/node-1/mysql-gen.log
偶尔用于帮助分析系统问题，对性能有影响
现在线上一般不会打开，5.5以前慢查询无法记录1秒以下的慢查询，所以有些需要综合查询，5.5以后可以。同时打开会有一定的IO开销。


查询日志的输出与文件切换
日志输出参数
log_output = {file|table|none}
线上习惯定义到文件中，none通常不会配置

如果日志文件过大，可以定期截断并切换新文件
flush logs;

过程：
mv SAM-LIT.log SAM-LIT.log.old---重命名
flush logs;------生成新的

### 存储引擎事务日志
部分存储引擎拥有重做日志（redo log）
如InnoDB,TokuDB等 WAL(Write Ahead Log)机制存储引擎
日志随事务commit优先持久化，确保异常恢复不丢数据
日志顺序写性能较好

InnoDB事务日志重用机制
InnoDB事务日志采用两组文件交替重用


### 二进制日志（binlog）
binlog(binary log)
-记录数据引起数据变化的SQL语句或数据逻辑变化的内容
- MySQL服务层记录，无关存储引擎。
#### binlog的主要作用：
基于备份恢复数据
数据库主从同步
挖掘分析SQL语句

#### 开启binlog
主要参数：

1、log_bin = c:/tmp/mylog/mysql-bin  --指定文件名，也可以是完整文件路径，或者on或者off(即0或1)，若只是设置on，也是默认保存在数据文件目录下。静态初始化参数，启动后不能被修改，只要是非0，就认为是打开的。

2、sql_log_bin = 1;  ---不能被全局设置的参数，用于设置当前Session将要发生的所有修改数据库操作是否被记录，即当前Session是否记录binlog。线上一般很少在Session级别将其关闭，

3、sync_binlog = 1;-----mysql的binlog文件持久化方式。

1，随着事务提交，会自动刷新，若为0，mysql不会主动刷到磁盘上，会由操作系统的脏数据刷出自动完成。设置为0，若服务器发生异常，binlog可能会丢失，对数据造成影响。设置大于1时，意味着数据库每进行大于1的这个数的操作，期间数据库发生异常，会丢失由这个值造成的数据。设置为1，对服务器IO性能要求，但是最安全的选项。对不能丢失的数据设置为1合理，若对写的性能要求较高，可以设置为非1或0，牺牲安全保证性能。

#### 查看binlog文件 
```
show binary logs;
```

#### binlog管理

max_binlog_size= 100MB;----单个最大100MB,超过后自动创建新的。

expire_logs_days = 7;---自动保存7天以内的文件，超出的MySQL自动清理

binlog始终生成新文件，不会重用。

手工清理binlog
```
purge binary logs to 'mysql-bin.000009'---使用文件名，之前的被清理
purge binary logs before '2015-06-01 22:46:26'--使用时间，保存到指定时间为止
```

查看binlog内容
日志(log)
```
show binlog events in 'mysql-bin.000009 '
show binlog events in 'mysql-bin.000009 ' from 60 limit 3;
```

#### mysqlbinlog工具
```
mysqlbinlog c:/tmp/mylog/mysql-bin.000001
--start-datetime|--stop-datetime
--start-position|--stop-position
```

#### binlog格式
 主要参数
 ```
binlog_format = {ROW|STATEMENT|MIXED}
```

查看row模式的binlog内容
```
mysqlbinlog --base64-output=decode-rows -v c:/tmp/mylog/mysql-bin.000001
```

ROW：安全但会产生大量日志文件。
MIXED ：STATEMENT安全时使用STATEMENT，不安全时用ROW。
