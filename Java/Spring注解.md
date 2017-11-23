---
title: Spring注解 
tags: Java,Spring
grammar_cjkRuby: true
---

## @PathVariable

## @RequestBody

## @RequestParam


## @Component
泛指组件，当组件不好归类的时候，我们可以使用这个注解进行标注。  

[spring @component的作用](http://tomfish88.iteye.com/blog/1497557)

[@Component注解](http://uule.iteye.com/blog/2106427)

## @ConditionalOnProperty
当指定的配置属性要有一个明确的值

[深入SpringBoot:自定义Conditional](http://www.jianshu.com/p/1d0fb7cd8a26)
[spring-boot学习笔记之Conditional](http://www.jianshu.com/p/0740c07f6c1d)
[springboot根据不同的条件创建bean，动态创建bean，@Conditional注解使用](http://blog.csdn.net/tianyaleixiaowu/article/details/78201587)

## @Configuration
标明该类使用Spring基于Java的配置。@Configuration注解该类，等价于XML中配置beans；用@Bean标注方法等价于XML中配置bean。