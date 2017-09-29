---
title: 4-3Mysql线上运维
tags: MySQL
grammar_cjkRuby: true
---

## Mysql线上部署
### 考虑因素
- 版本选择：5.1/5.5/5.6+
- 分支选择：官方社区版、percona server、Mariadb
- 安装方式：包安装？二进制包安装？源码？---线上推荐二进制
- 路径配置、参数配置（尽量模板化、标准化）
- 一个实例多个库or多个实例单个库?
### 二进制安装Mysql
1、下载软件安装包

2、解压放到指定目录(如：/use/local)
```
#tar -zxf mysql-5.6.24-linux-glibc2.5-x86_64.tar.gz
#cd /usr/local
#mv mysql-5.6.24-linux-glibc2.5-x86_64 /usr/local/
#mv mysql-5.6.24-linux-glibc2.5-x86_64 mysql56
```
3、将Mysql目录放到PATH中
```
#cd mysql56/bin
#export PATH=/usr/local/mysql56/bin:$PATH
```
4、初始化实例，编辑配置文件并启动
```
#mkdir -p /mysqldata/node1

//初始化实例

#/usr/local/mysql56/script/mysql_install_db --user=mysql --basedir=/usr/local/mysql56 --datadir=/mysqldata/node1

#cd /mysqldata/node1
#ll -h
#cp /usr/local/mysql56/my.cnf .

//编辑配置文件
#vim my.cnf
(
[mysqld]
innodb_buffer_pool_size=100M;
basedir;
datadir;port;
server_id;socket;
join_buffer_size;sort_buffer_size;
read_rnd_buffer_size;
character-set-server;
max_connections;
log_bin;expire_logs_days;max_binlog_size;binlog_format;innodb_flush_log_at_trx_commit;sync_binlog;tmp_dir;log-error)

//启动，mysqld_safe ---mysqld的守护进程脚本
#/usr/local/mysql56/bin/mysqld_safe --defaults-file=/mysqldata/my.cnf &
//查看进程
#ps -ef|grep mysqld
```
5、账户安全设置

a、删除无关用户，对本地root用户创建密码
```
#mysql -uroot --socket=/tmp/mysql.sock
select user,host,password from mysql.user;
delete from msyql.user where user='';
delete from mysql.user where host<>'localhost';
set password for root@'localhost'=password('123456');
flush privileges;
```

b、测试test库，并删除
```
#mysql -uroot -p123456--socket=/tmp/mysql.sock 
grant select on *.* to test@'localhost' identified by '123';
#mysql -utest -p123 --socket=/tmp/mysql.sock
use test
create table t1(id int);(成功)
select * from mysql.db limit 5\G
delete from mysql.db;
flush privileges;
#mysql -uroot -p123456--socket=/tmp/mysql.sock
show databases;(看不到test库了)
```
### 源码安装
1、下载MySQL源码包

2、安装必要包（make cmake bison-devel ncurses-devel build-essential）
```
#tar –zxf mysql-5.6.25.tar.gz

#aptitude install make cmake bison libbison-dev ncurses ncurses-dev build-essential
```
3、Cmake 配置mysql编译选项，可定制需要安装的功能，编译选项参考：

http://dev.mysql.com/doc/refman/5.6/en/source-configuration-options.html

```
#cd mysql-5.6.25
#cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/usr/localmysql5.6/data -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLACTION=utf8_general_ci -DMYSQL_TCP_PORT=4000 -DMYSQL_UNIX_ADDR=/tmp/mysqld.sock -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_ARCHIVE_STORAGE_ENGINE=1 -DWITH_BLACKHOLE_STORAGE_ENGINE=1 -DWITH_PERFSCHEMA_STORAGE_ENGINE=1 -DWITH_PARTITION_STORAGE_ENGINE=1

```
4、make && make install
```
#make && make install
#cd /usr/local/mysql
#ll -h
```
5、初始化实例、编辑配置文件并启动

同二进制安装

6、账户安全设置

同二进制安装

