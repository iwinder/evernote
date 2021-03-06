---
title: Gradle小记 
tags: Gradle
grammar_cjkRuby: true
---

## repositories 

仓库。
Gradle需要你指定至少一个仓库作为依赖下载的地方，比如mavenCentral
```
repositories {
    mavenCentral()
}
```
|仓库|含义|
|---|---|
|mavenLocal()|本地仓库|
|mavenCentral()|远程maven仓库|
| maven {name 'Custom Maven Repository',url 'http://repository.forge.cloudbees.com/release/')}|自定义仓库|
## dependencies 
定义依赖，依赖通过group标识，name和version来确定，比如下面这个：
```
dependencies {
    compile group: 'org.apache.commons', name: 'commons-lang3', version: '3.1'
}
```
## 构建块

每个Gradle构建都包括三个基本的构建块：项目(projects)、任务(tasks)和属性(properties)，每个构建至少包括一个项目，项目包括一个或者多个任务，项目和任务都有很多个属性来控制构建过程。

### 外部属性
外部属性一般存储在键值对中，要添加一个属性，你需要使用ext命名空间。

外部属性可以定义在一个属性文件中： 通过在/.gradle路径或者项目根目录下的gradle.properties文件来定义属性可以直接注入到你的项目中，他们可以通过project实例来访问，注意/.gradle目录下只能有一个Gradle属性文件即使你有多个项目

## allprojects 
配置对所有项目生效
## subprojects
配置对所有子项目生效
## project(':web') 
指定项目的行为通过project方法来定义---定义项目特有的行为
## buildscript
buildscript中的声明是gradle脚本自身需要使用的资源。可以声明的资源包括依赖项、第三方插件、maven仓库地址等。而在build.gradle文件中直接声明的依赖项、仓库地址等信息是项目自身需要的资源。

[Gradle In Action(Gradle实战)中文版](https://www.gitbook.com/book/lippiouyang/gradle-in-action-cn/details)
[Gradle中的buildScript代码块](http://www.cnblogs.com/huang0925/p/3940528.html)

## compile files

```
compile fileTree(dir: 'libs', include: '*.jar')
```
将libs目录下所有jar文件进行编译并打包
[compile、provided、compile files、compile project四者的区别](http://blog.csdn.net/xiaoxiaoniaoge/article/details/50519501)


## bootRepackage.enabled

bootRepackage任务,它的作用是重新打包jar为可执行的jar

[spring-boot填坑](http://blog.csdn.net/buyaore_wo/article/details/78062684)
[Spring Boot Gradle插件](https://course.tianmaying.com/spring-boot-reference+lesson-59#8)


## 多模块
[Gradle实战系列（三）__多模块项目](https://www.cnblogs.com/linjl/p/6361970.html)

[Gradle构建多模块项目](https://yq.aliyun.com/articles/25589)

[Java Gradle入门指南之内建与定制任务类（buildSrc、Groovy等）](https://www.cnblogs.com/gzdaijie/p/5285160.html)
