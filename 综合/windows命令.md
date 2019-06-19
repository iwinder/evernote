```shell
# 查看进程
netstat -ano

# 查看端口号
netstat -ano |findstr 2181

# 结束进程
taskkill /pid 15868 /F
```


```
 bin/windows/zookeeper-server-start.bat config/zookeeper.properties
 bin/windows/kafka-server-start.bat config/server.properties &
 bin/windows/kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
 bin/windows/kafka-topics.bat --list --zookeeper localhost:2181

 bin/windows/kafka-topics.bat --describe --zookeeper localhost:2181 --topic test

  bin/windows/kafka-console-producer.bat --broker-list localhost:9092 --topic test
  bin/windows/kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test --from-beginning
```