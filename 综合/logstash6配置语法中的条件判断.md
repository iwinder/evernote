---
title: logstash6配置语法中的条件判断
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---
详情可见[官方文档-conditionals](https://www.elastic.co/guide/en/logstash/6.x/event-dependent-configuration.html#conditionals)。

有时您只想在特定条件下过滤或输出事件。为此，您可以使用条件（conditional）。

Logstash中的条件查看和行为与编程语言中的条件相同。

条件语支持if，else if和else语句并且可以嵌套。

条件语法如下：
```
if EXPRESSION {
  ...
} else if EXPRESSION {
  ...
} else {
  ...
}
```

**比较操作**：

- 相等: ```==```, ```!=```, ```<```, ```>```, ```<=```, ```>=```
- 正则: ```=~```(匹配正则), ```!~```(不匹配正则)
- 包含: ```in```(包含), ```not in```(不包含)

**布尔操作**：
- ```and```(与), ```or```(或), ```nand```(非与), ```xor```(非或)

***一元运算符***：
表达式可能很长且很复杂。表达式可以包含其他表达式，您可以使用！来取消表达式，并且可以使用括号（...）对它们进行分组。
- ```!```(取反)
- ```()```(复合表达式), ```!()```(对复合表达式结果取反)

如若action是login则mutate filter删除secret字段：

```
filter {
  if [action] == "login" {
    mutate { remove_field => "secret" }
  }
}
```

若是正则匹配，成功后添加自定义字段：
```
filter {
	if [message] =~ /\w+\s+\/\w+(\/learner\/course\/)/ {
		mutate {
			add_field => { "learner_type" => "course" }
		}
	}
}
```

在一个条件里指定多个表达式：
```
output {
  # Send production errors to pagerduty
  if [loglevel] == "ERROR" and [deployment] == "production" {
    pagerduty {
    ...
    }
  }
}
```

在in条件，可以比较字段值：
```
filter {
  if [foo] in [foobar] {
    mutate { add_tag => "field in field" }
  }
  if [foo] in "foo" {
    mutate { add_tag => "field in string" }
  }
  if "hello" in [greeting] {
    mutate { add_tag => "string in field" }
  }
  if [foo] in ["hello", "world", "foo"] {
    mutate { add_tag => "field in list" }
  }
  if [missing] in [alsomissing] {
    mutate { add_tag => "shouldnotexist" }
  }
  if !("foo" in ["hello", "world"]) {
    mutate { add_tag => "shouldexist" }
  }
}
```
not in示例：
```
output {
  if "_grokparsefailure" not in [tags] {
    elasticsearch { ... }
  }
}
```

### @metadata field 

在logstash1.5版本开始，有一个特殊的字段，叫做@metadata。@metadata包含的内容不会作为事件的一部分输出。
```
input { stdin { } }

filter {
  mutate { add_field => { "show" => "This data will be in the output" } }
  mutate { add_field => { "[@metadata][test]" => "Hello" } }
  mutate { add_field => { "[@metadata][no_show]" => "This data will not be in the output" } }
}

output {
  if [@metadata][test] == "Hello" {
    stdout { codec => rubydebug }
  }
}
```
输出结果
```
$ bin/logstash -f ../test.conf
Pipeline main started
asdf
{
    "@timestamp" => 2016-06-30T02:42:51.496Z,
      "@version" => "1",
          "host" => "windcoder.com",
          "show" => "This data will be in the output",
       "message" => "asdf"
}
```

"asdf"变成message字段内容。条件与@metadata内嵌的test字段内容判断成功，但是输出并没有展示@metadata字段和其内容。

不过，如果指定了metadata => true，rubydebug codec允许显示@metadata字段的内容。

```
   stdout { codec => rubydebug { metadata => true } }
```
输出结果
```
$ bin/logstash -f ../test.conf
Pipeline main started
asdf
{
    "@timestamp" => 2016-06-30T02:46:48.565Z,
     "@metadata字段及其子字段内容。" => {
           "test" => "Hello",
        "no_show" => "This data will not be in the output"
    },
      "@version" => "1",
          "host" => "windcoder.com",
          "show" => "This data will be in the output",
       "message" => "asdf"
}
```

现在就可以见到@metadata字段及其子字段内容。

只有rubydebug codec允许显示@metadata字段的内容。

只要您需要临时字段但不希望它在最终输出中，就可以使用@metadata字段。


最常见的情景是filter的时间字段，需要一临时的时间戳。如：

```
input { stdin { } }

filter {
  grok { match => [ "message", "%{HTTPDATE:[@metadata][timestamp]}" ] }
  date { match => [ "[@metadata][timestamp]", "dd/MMM/yyyy:HH:mm:ss Z" ] }
}

output {
  stdout { codec => rubydebug }
}
```

输出结果
```
$ bin/logstash -f ../test.conf
Pipeline main started
02/Mar/2014:15:36:43 +0100
{
    "@timestamp" => 2014-03-02T14:36:43.000Z,
      "@version" => "1",
          "host" => "windcoder.com",
       "message" => "02/Mar/2014:15:36:43 +0100"
}
```

### 参考资料
[官方文档-conditionals](https://www.elastic.co/guide/en/logstash/6.x/event-dependent-configuration.html#conditionals)

[ELK logstash 配置语法(24th)](http://www.ttlsa.com/elk/elk-logstash-configuration-syntax/)