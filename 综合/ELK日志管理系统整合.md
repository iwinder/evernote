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

## 准备工作
### 基础环境检查
```

[parim@dev ~]# cat /etc/hosts

127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1       localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.0.76	ut-train-cache mongodb

192.168.0.76	dev.qc.net db-master

192.168.0.79	sk.qc.net

```
### 软件包
wget获取所需软件包，这里默认nignx以安装配置。最新的软件包可从[官网](https://www.elastic.co/cn/downloads) 下载。这里使用的tar.gz压缩格式的安装包。

```

[parim@dev ~]# mkdir elk
[parim@dev ~]# cd elk/
[parim@dev elk]# wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.4.0.tar.gz
[parim@dev elk]# wget https://artifacts.elastic.co/downloads/logstash/logstash-6.4.0.tar.gz
[parim@dev elk]# wget https://artifacts.elastic.co/downloads/kibana/kibana-6.4.0-linux-x86_64.tar.gz
[parim@dev elk]# wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-6.4.0-linux-x86_64.tar.gz

```
### 检测
检测安装包下载情况
```

[parim@dev elk]# ls
elasticsearch-6.4.0.tar.gz filebeat-6.4.0-linux-x86_64.tar.gz kibana-6.4.0-linux-x86_64.tar.gz logstash-6.4.0.tar.gz

```

服务器只需要安装Elasticsearch、Logstash、Kibana, 客户端只需要安装filebeat。

### JDK检测
elasticsearch、Logstash均需要jdk支持，故若服务器上没有，需要先安装JDK。5.x级以上版本均需要jdk1.8的支持。客户端上使用的是filebeat软件，它不依赖java环境，所以不需要安装。

## 安装Elasticsearch
### 解压
```

[parim@dev elk]# tar -zxvf elasticsearch-6.4.0.tar.gz
[parim@dev elk]# cd elasticsearch-6.4.0

```
### 配置

Elasticsearch有三个配置文件：
- elasticsearch.yml 用于配置Elasticsearch
- jvm.options 用于配置Elasticsearch JVM设置
- log4j2.properties 用于配置Elasticsearch日志记录

这些文件位于config目录中，此处只配置 elasticsearch.yml
```

[parim@dev elasticsearch-6.4.0]# vi config/elasticsearch.yml

```
配置Elasticsearch节点的host、name、port，具体参数请根据实际情况自行修改。
```

node.name: sk.qc.net
network.host: 192.168.0.79
http.port: 9200

```
### 启动

[Starting Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/starting-elasticsearch.html#start-targz) 

#### 命令行启动

```

[parim@dev elasticsearch-6.4.0]# ./bin/elasticsearch

```
此时是在窗口中启动的，可以通过Ctrl+C停止。

#### 作为守护进程启动

要将Elasticsearch作为守护进程运行，请在命令行中指定-d，并使用-p选项将进程ID记录在文件中：
```

[parim@dev elasticsearch-6.4.0]# ./bin/elasticsearch -d -p pid

```
日志可在```$ES_HOME/logs/```文件夹中查看。

为了结束Elasticsearch，此时需要kill这个守护进程id.
```
netstat -nltp | grep java

kill -9 PID

```

> [RPM](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.4.0.deb)和[Debian](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.4.0.deb) 的包中提供了启动脚本，你可以用它来启动和停止Elasticsearch 进程，其余几个同此。

## Kibana
### 解压
```

[parim@dev elk]# tar -zxvf kibana-6.4.0-linux-x86_64.tar.gz
[parim@dev elk]# cd kibana-6.4.0-linux-x86_64

```

### 配置
```
[parim@dev kibana-6.4.0-linux-x86_64]# vi config/kibana.yml

```

这里主要配置kibana的访问端口和Elasticsearch访问url

```
server.host: "0.0.0.0"
elasticsearch.url: "http://192.168.0.79:9200"

```
若elasticsearch有用户名和密码，也需在这里配置：
```
elasticsearch.username: "user"
elasticsearch.password: "pass"
```