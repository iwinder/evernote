---
title: 5-2MySQL读写分离
tags: MySQL
grammar_cjkRuby: true
description: 网易DBA微专业笔记
---

## 读写分离-what

![enter description here][1]

读操作（select） 访问从库

写（事务）操作（DML DDL DCL）访问主库

在**高并发查询场景**下，为满足应用访问需求，通常都会部署多个从库提供查询服务以**提升数据库查询的扩展性**，将查询分发到从库，让从库**分担主库查询负载**的技术我们通常称为读写分离。

## 读写分离-Why

- 提升数据库查询的扩展性
- 提升资源利用

## 读写分离-How

- 应用端读写分离
- 服务器端读写分离 	
## 读写分离主要问题
即缺点：
- 1、数据可能不一致
- 2、部署、实现相对复杂



## 读写分离方案
常用方案：
- MySQL Proxy
- Amoeba For MySQL
- MySQL Router
- 应用端实现

## MySQL Proxy
MySQL Proxy是MySQL官方开发死亡一款读写分离中间件，开发多年一直未GA,在功能 上存在许多缺陷，目前官方已经放弃开发。

- 使用C/C++开发服务模块
- 使用lua脚本语言分析sql语句进行读写分离。

### MySQL Proxy(Atlas)
Atlas是360公司开发改进的一款MySQL Proxy的分支，基于MySQL Proxy0.8.2，修正大量Bug，并添加许多使用功能。已经大规模运行于360公司的各种业务中，经过线上大并发业务考验，也有许多其他公司使用，肯拉着与使用者比较活跃。

#### Fix:
- Lua 改为C语言，提高性能
- 并发下字符集修正
- 避免死等待
- 主库宕机仍可以读
- MySQL新版本兼容等

#### Add:
- 从库动态上下线
- 分库分表
- 从库指定负载权重
- IP过滤等

## Amoeba For MySQL
Amoeba是类似MySQL Proxy的中间件，采用Java语言编写，具有读写分离、数据切分和过滤等功能，是非常早的一款国产MySQL读写分离软件。

- 支持读写分离
- 支持数据拆分
- 对事务支持不好
- 数据拆分比较简单
基本属于停滞开发状态，不推荐。

## MySQL Router
MySQL Router是MySQL官方开发的一款轻量级的用来实现高可用和扩展性的中间件，支持读写分离负载均衡，支持根MySQL Fabric一起使用，刚GA，稳定性等还有待考验。

![enter description here][2]

MySQL Router提供2个端口，读写与只读端口。

- 读写端口：first available
- 只读端口：round robin

更像高可用工具，读写分离部分本身比较鸡肋，与MySQL Fabric一起使用契合度高。

### MySQL Proxy(Atlas)部署

#### 准备：
- 1、虚拟机
- 2、Atlas:https://github.com/Qihoo360/Atlas/releases 

#### 操作：
- 1、sudo dpkg -i  Atlas-2.2-debian7.0-x86_64.deb
- 2、编辑配置文件
- 3、/usr/local/mysql-proxy/bin/mysql-proxyd instancename start
- 
注：看到新版中已经没有deb包，可前往官方[Atlas的安装](https://github.com/Qihoo360/Atlas/wiki/Atlas%E7%9A%84%E5%AE%89%E8%A3%85)查看新版方式。
安装

  [1]: https://assets.windcoder.com/xiaoshujiang/mysql_study_duxiefenli01.png "mysql_study_duxiefenli01"
  [2]: https://assets.windcoder.com/xiaoshujiang/mysql_study_duxiefenli02.png "mysql_study_duxiefenli02"