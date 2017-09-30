---
title: 5-2MySQL读写分离
tags: MySQL
grammar_cjkRuby: true
description: 网易DBA微专业笔记
---

读写分离-what

![enter description here][1]

读操作（select） 访问从库

写（事务）操作（DML DDL DCL）访问主库

在**高并发查询场景**下，为满足应用访问需求，通常都会部署多个从库提供查询服务以**提升数据库查询的扩展性**，将查询分发到从库，让从库**分担主库查询负载**的技术我们通常称为读写分离。

读写分离-how


  [1]: https://assets.windcoder.com/xiaoshujiang/mysql_study_duxiefenli01.png "mysql_study_duxiefenli01"