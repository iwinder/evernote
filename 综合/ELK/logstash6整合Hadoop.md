---
title: logstash6整合Hadoop 
tags: Logstash,Hadoop 
grammar_cjkRuby: true
---

## 前提
本文是之前elk的后续，故默认已搭建好logstash等elk相关环境。侧重点是Hadoop安装以及其与logstash的Output插件的整合。

假设存在两台服务器并处于同一局域网中，分别是192.168.0.79和192.168.0.80。ELK系统已部署在192.168.0.79，Hadoop将部署于192.168.0.80。



