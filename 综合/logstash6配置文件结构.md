---
title: logstash6配置文件结构
tags: 新建,模板,小书匠
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

