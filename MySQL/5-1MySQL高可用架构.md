---
title: 5-1MySQL高可用架构
tags: MySQL
grammar_cjkRuby: true
---
## 高可用
应用提供持续不间断（可用）的服务的能力
系统高可用性的平价通常用可用率表示

|可用率目标|全年不可用时间
| --- |---|
|99%|87.6小时|
|99.9%|8.7小时|
|99。99%|52分钟|
|99.999%|5分钟|

### 造成不可用的原因
- 硬件故障（各种）
- 预期中的系统软硬件维护
- 软件缺陷（应用代码，服务器程序都可能存在bug..）
- 攻击，泄露，人为失误...等安全事件
- 对于系统来说，不可用时间是各关键组件不可用时间的总和。

## 提高可用性的主要手段
- 冗余，RedunDancy
- 关键软硬件通过备份冗余避免故障时长时间的不可用
- 数据软件，硬件，存储的数据，都需要通过冗余确保故障时可替换。

## 数据库冗余与可用性
数据库服务在冗余实现上有其特殊性
数据：服务“有状态”与数据冗余
实现放回多种多样，同一种数据也会有多种实现方案
可用性目标循序渐进：
任何故障都不会造成数据丢失→可以较快速恢复服务（高可用）

## MySQL-基于共享存储的单活方案（比较扯）

![enter description here][1]

非常可行，非常有效、非常稳定
共享存储一般由比较靠谱的几家供应商提供，如IBM等大厂提供
但费用比较昂贵，而且两台主机的成本只有一台在提供服务。
从技术目标上非常稳健可靠。
## 基于存储复制的数据冗余单活（不常用）

![enter description here][2]

两台机器仅一台提供服务，存在一定浪费，且DRBD也不能做到百分之百的保障。

## 基于MySQL主从复制（常用，普适）

![enter description here][3]

主库读写，从库可用于部分只读，另一从库只做备份或一些离线计算。

### 需要改进的问题
- 主从服务器各自有IP地址，发生主从切换后应用需要修改重启
- 人工判断主库是否故障在发起切换需要花较多时间
- 主从复制存在客观延迟，切换后可能造成事务数据丢失。

### 方案改进
- 为了避免应用人工修改切换IP，引入VIP漂移方案

 
 **方案一：使用虚拟IP** 。一旦主库有问题，将虚拟IP绑定从库。一旦漂移成功，程序基本不需要做什么修改。



![enter description here][4]

![enter description here][5]


**方案二：使用域名解析**。主库出问题，将域名解析到从库。

- 为了减少人工接入处理的时间开销引入自动探活处理机制。

![enter description here][6]

#### 高可用中间层与RDS

- VIP解决应用切换问题
- 监控和管理服务器解决自动判断故障切换和VIP漂移
- VIP管理+探活+主从关系切换 = 高可用中间层
- 云环境 + 高可用中间层 + 底层数据库 = 一种PaaS = 基本RDS

#### 高可用中间层

比较保守的两种。

##### MHA
- 自动选择复制延迟最小的从节点并试图补全日志（但大部分主机故障情况下行不通）
- 通常要求两从以上，会进行主从关系切换
- 不提供VIP管理方案。--需要自己通过

线上项目推荐方案。

##### MMM
- 提供了基本的VIP管理功能
- 适合双主配置的一对主机，不会主动主从关系
- 不支持主从数据延迟判断和补全
#### MySQL主从复制延迟
日志传输延迟

数据库接收一个commit请求后，主库日志持久化到redo log中，redo log 一旦持久化成功返回commit，告知用户持久化成功，不需要写进数据库文件中，只需写进主库日志中即可。数据从写入主库日志然后到传输到从库，有时间间隔。从主库写到从库中再到从库持久化完成又一个时间间隔。

若主库日志持久化完成到传输到从库期间发生问题，可能由于主库出问题没来得及传到从库从而导致主从不一致。

![enter description here][7]

#### MySQL半同步

主库接收到commit提交，主库完成持久化外需保证传到从库并在从库完成持久化才对主库返回commit成功。

当等待从库时，时间不可控。

![enter description here][8]

如，从库出问题

![enter description here][9]

有个关键参数，可以在等待n秒后，可将后面所有请求降级为异步，不保证从库的提交方式等。



### 基于集群提交通信协议的多主复制（一定场景使用）

![enter description here][10]

### 较完善的MySQL高可用方案

半同步复制+高可用中间层+VIP管理方案

如：
- 半同步复制+MHA+Keepalive
- 半同步复制+RDS



  [1]: https://assets.windcoder.com/xiaoshujiang/mysql_study_gaokeyung01.png "mysql_study_gaokeyung01"
  [2]: https://assets.windcoder.com/xiaoshujiang/mysql_study_gaokeyung02.png "mysql_study_gaokeyung02"
  [3]: https://assets.windcoder.com/xiaoshujiang/mysql_study_gaokeyung03.png "mysql_study_gaokeyung03"
  [4]: https://assets.windcoder.com/xiaoshujiang/mysql_study_gaokeyung05.png "mysql_study_gaokeyung05"
  [5]: https://assets.windcoder.com/xiaoshujiang/mysql_study_gaokeyung06.png "mysql_study_gaokeyung06"
  [6]: https://assets.windcoder.com/xiaoshujiang/mysql_study_gaokeyung08.png "mysql_study_gaokeyung08"
  [7]: https://assets.windcoder.com/xiaoshujiang/mysql_study_gaokeyung09.png "mysql_study_gaokeyung09"
  [8]: https://assets.windcoder.com/xiaoshujiang/mysql_study_gaokeyung010.png "mysql_study_gaokeyung010"
  [9]: https://assets.windcoder.com/xiaoshujiang/mysql_study_gaokeyung011.png "mysql_study_gaokeyung011"
  [10]: https://assets.windcoder.com/xiaoshujiang/mysql_study_gaokeyung04.png "mysql_study_gaokeyung04"