### MySQL升级
以5.5升级5.6为例
1、下载MySQL5.6安装包并配置MySQL5.6的安装目录
2、关闭5.5的实例，更改部分参数，使用5.6启动
```
#ps -ef|grep mysqld
mysql>status
#cd /usr/local/mysql56

//关闭
#mysqladmin -uroot --socket=XXX shutdown
//修改参数
#vim my.cnf(把55目录都改成56的)
//启动56
#/usr/local/mysql56/bin/mysqld_safe --defaults-file=(参数文件) &
```
3、执行MySQL5.6路径下的mysql_upgrade脚本

-s 之升级系统权限表，不升级数据。
```
mysql>status
#/usr/local/mysql56/bin/mysql_upgrade -uroot --socket=XXXX -s 
```
4、验证是否成功升级
### 多实例安装
1、部署好MySQL软件
2、编辑多个配置文件，初始化多个实例
3、启动MySQL实例
为何多部署？
充分利用系统资源
资源隔离
业务、模块隔离

多实例安装
```
#mkdir -p /mysqldata/node3
#/usr/local/mysql56/script/mysql_install_db --user=mysql --basedir=/usr/local/mysql56 --datadir=/mysqldata/node3
#cd /mysqldata
#ll -h
#cp my2.cnf my3.cnf
#vim my.cnf(修改相关参数)
#/usr/local/mysql56/bin/mysqld_safe --defaults-file=/mysqldata/my3.cnf &
#mysql -uroot --socket=/mysqldata/node3/mysqld.sock
status
delete from msyql.user where user='';
delete from mysql.user where host<>'localhost';
set password for root@'localhost'=password('123456');
delete from mysql.db;
drop database test;
flush privileges;
```
## 主从复制
![enter description here][1]
### 主从复制用途
实时灾备，用于故障切换

读写分离，提供查询服务

备份，避免影响业务
### 部署
#### 必要条件：
- 主库开启binlog日志（设置log-bin参数）
- 主从server-id不同
- 从库服务器能联通主库
主从部署步骤：
1、备份还原---主库全备，备库恢复
```
mysqldump -uroot -p123456 --socket=/data/mysql/node1/mysqld.sock --single-transaction -A --master-data=1 > all_db.sql
mysql -utest -ptest -h(从库IP) -P3306 
mysql>source all_db.sql;
```
2、授权----主库授权用户
```
grant replication slave on *.* to repl@'(从库IP)' identified by 'repl';
```
3、配置复制，并启动
```
less all_db.sql|grep "change master to"
change master to master_host='(主库IP)',master_user='repl',master_password='repl',master_log_file='XXX',master_log_pos=XXX;
start stave;
show slave status\G
```
4、查看主从复制信息---复制检测
```
主库：
use db1;
insert into t1 values(10);
从库：
use db1;
select * from t1;(获得数据)
主库：
drop database db2;
从库：
show databases;(显示db2被删除)
```
### 原理：

![enter description here][2]
1、当在从库上执行drop database db2;语句后，

2、从库生成两个线程：SQL Thread和I/O Thread

3、I/O线程与主库连接，请求主库binlog

4、主库接收到从库请求，生成log dump Thread ，并将主库的binlog dump给从库的I/O Thread

5、从库的I/O Thread将接收到存储在Relay log（ 中继日志）中。

6、从库的SQL Thread解析这个 中继日志 ，并在从库上进行运用，如此两者都执行了该命令。
### 主从复制存在的问题
#### 存在的问题：
- 主库宕机后，数据可能丢失(从库)
- 从库只有一个sql Thread，主库写压力大，复制可能延时
#### 解决办法：
- 半同步复制--可有效解决数据丢失问题
- 并行复制----可让从库启动更多个thread去执行binlog，这样主是多线程写，从是多线程复制。
### MySQL semi-sync(半同步复制)
#### 半同步复制：
-  5.5集成到MySQL，以插件的形式存在，需要单独安装
-  确保事务提交后binlog至少传输到一个从库
-  不保证从库应用完成这个事务的binlog
-  性能有一定的降低，响应时间会更长
-  网络异常或从库宕机，卡主主库，直接超时或从从库恢复正常。


