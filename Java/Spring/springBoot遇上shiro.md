---
title: SpringBoot邂逅Shiro
tags: SpringBoot,Shiro
grammar_cjkRuby: true
---
## 前言
本篇仅是记录集成的基础过程，至于shiro框架的基础概念和使用细节，可以自行查阅相关资料，本文不做讨论。

###  集成环境
部分jar包如下：
- springBoot: 1.5.8.RELEASE
- shiro-spring-boot-web-starter: 1.4.0-RC2
- shiro-redis: 3.1.0
- spring-boot-starter-data-redis
项目核心包为SpringBoot 1.5.8.RELEASE以及shiro-spring 1.4.0，预计集成redis，同时使用redis管理Session，所以追加了shiro-redis。

## 参考资料
[https://blog.csdn.net/u013615903/article/details/78781166/](https://blog.csdn.net/u013615903/article/details/78781166/)
[跟我学Shiro目录贴](http://jinnianshilongnian.iteye.com/blog/2018398)