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
| BASE16NUM | 十六进制数字 | 整数 | ```0x0045fa2d```、```-0x3F8709``` |
| WORD | 字符串 |  包括数字和大小写字母 | ```String```、```3529345```、```ILoveYou``` |
| NOTSPACE | 不带任何空格的字符串 | | |
| SPACE | 空格字符串| | |
|QUOTEDSTRING 或 QS | 带引号的字符串| |```"This is an apple"```、```'What is your name?'``` |
| UUID | 标准UUID| |```550E8400-E29B-11D4-A716-446655440000``` |
| MAC | MAC地址 | 可以是Cisco设备里的MAC地址，也可以是通用或者Windows系统的MAC地址| |
| IP | IP地址 | IPv4或IPv6地址 | ```127.0.0.1```、```FE80:0000:0000:0000:AAAA:0000:00C2:0002``` |
| HOSTNAME | IP或者主机名称 | | |
| HOSTPORT | 主机名(IP)+端口 | | ```127.0.0.1:3306```、```api.stozen.net:8000```|
|PATH | 路径 | Unix系统或者Windows系统里的路径格式 | ```/usr/local/nginx/sbin/nginx```、```c:\windows\system32\clr.exe``` |
| URIPROTO | URI协议 | |```http```、```ftp```|
| URIHOST | URI主机 | | ```www.stozen.net```、```10.0.0.1:22``` |