若业务比较重要，推荐，若不太重要，可以不部署，使用原生的主从复制。若担心性能，可提高配置。

#### 异步复制与半同步复制逻辑
![enter description here][3]

![enter description here][4]

#### 配置MySQL半同步复制
演示安装semisync
只需一次
1、主库安装插件
```
#查看已包含插件
show plugins;
#安装命令
install PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so'; 
```
2、从库安装插件
```
show plugins;
INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';
```
3、动态设置：

a、主库：
```
show variables like '%semi%';
SET GLOBAL rpl_semi_sync_master_enabled=1;
#master 延迟切异步--过长，卡住时可能引起雪崩效应，所以建议设短一些
SET GLOBAL rpl_semi_sync_master_timeout=N; 
```
b、从库
```
SET GLOBAL rpl_semi_sync_slave_enabled=1;
```

4、重启主从复制
```
从库：
stop slave;
start slave;
```
5、状态检查
```
show global status like '%semi%';
```
6、复制检查
```
主库：
use db1;
insert into t1 values(100);
从库：
use db1;
select * from t1;(获得数据)
```
7、测试延迟
```
从库：
stop slave;
主库：
use db1;
insert into t1 values(1);(被卡10s)
set global rpl_semi_sync_master_timeout=1000;
从库：
start slave;
stop slave;
主库：
use db1;
insert into t1 values(88);(被卡1s)
```
### 配置MySQL并行复制
- 社区版5.6新增
- 并行是指从库多线程apply binlog
- 库级别并行应用binlog，同一数据库更改还是串行的（5.7版并行复制基于事务组）
 设置：

```
set globl slave_parallel_workers = 10;设置sql线程数为10
```

#### MySQL并行复制实例：

```
从库：
show variable like '%slave_par%'
set global slave_parallel_workers=10; 设置sql线程数为10
stop slave;
start slave;
show processlist;(十个worker线程)
```

### 部分数据复制
主库添加参数
```
binlog_do_db = db1
binlog_ignore_db = db1
binlog_ignore_db = db2
```
 或从库添加
```
replicate_do_db = db1;
replicate_ignore_db = db1;
replicate_do_table = db1.t1
replicate_ignore_table = db1.t1
replicate_wild_do_table = db%.%
replicate_wild_ignore_table = db1.%
```

#### 实例：
```
1)主库：
create database db2;
2)从库部分复制配置
#vim /data/mysql/my1.cnf(replicate_do_db=db2)
#mysqladmin -uroot --socket=XXX --port=3306 -p123456 shutdown
#/usr/local/mysql/bin/mysqld_safe --defaults-file=/data/mysql/my1.cnf &
show slave status;
3)测试
主库：
use db1;
delete from t1;
从库：
use db1;
select * from t1;(任然保留数据)
主库：
use db2;
create table user(a int,b int);
从库：
use db2;
show tables;(查看到user表)
```
### 联级复制
- A->B->C
- B中添加参数：
- log_slave_updates
- B将把A的binlog记录到自己的binlog日志中


#### 实例
```
1)从库：
#vim /data/mysql/my1.cnf(log_slave_updates)
#mysqladmin -uroot --socket=XXX --port=3306 -p123456 shutdown
#/usr/local/mysql/bin/mysqld_safe --defaults-file=/data/mysql/my1.cnf &
2)创建新实例
在主库服务器上创建一个从库2实例
#mysqladmin -uroot --socket=XXX --port=3306 -p123456 shutdown
#kill -9 (mysqld_safe进程号)
#cp -r node1 node2
#vim my.cnf(修改相关参数,端口3307)
#chown -R mysql.mysql node2
#/usr/local/mysql56/bin/mysqld_safe --defaults-file=/data/mysql/my1.cnf &
#/usr/local/mysql56/bin/mysqld_safe --defaults-file=/data/mysql/my2.cnf &
#mysqldump -utest -ptest -hXXX -P3306 -A --master-data=1 > d731.sql(dump从库1的全备)
#mysql -uroot --socket=/data/mysql/node2/mysqld.sock -p123456 < d731.sql
3)配置从1和从2的主从
从1授权：
grant replication slave on *.* to repl@'(从2IP)' identified by 'repl';
从2配置复制：
less d731.sql|grep "change master to"
change master to master_host='(从1IP)',master_user='repl',master_password='repl',master_log_file='XXX',master_log_pos=XXX;
start stave;
show slave status\G
show processlist;
4)联级复制测试
主库：
create database db3;
从1：
show databases;(获得新建库)
从2：
show databases;(获得新建库)
```
### 监控主从复制
```
show slave status\G
```

