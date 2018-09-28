---
title: logstash6整合Hadoop 
tags: Logstash,Hadoop 
grammar_cjkRuby: true
---

## 前提
本文是之前elk的后续，故默认已搭建好logstash等elk相关环境。侧重点是Hadoop安装以及其与logstash的Output插件的整合。

假设存在两台服务器并处于同一局域网中，分别是192.168.0.79和192.168.0.80，系统均是CentOS7，且均已安装Java 8。ELK系统已部署在192.168.0.79，Hadoop将部署于192.168.0.80。

原则Hadoop集群需要ssh免密登陆配置，以便操作节点的启动与停止，这里暂不涉及此方面需求，故舍去。相关的可在 [官方文档-Hadoop: Setting up a Single Node Cluster.](https://hadoop.apache.org/docs/r2.8.5/hadoop-project-dist/hadoop-common/SingleCluster.html) 查看。

## 下载并解压Hadoop 
可从官方[ Apache Download Mirrors](http://www.apache.org/dyn/closer.cgi/hadoop/common/)获取链接下载。本文下载的为Hadoop-2.8.5：
```
wget http://mirrors.hust.edu.cn/apache/hadoop/common/hadoop-2.8.5/hadoop-2.8.5.tar.gz
```
完成后解压文件：

```
tar -zxvf hadoop-2.8.5.tar.gz
```
得到hadoop-2.8.5文件夹

## 配置
进入解压后的hadoop-2.8.5文件夹：
```
cd hadoop-2.8.5
```
### etc/hadoop/hadoop-env.sh
编辑etc/hadoop/hadoop-env.sh配置一些默认项，这里主要是设置 JAVA_HOME，即填写完整的 Java 安装路径。
```
vi etc/hadoop/hadoop-env.sh
```
注释掉原来的
```
# export JAVA_HOME=${JAVA_HOME}
```
192.168.0.80的Java安装目录为/home/parim/apps/jdk1.8.0_181，故添加：
```
export JAVA_HOME=/home/parim/apps/jdk1.8.0_181
```
![enter description here](./images/1538128064461.png)

若是需要搭建Local (Standalone) Mode，通过如下命令启动即可：
```
bin/hadoop
```
本文需要搭建Pseudo-Distributed Operation，故需要继续配置其他文件。
### etc/hadoop/core-site.xml
该文件中有两个参数需要设置：
1. fs.defaultFS - 默认文件系统的名称
2. hadoop.tmp.dir - 其他临时目录的根目录
打开core-site.xml文件：
```
vi etc/hadoop/core-site.xml
```
拷贝以下所有行的内容放入到标签 ```<configuration></configuration>``` 中间。
```
<property>
	<name>fs.defaultFS</name>
	<value>hdfs://192.168.0.80:54310</value>
</property>

<property>
  <name>hadoop.tmp.dir</name>
  <value>/home/parim/apps/hadoop-2.8.5/tmp-data</value>
  <description>Parent directory for other temporary directories.</description>
</property>
```
#### fs.defaultFS
默认文件系统的名称，本身是一个URL，其方案和权限决定了FileSystem的实现。
此URL的方案（scheme ）确定命名FileSystem 实现类的配置属性（fs.SCHEME.impl）。
此URL的权限（authority ）用于确定文件系统的主机，端口等。
参考[core-default.xml#fs.defaultFS](https://hadoop.apache.org/docs/r2.8.5/hadoop-project-dist/hadoop-common/core-default.xml#fs.defaultFS)

#### hadoop.tmp.dir

hadoop.tmp.dir的目录默认指向的是：/tmp/hadoop-${USERNAME}

这样会有个问题，系统重启时会自动删除/tmp目录下的文件，导致你之前对hadoop做的很多

操作都被删除了，需要重新再来，比如你想hdfs导入的文件会都被删除。

参考：
1. [Hadoop安装](https://www.yiibai.com/hadoop/how_to_install_hadoop.html)
2. [hadoop修改hadoop.tmp.dir](https://blog.csdn.net/sunrising_hill/article/details/50526858)

### etc/hadoop/hdfs-site.xml
这里有两个参数需要配置：
1. dfs.replication - 默认备份（块复制,block replication）。创建文件时可以指定实际的备份数。如果未在创建时指定备份，则使用默认值。[hdfs-default.xml#dfs.replication](https://hadoop.apache.org/docs/r2.8.5/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml#dfs.replication)
2. dfs.datanode.hostname - datanode默认的hostname，这个不是必须的，因为这里是两台服务器之间传递信息，所以需要配置，否则会导致后面的读写操作异常。

打开hdfs-site.xml文件：
```
vi etc/hadoop/hdfs-site.xml
```
拷贝以下所有行的内容放入到标签 ```<configuration></configuration>``` 中间。
```
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>

    <property>
        <name> dfs.datanode.hostname</name>
        <value>192.168.0.80</value>
    </property>

```
### 格式化HDFS 
在第一使用 Hadoop 之前，需要先格式化 HDFS，使用下面的命令
```
bin/hdfs namenode -format
```
若有需要选择的，输入按提示Y即可。

### 启动单节点集群
使用以下命令启动cHadoop 的单节点集群(使用对应的用户来启动)，如下：
```
sbin/start-dfs.sh
```
若该用户之前设置了免密登录，此处可免去多次输入密码的操作，反之则需要根据提示多次输入登录密码。

### 检测运行

使用 'jps' 工具/命令, 验证是否所有 Hadoop 相关的进程正在运行。
```
/home/parim//apps/jdk1.8.0_181/bin/jps
```
如果 Hadoop 成功启动，那么 jps 输出应显示： NameNode, SecondaryNameNode, DataNode.

### 停止/关闭 Hadoop

```
sbin/stop-dfs.sh
```

## Hadoop与Java版本
| Hadoop | Java |
| --- | --- |
| 2.7及以后版本 | Java 7 + |
| 2.6及以前版本 | Java 6 +|
[HadoopJavaVersions](https://wiki.apache.org/hadoop/HadoopJavaVersions)