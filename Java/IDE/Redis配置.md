---
title: Redis配置
tags: redis
grammar_cjkRuby: true
---
## 设置密码
打开redis.conf文件，搜索requirepass关键字

找到#requirepass foobared一行， 去掉#，修改“ foobared”为想要设置的密码即可。

## 开启远程连接
redis服务器默认是处于保护模式并只能本地访问，打开redis.conf文件可以看到如下配置
1. 放开redis端口，如6379。
2. 修改redis.conf配置文件
	本地访问控制：找到bind 127.0.0.1，把这行前面加个#注释掉
	保护模式控制：