### 复制出错处理
常见：1062（主键冲突）1032（记录不存在）
解决：手动处理
或：
跳过复制出错：
```
set global sql_slave_skip_counter=1;
```
#### 实例:
数据不一致问题
从库t1有数据，主库t1无数据情况下
主库：insert into t1 values(1);
从库：show slave status\G(1062错误)
[处理]
a.从库删除记录：
```
delete from t1 where id=1;
start slave;
```
b.从库跳过错误：
```
show variables like '%counter%';
set global sql_slave_skip_counter=1;
stop slave;
start slave;
```
### 主从复制课程小结
- MySQL主从复制是MySQL高可用性、高性能（负载均衡）的基础
- 简单、灵活，部署方式多样，可以根据不同业务场景部署不同的复制结构
- MySQL主从复制目前也存在一些问题，可以根据需要部署复制增强功能来解决问题
- 复制过程中应该时刻监控复制状态，复制出错或延时可能给系统造成影响
- MySQL复制是MySQL数据库工程师必知必会的一项基本技能

## MySQL日常运维
### DBA运维工作
1、日常：
- 导数据、数据修改、表结构变更
- 加权限、问题处理
2、其它：
数据库选型部署、设计、监控、备份、优化等
### 导数据及注意事项
1、数据最终形式（csv、sql文本还是直接导入某库中）

2、导数据方法（mysqldump、select into outfile）

3、导数据注意事项
- 导出为csv格式需要file权限，并且只能数据库本地导
- 避免锁库锁表（mysqldump使用--single-transaction选项不锁表）
- 避免对业务造成影响，尽量在镜像库做

#### 实例：
```
use db1;
select count(*) from t1;(先看一下数据量)
1)mysqldump导出
#mysqldump -uroot -p123456 --single-transaction --socket=/mysqldata/node3/mysqld.sock db1 t1 > /tmp/t1.sql
grant select on *.* to netease@'localhost' identified by '163';
#mysqldump -unetease -p163 --socket=/mysqldata/node3/mysqld.sock db1 t1 > /tmp/t2.sql(报错，没有锁表权限)
#mysqldump -unetease -p163 --single-transaction --socket=/mysqldata/node3/mysqld.sock db1 t1 > /tmp/t2.sql(成功)
#mysqldump -uroot -p123456 --single-transaction --socket=/mysqldata/node3/mysqld.sock db1 t1 -T /tmp
use db1;
2)以file权限into outfile导出数据
select * from t1 into outfile '/tmp/t1_2.txt';
select t1.c,t3,b from t1.id=t3.id into outfile '/tmp/t13.txt';
```
### 数据修改及注意事项
1、修改前切记做好备份
2、开事务做，修改完检查好了再提交。
3、避免一次修改大量数据，可以分批修改
4、避免业务高峰期做

