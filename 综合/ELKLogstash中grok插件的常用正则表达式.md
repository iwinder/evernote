---
title: Logstash中grok插件的常用正则表达式
tags: Logstash
grammar_cjkRuby: true
---
## grok默认表达式
Logstash 内置了120种默认表达式，可以查看[patterns](https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns)，里面对表达式做了分组，每个文件为一组，文件内部有对应的表达式模式。下面只是部分常用的。

### 常用表达式
| 表达式标识|名称|详情|匹配例子|
| --- | --- | --- | --- |
| USERNAME 或 USER | 用户名 | 由数字、大小写及特殊字符(.\_\-)组成的字符串 | ```1234```、```Bob```、```Alex.Wong``` |
| EMAILLOCALPART | 用户名 | 首位由大小写字母组成，其他位由数字、大小写及特殊字符(\_.+-=:)组成的字符串。注意，国内的QQ纯数字邮箱账号是无法匹配的，需要修改正则 | ```stone```、```Gary_Lu```、```abc-123``` |
| EMAILADDRESS | 电子邮件 | | ```stone@abc.com```、```Gary_Lu@gmail.com```、```abc-123@163.com```|
| HTTPDUSER | Apache服务器的用户 | 可以是```EMAILADDRESS```或```USERNAME```| |
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
| URIPATH | URI路径 | | ```//www.stozen.net/abc/```、```/api.php``` |
|  URIPARAM |  URI里的GET参数 | | ```?a=1&b=2&c=3``` |
| URIPATHPARAM | URI路径+GET参数 | ```//www.stozen.net/abc/api.php?a=1&b=2&c=3```|
| URI | 完整的URI | | ```http://www.stozen.net/abc/api.php?a=1&b=2&c=3```|
|LOGLEVEL| Log表达式 | Log表达式 | ```Alert```、```alert```、```ALERT```、```Error``` |
### 日期时间表达式
| 表达式标识|名称|匹配例子|
| --- | --- | --- | 
| MONTH | 月份名称 | ```Jan```、```January``` |
| MONTHNUM | 月份数字 |  ```03```、```9```、```12``` |
| MONTHDAY | 日期数字 |  ```03```、```9```、```31``` |
| DAY | 星期几名称 | ```Mon```、```Monday``` |
| YEAR | 年份数字 | |
| HOUR | 小时数字 | |
| MINUTE | 分钟数字 ||
| SECOND | 秒数字 | |
| TIME | 时间 | ```00:01:23``` |
| DATE_US | 美国时间 |  ```10-01-1892```、```10/01/1892/```|
| DATE_EU | 欧洲日期格式 | ```01-10-1892```、```01/10/1882```、```01.10.1892``` |
| ISO8601_TIMEZONE | ISO8601时间格式 |  ```+10:23```、```-1023``` |
| TIMESTAMP_ISO8601 | ISO8601时间戳格式 | ```2016-07-03T00:34:06+08:00``` |
| DATE | 日期 | 美国日期%{DATE_US}或者欧洲日期%{DATE_EU} | |
| DATESTAMP | 完整日期+时间| ```07-03-2016 00:34:06``` |
| HTTPDATE | http默认日期格式 | ```03/Jul/2016:00:36:53 +0800``` |

## 自定义grok表达式
上面列举的只是一部分，更多的可以自己搜索查找，如果需要自定义，需要按以下步骤进行：

-  创建一个名为patterns的目录，其中包含一个名为extra的文件（文件名无关紧要，但为自己命名有意义）
-  在该文件中，将您需要的模式按如下格式书写：模式名称，空格，然后是该模式的正则表达式。

例如，获取 一个queue id：
```
# contents of ./patterns/postfix:
POSTFIX_QUEUEID [0-9A-F]{10,11}
```

然后使用此插件中的patterns_dir 字段设置告诉logstash您的自定义模式目录所在的位置。
这是一个包含示例日志的完整示例：

```
Jan  1 06:25:43 mailserver14 postfix/cleanup[21403]: BEF25A72965: message-id=<20130101142543.5828399CCAF@mailserver14.example.com>

```
配置：
```
filter {
  grok {
    patterns_dir => ["./patterns"]
    match => { "message" => "%{SYSLOGBASE} %{POSTFIX_QUEUEID:queue_id}: %{GREEDYDATA:syslog_message}" }
  }
}
```
结果：
```
timestamp: Jan 1 06:25:43
logsource: mailserver14
program: postfix/cleanup
pid: 21403
queue_id: BEF25A72965
syslog_message: message-id=<20130101142543.5828399CCAF@mailserver14.example.com>
```
另一种选择是使用[pattern_definitions](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html#plugins-filters-grok-pattern_definitions)在过滤器中定义内联模式。
这主要是为了方便起见，并允许用户定义一个可以在该过滤器中使用的模式。
pattern_definitions中新定义的模式在特定的grok过滤器之外将不可用。