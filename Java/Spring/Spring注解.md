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

## @EnableScheduling

在 SpringBootApplication 上使用@ServletComponentScan 注解后，Servlet、Filter、Listener 可以直接通过 @WebServlet、@WebFilter、@WebListener 注解自动注册，无需其他代码。

[ Spring Boot Servlet](http://blog.csdn.net/catoop/article/details/50501686)
[Spring @EnableScheduling 注解解析](http://blog.csdn.net/tramp_zzy/article/details/77543269)

## @EnableAsync
开启异步
### @Async

异步调用则是只是发送了调用的指令，调用者无需等待被调用的方法完全执行完毕；而是继续执行下面的流程。

[Spring中@Async用法总结](https://www.cnblogs.com/lcngu/p/6185363.html)

#@EnableScheduling 
通过@EnableScheduling注解开启对计划任务的支持
[【Spring】Spring高级话题-计划任务-@EnableScheduling](http://blog.csdn.net/qq_26525215/article/details/53543816)

[Spring @EnableScheduling 注解解析](http://tramp.cincout.cn/2017/08/18/spring-task-2017-08-18-spring-boot-enablescheduling-analysis/)


##  @JmsListener(destination = "demo.consumer.queue") 
JMS 监听 destination =命名空间


## spring.jpa.hibernate.ddl-auto
是一个Hibernate特性，以更细粒度的方式控制行为

[78. Database initialization](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-database-initialization.html)

## mybatis.mapper-locations
自动扫描路径

## mybatis.configuration.map-underscore-to-camel-case
设为true表示开启驼峰转换。

[【spring boot+mybatis】注解使用方式（无xml配置）设置自动驼峰明明转换（mapUnderscoreToCamelCase），IDEA中xxDao报错could not autowire的解决方法](https://www.cnblogs.com/zhangdong92/p/6986653.html)


## @EnableJpaAuditing
审计相关

[Spring Data JPA教程：审计（一）](http://blog.csdn.net/ro_wsy/article/details/50767207)
[Spring Data JPA教程：审计（二）](http://blog.csdn.net/ro_wsy/article/details/50807579)
[Spring Data审计功能@CreatedDate、@CreatedBy、@LastModifiedDate、@LastModifiedBy的使用](http://blog.csdn.net/tracycater/article/details/74060721)
[【tmos】字段create_time如何动态的生成](https://www.cnblogs.com/mrx520/p/7698463.html)
[Hades——JPA的开源实现](http://www.infoq.com/cn/articles/hades_jpa_repositories_done_right)

## HandlerMethodArgumentResolver
自定参数解析器