实例
```
1.修改t1表中id<5的数据
1)备份
select * from t1 where id<5 into outfile '/tmp/t1id5.txt';
2)执行修改
begin;
update t1 set b=100,c=100 where id<5;
select * from t1 where id<5;
rollback;/commit;
2.表结构变更
1)5.5版本：
alter table t55 add c1 int;
delete from t55 where id<100;(卡住一会后才执行)
2)5.6版本：
use db1;
alter table t55 add c1 int;
delete from t55 where id<100;(执行顺畅)
alter table t1 modify c1 varchar(90);
delete from t55 where id<100;(卡住一会后才执行)
3)pt-online-schema-change工具
./pt-online-schema-change --user=root --password=123456 --host=localhost --socket=/mysqldata/node4/mysqld.sock D=db55,t=t55 --alter "add c2 int" --print --dry-run
3.加权限
grant select,insert,delete on *.* to netease@'localhost' identified by '163';
#mysql -unetease -p163 --socet=/mysqldata/node3/mysqld.sock --port=4001(登陆成功)
grant select,insert,delete on *.* to netease@'localhost' identified by '123';
#mysql -unetease -p163 --socet=/mysqldata/node3/mysqld.sock --port=4001(登陆失败)
4../tcpstat --port 4001 -t 1 -n 0
```
## MySQL参数调优
### 为什么要调整参数：
- 不同服务器之间的配置、性能不一样
- 不同业务场景对数据的需求不一样
- MySQL的默认参数只是个参考值，并不适合所有业务场景
### 优化之前需要知道什么：
- 服务器相关的配置
- 业务相关的情况
- MySQL相关配置
服务器上需要关注：
- 硬件情况
- 操作系统版本
- CPU、网卡节点模式
- 服务器numa设置
- RAID卡缓存

### 磁盘调度策略

- Write Back：数据写入cache既返回，数据异步的从cache刷入存储介质

- Write Through：数据同时写入cache和存储介质才返回写入成功。

#### Write Back VS Write Through
Write Back 性能优于Write Through
Write Through 比 Write Back 安全性高

### RAID
RAID Redundant Array of Independent Disks
- 生产环境中一般不会用裸设备，通常会使用RAID卡对一块盘或多块盘做RAID
- RAID卡会预留一块内存，来保证数据高效的存储与读取
- 常见的RAID类型有：RAID1、RAID0、RAID10和RAID5
#### RAID如何保证数据安全
**BBU(Backup Battery Unit)**
BBU保证在WEB策略下，即使服务器发生掉电或宕机，也能够将缓存中的数据写入磁盘，从而保证数据的安全。

### MySQL有哪些注意事项
MySQL的部署安装
MySQL的监控
MySQL参数调优

####  部署MySQL的要求
- 推荐MySQL版本：>=MySQL5.5
- 推荐的MySQL引擎：InnoDB
#### 系统调优的依据：监控
- 实时监控MySQL的Slow Log
- 实时监控数据库服务器的负载情况
- 实时监控MySQL内部状态
#### 通常关注的MySQL Status
Com_Select/Update/Delete/Insert：mysql查询更新删除插入的次数，查看数据库的请求是否变多
Bytes_received/Bytes_sent：接收和发送字节数，推测MySQL的吞吐量
Buffer Pool Hit Rate：Buffer池的命中率，InnoDB引擎中维护中自己的一个Buffer池，Buffer池命中率直接决定InnoDB内部的性能的一个关键，命中率低肯定是性能出现问题。
Threads_connected/Threads_created/Threads_running：连接相关的状态，当前已经连到的/历史已经创建的/当前活跃的连接数。前两个特别多，可推出应用程序是不是没有使用连接池以及连接池配置是否合适。第三个特别多时，数据库很忙，可能会被人恶意的攻击或做活动一些原因。
### MySQL参数调优
#### 读优化
- 合理利用索引对MySQL查询性能至关重要
- 适当的调整MySQL参数也能提升查询性能

#### innodb_buffer_pool_size 
- InnoDB缓冲池大小
- InnoDB数据引擎自己维护一块内存区域完成新老数据的替换
内存越大越能缓存更多的数据
#### innodb_thread_concurrency 
- 保护参数。
- InnoDB内部并发控制参数，设置为0代表不做控制。
- 如果并发请求较多，参数设置较小，后进来的请求将会排队，推荐较新的版本中关掉。
### 写优化
- 表结构设计上使用自增字段作为表的主键
- 只对合适的字段家索引，索引太多影响写入性能
- 监控服务器磁盘IO情况，如果写延迟较大则需要扩容
- 选择正确的MySQL版本，设置合理参数。

