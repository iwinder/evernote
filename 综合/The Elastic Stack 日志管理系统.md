---
title: The Elastic Stack 日志管理系统
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

https://www.elastic.co/cn/downloads

```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.4.0.tar.gz
wget https://artifacts.elastic.co/downloads/logstash/logstash-6.4.0.tar.gz
wget https://artifacts.elastic.co/downloads/kibana/kibana-6.4.0-linux-x86_64.tar.gz
wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-6.4.0-linux-x86_64.tar.gz
```
 5.0级以上版本至少需要JDK1.8.
 2.4的版本可在1.7

|ES  | JDK|
|----| --- |
| 0.90 | 1.6 |
| 1.3-2.4 | 1.7|
| 5.0+ | 1.8 |

```
wget https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.4.6/elasticsearch-2.4.6.tar.gz

https://download.elastic.co/kibana/kibana/kibana-4.6.6-linux-x86_64.tar.gz

https://www.elastic.co/downloads/past-releases/logstash-2-4-1

https://download.elastic.co/beats/filebeat/filebeat-1.3.1-x86_64.tar.gz
```




java.lang.UnsupportedOperationException: seccomp unavailable: CONFIG_SECCOMP not compiled into kernel, CONFIG_SECCOMP and CONFIG_SECCOMP_FILTER are needed

内核版本过低。[ElasticSearch 5.0.0 安装部署常见错误或问题](https://blog.csdn.net/u012246178/article/details/63253531)


[错误：无法找到或加载主类org.elasticsearch.tools.JavaVersionChecker](https://discuss.elastic.co/t/error-could-not-find-or-load-main-class-org-elasticsearch-tools-javaversionchecker/82213)

yum -y install sudo

[linux下elasticsearch 安装、配置及示例](https://blog.csdn.net/sinat_28224453/article/details/51134978)

[Elasticsearch unable to start - Permission issue](https://stackoverflow.com/questions/41057917/elasticsearch-unable-to-start-permission-issue)
---  No factory method found for class org.apache.logging.log4j.core.appender.RollingFileAppender ----
[elasticsearch](https://blog.csdn.net/qq_26712449/article/details/73346474)

