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

若是正则匹配，成功后添加字段：
```
filter {
	if [message] =~ /\w+\s+\/\w+(\/learner\/course\/)/ {
		mutate {
			add_field => { "learner_type" => "course" }
		}
	}
}
```