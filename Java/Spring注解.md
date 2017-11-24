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
当指定的配置属性要有一个明确的值。

这个注解能够控制某个configuration是否生效。具体操作是通过其两个属性name以及havingValue来实现的，其中name用来从application.properties中读取某个属性值，如果该值为空，则返回false;如果值不为空，则将该值与havingValue指定的值进行比较，如果一样则返回true;否则返回false。如果返回值为false，则该configuration不生效；为true则生效。
```
//查找test-hehe属性的默认值并与havingValue中的值true比较，为true时符合条件，将执行被注解项。如果没有设置则默认为true,此时条件符合，将执行被注解项。
@ConditionalOnProperty(name="test-hehe", havingValue = "true", matchIfMissing=true)
```


[深入SpringBoot:自定义Conditional](http://www.jianshu.com/p/1d0fb7cd8a26)
[spring-boot学习笔记之Conditional](http://www.jianshu.com/p/0740c07f6c1d)
[springboot根据不同的条件创建bean，动态创建bean，@Conditional注解使用](http://blog.csdn.net/tianyaleixiaowu/article/details/78201587)
[SpringBoot学习笔记(3) Spring Boot 运行原理,自动配置](http://blog.csdn.net/a67474506/article/details/52013634)
[配置Spring Boot通过@ConditionalOnProperty来控制Configuration是否生效](http://blog.csdn.net/dalangzhonghangxing/article/details/78420057)
### 拓展
[spring boot应用启动原理分析](https://yq.aliyun.com/articles/6056)

## @Configuration
标明该类使用Spring基于Java的配置。@Configuration注解该类，等价于XML中配置beans；用@Bean标注方法等价于XML中配置bean。
[Spring中基于Java的配置@Configuration和@Bean用法](http://blog.csdn.net/vvhesj/article/details/47661001)