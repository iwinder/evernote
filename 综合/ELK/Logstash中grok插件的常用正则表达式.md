---
title: Logstash中grok插件的常用正则表达式
tags: Logstash
grammar_cjkRuby: true
---

| 表达式标识|名称|详情|匹配例子|
| --- | --- | --- | --- |
| USERNAME 或 USER | 用户名 | 由数字、大小写及特殊字符(.\_\-)组成的字符串 | ```1234```、```Bob```、```Alex.Wong``` |
| EMAILLOCALPART | 用户名 | 首位由大小写字母组成，其他位由数字、大小写及特殊字符(\_.+-=:)组成的字符串。注意，国内的QQ纯数字邮箱账号是无法匹配的，需要修改正则 | ```stone```、```Gary_Lu```、```abc-123``` |
| EMAILADDRESS | 电子邮件 | | ```stone@abc.com```、```Gary_Lu@gmail.com```、```abc-123@163.com```|
| HTTPDUSER | Apache服务器的用户 | | 可以是```EMAILADDRESS```或```USERNAME```|
| INT | 整数 | 包括0和正负整数 | ```0```、```-123```、```43987```|
| BASE10NUM 或 NUMBER | 十进制数字 | 包括整数和小数 | ```0```、```18```、```5.23``` |
