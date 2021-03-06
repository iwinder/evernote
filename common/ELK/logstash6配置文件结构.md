---
title: logstash6配置文件结构
tags: Logstash
grammar_cjkRuby: true
---
## 配置文件的结构
对于要添加到事件处理管道的每种类型的插件，Logstash配置文件都有一个单独的区域（section）。


```
# This is a comment. You should use comments to describe
# parts of your configuration.
input {
  ...
}

filter {
  ...
}

output {
  ...
}
```
Logstash 用 {} 来定义区域（section ）。区域内可以包括插件区域定义，你可以在一个区域内定义多个插件。插件区域内则可以定义键值对设置。

## 插件配置结构
插件的配置包括插件名称，其后跟该插件的设置块。如下面的input区域定义了两个File组件：

```
input {
  file {
    path => "/var/log/messages"
    type => "syslog"
  }

  file {
    path => "/var/log/apache/access.log"
    type => "apache"
  }
}
```
在此示例中，为每个文件输入配置了两个设置：路径和类型。
您可以配置的设置因插件类型而异。
详情可见 [Input Plugins](https://www.elastic.co/guide/en/logstash/6.x/input-plugins.html), [Output Plugins](https://www.elastic.co/guide/en/logstash/6.x/output-plugins.html)，[Filter Plugins]()， 以及 [Codec Plugins](https://www.elastic.co/guide/en/logstash/6.x/codec-plugins.html)。

| 插件 | 用途 |
| --- | --- |
| Input Plugins |输入插件，使Logstash能够读取特定的事件源。 |
| Output Plugins| 输出插件 ，输出插件将事件数据发送到特定目标。输出是事件管道的最后阶段。本身支持多输出配置。 |
| Filter Plugins | 过滤器插件对事件执行中间处理。过滤器通常根据事件的特征有条件地应用。|
| Codec Plugins |过滤器插件对事件执行中间处理。过滤器通常根据事件的特征有条件地应用。 |
### 工作原理
Logstash事件处理管道有三个阶段：输入→过滤器→输出。

输入生成事件，过滤器修改它们，输出将它们发送到其他地方。

输入和输出支持编解码器，使您能够在数据进入或退出管道时对数据进行编码或解码，而无需使用单独的过滤器。

## 数据类型
插件可以要求设置的值为特定类型，例如布尔值（boolean），列表（list）或散列（hash）。支持的值类型如下：

- Array  

```
users => [ {id => 1, name => bob}, {id => 2, name => jane} ]
```

- Lists 

``` 
path => [ "/var/log/messages", "/var/log/*.log" ]
uris => [ "http://elastic.co", "http://example.net" ]
``` 

- Boolean

```
ssl_enable => true
```

 - Bytes

``` 
  my_bytes => "1113"   # 1113 bytes
  my_bytes => "10MiB"  # 10485760 bytes
  my_bytes => "100kib" # 102400 bytes
  my_bytes => "180 mb" # 180000000 bytes
``` 
- Codec

```
 codec => "json"
```
- Hash

```
match => {
  "field1" => "value1"
  "field2" => "value2"
  ...
}
```
- Numbers 

 ```
port => 33
 ```

- Password

```
my_password => "password"
```

- URI 

```
my_uri => "http://foo:bar@example.net" 
```

- Path

```
my_path => "/tmp/logstash"
```

- string

```
host => "hostname"
```

- Escape sequences
默认情况下，不启用转义序列。如果您希望在带引号的字符串中使用转义序列，则需要在logstash.yml中设置```config.support_escapes：true```。如果为true，则引用的字符串（double和single）将具有此转换：

| Text | Result |
| --- | --- |
| \r |  carriage return (ASCII 13) |
| \n | new line (ASCII 10) |
| \t |  tab (ASCII 9) |
| \\ | backslash (ASCII 92) |
| \" | double quote (ASCII 34) |
| \' | single quote (ASCII 39) |
```
  name => "Hello world"
  name => 'It\'s a beautiful day'
```
- Comments 
注释
```
# this is a comment

input { # comments can appear at the end of a line, too
  # ...
}  
```

 ### 参考资料
 [Structure of a Config File](https://www.elastic.co/guide/en/logstash/6.x/configuration-file-structure.html)
 
 [配置语法](https://doc.yonyoucloud.com/doc/logstash-best-practice-cn/get_start/full_config.html)
 

