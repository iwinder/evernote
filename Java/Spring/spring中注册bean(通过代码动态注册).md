---
title: spring中注册bean(通过代码动态注册)
tags: springBoot
grammar_cjkRuby: true
---

```
//将applicationContext转换为ConfigurableApplicationContext
		ConfigurableApplicationContext configurableApplicationContext = (ConfigurableApplicationContext) applicationContext;
		
		// 获取bean工厂并转换为DefaultListableBeanFactory
		DefaultListableBeanFactory defaultListableBeanFactory = (DefaultListableBeanFactory) configurableApplicationContext
				.getBeanFactory();
		
		// 通过BeanDefinitionBuilder创建bean定义
		BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder
				.genericBeanDefinition(UserService.class);
		// 设置属性userAcctDAO,此属性引用已经定义的bean:userAcctDAO
		beanDefinitionBuilder
				.addPropertyReference("userAcctDAO", "UserAcctDAO");
		
		// 注册bean
		defaultListableBeanFactory.registerBeanDefinition("sdfds",
				beanDefinitionBuilder.getRawBeanDefinition());



例子：

```
	private static ApplicationContext applicationContext = SpringUtil.getApplicationContext();
	private static DefaultListableBeanFactory defaultListableBeanFactory = (DefaultListableBeanFactory)applicationContext.getAutowireCapableBeanFactory();
	
    public static LdapTemplate setLdapBeen(LdapSetting ldap) {
    	String bennName = "ldapTemplate"+ldap.getSite().getId();
    	BeanDefinitionBuilder beanDefinitionBuilder =BeanDefinitionBuilder.genericBeanDefinition(LdapTemplate.class);
    	beanDefinitionBuilder.addPropertyValue("contextSource",getLdapContextSource(ldap));
    	defaultListableBeanFactory.registerBeanDefinition(bennName,beanDefinitionBuilder.getBeanDefinition());
    	return (LdapTemplate) applicationContext.getBean(bennName);
    }
```


[IDEA打包war部署到tomcat 404错误（tomcat配置正确）](https://blog.csdn.net/jyfjyt/article/details/78617112)


[https://coderanch.com/c/java](https://coderanch.com/c/java)

[Curious case of ConcurrentHashMap](https://medium.com/@itsromiljain/curious-case-of-concurrenthashmap-90249632d335)

[ConcurrentHashMap – avoid a common misuse!](https://ria101.wordpress.com/2011/12/12/concurrenthashmap-avoid-a-common-misuse/)

[How to Create a thread-safe ConcurrentHashSet in Java 8? Example

Read more: https://javarevisited.blogspot.com/2017/08/how-to-create-thread-safe-concurrent-hashset-in-java-8.html#ixzz5Wj2FbUQM]

[在Java 8中锁定备选方案](http://qaware.blogspot.com/2016/06/locking-alternatives-in-java-8.html)


[配置springboot在访问404时自定义返回结果以及统一异常处理](http://www.cnblogs.com/xujingyang/p/9103554.html)