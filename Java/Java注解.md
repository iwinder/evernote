---
title: Java注解
tags: Java,注解
grammar_cjkRuby: true
---


## @Document
说明该注解将被包含在javadoc中

##  @Retention 
注解的保留位置
### @Retention(RetentionPolicy.SOURCE)
//注解仅存在于源码中，在class字节码文件中不包含
### @Retention(RetentionPolicy.CLASS)  
// 默认的保留策略，注解会在class字节码文件中存在，但运行时无法获得，
### @Retention(RetentionPolicy.RUNTIME)
// 注解会在class字节码文件中存在，在运行时可以通过反射获取到