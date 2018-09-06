---
title: ELK日志管理系统整合
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

## 简介
![enter description here](./images/elk.jpg)
Filebeat轻量级的日志传输工具，可以读取系统、nignx、apache等logs文件，监控日志文件，传输数据到Elasticsearch或者Logstash，最后在Kibana中实现可视化。

在这里要实现的功能为：
1. Filebeat从nignx读取日志文件，将过滤后的数据传给Logstash。
2. Logstash收集到Filebeat传来的数据后格式化输出到 Elasticsearch。
3. 最后再由Kibana 访问Elasticsearch提供的比较友好的 Web 界面进行汇总、分析、搜索。