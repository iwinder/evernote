---
title: ELK日志管理系统整合
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

## 1.简介
![enter description here](./images/elk.jpg)
Filebeat轻量级的日志传输工具，可以读取系统、nignx、apache等logs文件，监控日志文件，传输数据到Elasticsearch或者Logstash，最后在Kibana中实现可视化。

在这里要实现的功能为：
1. Filebeat从nignx读取日志文件，将过滤后的数据传给Logstash。
2. Logstash收集到Filebeat传来的数据后格式化输出到 Elasticsearch。
3. 最后再由Kibana 访问Elasticsearch提供的比较友好的 Web 界面进行汇总、分析、搜索。

## 2.准备工作
### 2.1 基础环境检查
```

[parim@dev ~]# cat /etc/hosts

127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1       localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.0.76	ut-train-cache mongodb

192.168.0.76	dev.qc.net db-master

192.168.0.79	sk.qc.net

```
### 2.2 软件包
wget获取所需软件包，这里默认nignx以安装配置。最新的软件包可从[官网](https://www.elastic.co/cn/downloads) 下载。这里使用的tar.gz压缩格式的安装包。

```

[parim@dev ~]# mkdir elk
[parim@dev ~]# cd elk/
[parim@dev elk]# wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.4.0.tar.gz
[parim@dev elk]# wget https://artifacts.elastic.co/downloads/logstash/logstash-6.4.0.tar.gz
[parim@dev elk]# wget https://artifacts.elastic.co/downloads/kibana/kibana-6.4.0-linux-x86_64.tar.gz
[parim@dev elk]# wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-6.4.0-linux-x86_64.tar.gz

```
### 2.3 检测
检测安装包下载情况
```

[parim@dev elk]# ls
elasticsearch-6.4.0.tar.gz filebeat-6.4.0-linux-x86_64.tar.gz kibana-6.4.0-linux-x86_64.tar.gz logstash-6.4.0.tar.gz

```

服务器只需要安装Elasticsearch、Logstash、Kibana, 客户端只需要安装filebeat。

### 2.4 JDK检测
elasticsearch、Logstash均需要jdk支持，故若服务器上没有，需要先安装JDK。5.x级以上版本均需要jdk1.8的支持。客户端上使用的是filebeat软件，它不依赖java环境，所以不需要安装。

## 3.安装Elasticsearch
### 3.1 解压
[Install Elasticsearch with .zip or .tar.gz](https://www.elastic.co/guide/en/elasticsearch/reference/current/zip-targz.html)
```

[parim@dev elk]# tar -zxvf elasticsearch-6.4.0.tar.gz
[parim@dev elk]# cd elasticsearch-6.4.0

```
### 3.2 配置

Elasticsearch有三个配置文件：
- elasticsearch.yml 用于配置Elasticsearch
- jvm.options 用于配置Elasticsearch JVM设置
- log4j2.properties 用于配置Elasticsearch日志记录

这些文件位于config目录中，此处只配置 elasticsearch.yml
```

[parim@dev elasticsearch-6.4.0]# vi config/elasticsearch.yml

```
配置Elasticsearch节点的host、name、port，具体参数请根据实际情况自行修改。Elasticsearch默认端口号为9200。
```

node.name: sk.qc.net
network.host: 192.168.0.79
http.port: 9200

```
### 3.3 启动

[Starting Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/starting-elasticsearch.html#start-targz) 

#### 3.3.1 命令行启动

```

[parim@dev elasticsearch-6.4.0]# ./bin/elasticsearch

```
默认情况下，elasticsearch在前台运行，并可以通过按Ctrl+C来停止。


#### 3.3.2 作为守护进程启动

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

## 4.安装Kibana
不支持运行Kibana和Elasticsearch的不同主要版本（例如Kibana 5.x和Elasticsearch 2.x），也不支持比Elasticsearch版本更新的Kibana次要版本（例如Kibana 5.1和Elasticsearch 5.0）。[Set Up Kibana](https://www.elastic.co/guide/en/kibana/6.4/setup.html)
### 4.1 解压
[ Install Kibana with .tar.gz](https://www.elastic.co/guide/en/kibana/6.4/targz.html)

```

[parim@dev elk]# tar -zxvf kibana-6.4.0-linux-x86_64.tar.gz
[parim@dev elk]# cd kibana-6.4.0-linux-x86_64

```

### 4.2 配置
```
[parim@dev kibana-6.4.0-linux-x86_64]# vi config/kibana.yml

```

这里主要配置kibana的访问端口、host和Elasticsearch访问url。kibana默认端口号为5601。

```
server.host: "0.0.0.0"
elasticsearch.url: "http://192.168.0.79:9200"

```
若elasticsearch有用户名和密码，也需在这里配置：
```
elasticsearch.username: "user"
elasticsearch.password: "pass"
```

### 4.3 启动

#### 4.3.1 命令行启动
```
[parim@dev kibana-6.4.0-linux-x86_64]# ./bin/kibana

```
默认情况下，Kibana在前台运行，将其日志打印到标准输出（stdout），并可以通过按Ctr+C来停止。

#### 4.3.2 后台运行
常常我们更希望这些在后台运行，通过以下代码即可：
```
[parim@dev kibana-6.4.0-linux-x86_64] nohup ./bin/kibana &

```

使用命令shell时，使用nohup为命令添加前缀可防止在注销或退出shell时自动中止命令。其意为"no hangup."。命令末尾的“＆”符号指示bash在后台运行。----[Linux nohup command](https://www.computerhope.com/unix/unohup.htm)

#### 4.3.3 检查服务运行
Kibana默认 进程名：node ，端口5601
```
[parim@dev kibana-6.4.0-linux-x86_64] netstat -nltp
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name   
tcp        0      0 0.0.0.0:5601                0.0.0.0:*                   LISTEN      17821/./bin/../node 
```

当需要结束时,需执行下面语句：
```
kill -9 PID

```
如上面```kill -9 17821```

### 4.4 对外开放tcp/5601

修改防火墙，放行端口。这里仅提供iptables的修改方式(下面的//注释请勿复制)，firewalld方式类似。
```
         /sbin/iptables -I INPUT -p tcp --dport 5601 -j ACCEPT   // 写入修改
 
         /etc/init.d/iptables save   // 保存修改
 
        service iptables restart    // 重启防火墙，修改生效
```
这时，我们可以打开浏览器，测试访问一下kibana服务器 http://192.168.0.79:5601/ 如下图：

![enter description here](./images/1536215894929.png)

## 5.安装Logstash
### 5.1 解压
```

[parim@dev elk]# tar -zxvf logstash-6.4.0.tar.gz
[parim@dev elk]# cd logstash-6.4.0

```
### 5.2 生成证书
```
[parim@dev elk]# cd /etc/pki/tls/
[parim@dev tls]# openssl req -subj '/CN=sk.qc.net/' -x509 -days 3650 -batch -nodes -newkey rsa:2048 -keyout private/logstash-forwarder.key -out 
certs/logstash-forwarder.crt
```

### 5.3 配置
此处是创建了一个新的配置文件01-logstash-initial.conf，其文件名可自定义。
```
[parim@dev logstash-6.4.0]# vi config/01-logstash-initial.conf

```
配置文件具体如下：
```
input {
  beats {
    port => 5000
    type => "logs"
    host => "0.0.0.0"
    ssl => true
    ssl_certificate => "/etc/pki/tls/certs/logstash-forwarder.crt"
    ssl_key => "/etc/pki/tls/private/logstash-forwarder.key"
  }
}

 filter {
  if [fileset][module] == "nginx" {
    if [fileset][name] == "access" {
      grok {
	match => { "message" =>  ["%{IPORHOST:[nginx][access][client_ip]} - %{DATA:[nginx][access][user_name]} %{DATA:[nginx][access][msec]} \[%{DATA:[nginx][access][time_iso8601]}\] sid:\"%{DATA:[nginx][access][cookie_sid]}\" \"%{WORD:[nginx][access][method]} %{DATA:[nginx][access][url]} HTTP/%{NUMBER:[nginx][access][http_version]}\" %{NUMBER:[nginx][access][response_code]} %{NUMBER:[nginx][access][body_sent][bytes]} ref:\"%{DATA:[nginx][access][referer]}\" \"%{DATA:[nginx][access][user_agent]}\" \"%{DATA:[nginx][access][x_forwarded]}\""]}
    remove_field => "message"     
    }    
    mutate {
        add_field => { "read_timestamp" => "%{@timestamp}" }
      }
      date {
        match => [ "[nginx][access][time]", "dd/MMM/YYYY:H:m:s Z" ]
        remove_field => "[nginx][access][time]"
      }


    }
  }
}

output {
  elasticsearch {
    hosts => ["192.168.0.79:9200"]
    manage_template => false
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
 }
}
```
### 5.4启动
[Running Logstash from the Command Line](https://www.elastic.co/guide/en/logstash/6.x/running-logstash-command-line.html)
#### 5.4.1测试启动
```
[parim@dev logstash-6.4.0]# ./bin/logstash  -t -f /home/parim/elk/logstash-6.4.0/config/01-logstash-initial.conf
```

默认情况下，logstash在前台运行，并可以通过按Ctrl+C来停止。

#### 5.4.2 后台运行

```
nohup ./bin/logstash   -f /home/parim/elk/logstash-6.4.0/config/01-logstash-initial.conf &
```

当需要结束时,需执行下面语句：
```
# 获取logstash的Pid
ps -ef |grep logstash

# 结束进程
kill -9 PID
```

## 6.部署filebeat

filebeat只需部署到客户端，这里客户端服务器为同一个。

### 6.1 解压
```

[parim@dev elk]# tar -zxvf filebeat-6.4.0-linux-x86_64.tar.gz
[parim@dev elk]# cd filebeat-6.4.0-linux-x86_64

```

### 6.2 配置

首先要配置的是filebeat.yml，此次首要任务是监控处理Nginx Logs，filebeat提供了[Nginx module](https://www.elastic.co/guide/en/beats/filebeat/6.4/filebeat-module-nginx.html)可以帮我们便捷完成此项，故需要配置Nginx module。

#### 6.2.1 filebeat.yml

```
[parim@dev filebeat-6.4.0-linux-x86_64]# vi  filebeat.yml
```
这里主要配置的有：
- Filebeat inputs 可以配置日志类型的，也可以配置日志过滤。这次只配置过滤条件。
- Filebeat modules 默认应该不需要配置，具体配置有单独文件。
- Elasticsearch output 由于这里有一层Logstash，需要注释这里的配置。
- Logstash output 这里需要放开注释，配置我们之前的Logstash相关内容。

下面放出关键配置：
```
#=========================== Filebeat inputs =============================

filebeat.inputs:
- type: log
    enabled: false
- type: stdin
    include_lines: [(\/learner)+]
#============================= Filebeat modules ===============================
filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: true
#================================ Outputs =====================================
#-------------------------- Elasticsearch output ------------------------------
这里面的全注释即可。
#----------------------------- Logstash output --------------------------------
output.logstash:
   hosts: ["sk.qc.net:5000"]
   ssl.certificate_authorities: ["/etc/pki/tls/certs/logstash-forwarder.crt"]
```
logstash中hosts的地址必须与上面生成证书里面的地址相同，不然会报证书的相关错误。

#### 6.2.2 Nginx module
该文件在filebeat-6.4.0-linux-x86_64/modules.d中nginx.yml.disabled

```
[parim@dev filebeat-6.4.0-linux-x86_64]# cd modules.d/
[parim@dev modules.d]# cp nginx.yml.disabled nginx.yml
[parim@dev modules.d]# vi nginx.yml
```
此处复制了一份新的并重命名为nginx.yml，filebeat便可读取到该文件。
NGINX本身在/home/parim/apps/nginx-1.10，故此处配置如下：
```
- module: nginx
  # Access logs
  access:
    enabled: true

    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    var.paths: [/home/parim/apps/nginx-1.10/logs/access.log*]

  # Error logs
  error:
    enabled: true

    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    var.paths: [/home/parim/apps/nginx-1.10/logs/error.log*]
```
之后退回filebeat-6.4.0-linux-x86_64,执行如下操作可开启nginx
```
[parim@dev filebeat-6.4.0-linux-x86_64]# filebeat modules enable nginx
```
### 6.3启动
#### 6.3.1 命令行
```
[parim@dev filebeat-6.4.0-linux-x86_64]# ./filebeat -e -c filebeat.yml -d "publish"
```
默认情况下，filebeat在前台运行，并可以通过按Ctrl+C来停止。

#### 6.3.2 后台运行
```
[parim@dev filebeat-6.4.0-linux-x86_64]# [parim@dev filebeat-6.4.0-linux-x86_64]#
```
若想结束，通过获取Pid，然后Kill的方式即可。
```
ps -ef |grep filebeat

kill -9 PID
```

以上4个软件配置好并启动后，可在浏览器中看到效果：
![enter description here](./images/1536223737905.png)

## 7.附录1-指定JDK
服务器上默认为非1.8的JDK时，使用5.x的ELK需要指定单独的1.8的JDK才可。
### 7.1 Elasticsearch
需要修改elasticsearch启动脚本。
打开 elasticsearchHOME/bin/elasticsearch 编辑：
```
[root]# vim bin/elasticsearch
export JAVA_HOME=/home/parim/spark/apps/jdk1.8.0_144/    （此处配置的为刚下的1.8的配置目录）
export PATH=$JAVA_HOME/bin:$PATH

if [ -x "$JAVA_HOME/bin/java" ]; then
        JAVA="/home/parim/spark/apps/jdk1.8.0_144//bin/java"
else
        JAVA=`which java`
fi
```
完整配置文件（部分）
```
#!/bin/bash

……

# 配置自己的jdk1.8
export JAVA_HOME=/home/parim/spark/apps/jdk1.8.0_144/
export PATH=$JAVA_HOME/bin:$PATH

source "`dirname "$0"`"/elasticsearch-env

ES_JVM_OPTIONS="$ES_PATH_CONF"/jvm.options
JVM_OPTIONS=`"$JAVA" -cp "$ES_CLASSPATH" org.elasticsearch.tools.launchers.JvmOptionsParser "$ES_JVM_OPTIONS"`
ES_JAVA_OPTS="${JVM_OPTIONS//\$\{ES_TMPDIR\}/$ES_TMPDIR} $ES_JAVA_OPTS"

# 自己添加的jdk判断
if [ -x "$JAVA_HOME/bin/java" ]; then
        JAVA="/home/parim/spark/apps/jdk1.8.0_144//bin/java"
else
        JAVA=`which java`
fi

cd "$ES_HOME"
# manual parsing to find out, if process should be detached
if ! echo $* | grep -E '(^-d |-d$| -d |--daemonize$|--daemonize )' > /dev/null; then
  exec \
    "$JAVA" \
    $ES_JAVA_OPTS \
    -Des.path.home="$ES_HOME" \
    -Des.path.conf="$ES_PATH_CONF" \
    -Des.distribution.flavor="$ES_DISTRIBUTION_FLAVOR" \
    -Des.distribution.type="$ES_DISTRIBUTION_TYPE" \
    -cp "$ES_CLASSPATH" \
    org.elasticsearch.bootstrap.Elasticsearch \
    "$@"
else

……

```

### 7.2 Logstash


查看logstash启动脚本,没有关于java_home之类的相关配置，但logstash启动过程会引入lib文件bin/logstash.lib.sh ，经查看logstash.lib.sh中定义了一个setup_java的函数，setup_java被setup函数调用，最终被bin/logstash启动脚本调用。

因此，我们只需要在logstash或logstash.lib.sh的行首位置添加两个环境变量即可：
```
export JAVA_CMD=/home/parim/spark/apps/jdk1.8.0_144
export JAVA_HOME=/home/parim/spark/apps/jdk1.8.0_144
```
## 8.附录2-报错与解决
### Elasticsearch