#### 有助于提高写入性能的参数
**1、innodb_flush_log_at_trx_commit && sync_hinlog**
主要影响MySQL写性能的两个参数
**a)、innodb_flush_log_at_trx_commit**
控制InnoDB事务的刷新方式，一共有三个值：0，1，2

- N=0 每隔一秒，把事务日志缓存区的数据写到日志文件中，以及把日志文件的数据刷新到磁盘上。（高效，但不安全）
- N=1 每个事务提交时，把事务日志从缓存去写到日志文件中，并刷新日志文件的数据到磁盘上，优先使用此模式保障数据安全性。（低效，非常安全）
- N=2 每个事务提交时，把事务日志数据从缓存区写到日志文件中；没个一秒，刷新一次日志文件，但不一定刷新到磁盘上，而是取决于操作系统的调度。（高效，但不安全，比n=0稍微安全一点点）

**b)、sync_binlog**
控制每次写入Binlog，是否都需要进行一次持久化

##### 如何保证事务安全
innodb_flush_log_at_trx_commit和sync_binlog都设为1

保证是事务要和Binlog保证一致性

![enter description here][5]

**串行有哪些问题**

SAS盘一般每秒只能有150-200个Fsync.

换算到数据库每秒只能执行50-60个事务.

##### 社区和官方的改进
- MariaDB提出改进，即使这两个参数都是1也能做到合并效果，性能得到了大幅提高。
- 官方吸收了MariaDB的思想，并在此基础上进行了改进，性能再次得到了提高。
- 
Tips:
- 官方在MySQL5.6版本之后才做了这个优化。
- 
- Percona和MariaDB版本在MySQL5.5已经包含了这个优化。

**2、InnoDB Redo log**
innodb_log_file_size 控制日志文件大小

Write ahead Log

![enter description here][6]

##### InnoDB Redo log作用
Redo log 用在数据库崩溃后的故障恢复
##### Redo log问题
- 如果写入频繁导致Redo log里对应的最老的数据脏页还没刷新到磁盘，此时数据库将卡主，强制刷新脏页到磁盘
- MySQL默认配置两个文件才10M，非常容易写满，生产环境中应适当调整大小



**3、innodb_io_capacity**
- 控制io能力的参数
- InnoDB每次刷新多少个脏页，决定了InnoDB存储引擎的吞吐能力。
- 在SSD等高级性能存储介质下，应该提高改参数以提高数据库的性能。
 
**4、innodb_inster_buffer**

仅MySQL中存在

顺序读写 VS 随机读写

![enter description here][7]

随机请求性能远小于顺序请求。

将尽可能多的随机请求合并为顺序请求才是提高数据库性能的关键。

MySQL从5.1版本开始支持Insert Buffer

MySQL5.5版本之后同时支持Update和Delete的Merge

Insert Buffer只对二级索引且非唯一索引有效。
### 小结
服务器配置要合理（内核版本、磁盘调度策略、RAID卡缓存）

完善的监控系统，提前发现问题

数据库版本要跟上，不要太新，也不要太老

数据库性能优化：

查询优化：索引优化为主，参数优化为辅。

写入优化：业务优化为主，参数优化为辅。

  [1]: https://assets.windcoder.com/xiaoshujiang/mysql_study_master_slave01.png "mysql_study_master_slave01"
  [2]: https://assets.windcoder.com/xiaoshujiang/mysql_study_master_slave05.png "mysql_study_master_slave05"
  [3]: https://assets.windcoder.com/xiaoshujiang/mysql_study_master_slave06.png  "mysql_study_master_slave06"
  [4]: https://assets.windcoder.com/xiaoshujiang/mysql_study_master_slave07.png "mysql_study_master_slave07"
  [5]: https://assets.windcoder.com/xiaoshujiang/mysql_study_youhua_xietijiao01.png "mysql_study_youhua_xietijiao01"
  [6]: https://assets.windcoder.com/xiaoshujiang/mysql_study_youhua_xietijiao02.png "mysql_study_youhua_xietijiao02"
  [7]: https://assets.windcoder.com/xiaoshujiang/mysql_study_youhua_xietijiao03.png "mysql_study_youhua_xietijiao03"