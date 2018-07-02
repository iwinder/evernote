---
title: Gradle总结笔记 
tags:  Gradle, Gradle, Gradle
grammar_cjkRuby: true
---

## buildscript

官方解释为：

> Configures the build script classpath for this project.
> 
> The given closure is executed against this project's ScriptHandler.
> 
> The [ScriptHandler](https://docs.gradle.org/current/javadoc/org/gradle/api/initialization/dsl/ScriptHandler.html) is passed to the closure as the closure's delegate.
>
>来源:[buildscript { }](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html#org.gradle.api.Project:buildscript(groovy.lang.Closure))

配置此项目的构建脚本类路径。可声明用于编译和执行构建脚本的类路径。该类路径也用于加载构建脚本使用的插件。

简单说**即设置脚本的运行环境**。

**buildscript**中的声明是**gradle脚本自身**需要使用的资源。可以声明的资源包括依赖项、第三方插件、maven仓库地址等。

而在**build.gradle文件**中直接声明的依赖项、仓库地址等信息是**项目自身**需要的资源。

例：

```
buildscript {
    ext {
        springBootVersion = "1.5.8.RELEASE"
    }
    repositories {
        mavenCentral()
        maven { url "https://repo.spring.io/plugins-release" }
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}
apply plugin: 'org.springframework.boot'
```

### ext
额外的属性扩展允许将新属性添加到现有的域对象。即**用于配置额外的属性**。
详情：[ExtraPropertiesExtension](https://docs.gradle.org/current/dsl/org.gradle.api.plugins.ExtraPropertiesExtension.html)

### repositories
配置该项目的存储库。支持java 依赖库管理（maven/ivy）,用于项目的依赖。

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

#### 自定义仓库其他写法
```
 maven{url 'http://maven.aliyun.com/nexus/content/groups/public/'}
```


### dependencies
配置此项目的依赖关系。依赖包的定义。支持maven/ivy，远程，本地库，也支持单文件，如果前面定义了repositories{}maven 库，使用maven的依赖（我没接触过ivy。。）的时候只需要按照用类似于```com.android.tools.build:gradle:0.4```，gradle 就会自动的往远程库下载相应的依赖。

#### 写法：
1、依赖通过group标识，name和version来确定，比如下面这个：
```
dependencies {
    compile group: 'org.apache.commons', name: 'commons-lang3', version: '3.1'
}
```

2、简写

```
dependencies {
    compile("org.springframework.boot:spring-boot-starter-web:${springBootVersion}")
}
```
3、添加libs的所有jar包为依赖
```
    dependencies {
        compile fileTree(dir: 'lib', exclude:'', include: '*.jar')
    }
```
将libs目录下所有jar文件进行编译并打包。

4、引入另一个模块
```
    dependencies {
        compile project(":windcoder-com:test")
    }
```
 即是将另一个module（等同eclipse中的library)进行编译并打包

5、buildscript代码块独有
```
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
```
buildscript代码块中你可以对dependencies使用classpath声明。该classpath声明说明了在执行其余的build脚本时，class loader可以使用这些你提供的依赖项。这也正是我们使用buildscript代码块的目的。

[dependencies {}](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html#org.gradle.api.Project:repositories(groovy.lang.Closure))
[用Gradle 构建你的android程序](https://www.cnblogs.com/youxilua/archive/2013/05/20/3087935.html)
[compile、provided、compile files、compile project四者的区别](https://blog.csdn.net/xiaoxiaoniaoge/article/details/50519501)

## allprojects{}
配置此项目及其每个子项目。

## subprojects{}
配置该项目的子项目。

## configure(rootProject){}
配置一个对象，如此处配置根项目。
[Object configure（Objectobject，ClosureconfigureClosure）](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html#org.gradle.api.Project:configure(java.lang.Object,%20groovy.lang.Closure))


## bootRepackage
SpringBoot构建插件（spring-boot-gradle-plugin），有一个bootRepackage任务,它的作用是重新打包jar为可执行的jar。
所以他需要指定一个MainClass， 解决办法：
关掉bootRepackage任务
```
bootRepackage.enabled = false
```
或配置mainclass
```
    springBoot {
        mainClass = "com.windcoder.nigthbook.BookApplication";
        buildInfo()
    }
```
[spring-boot填坑](https://blog.csdn.net/buyaore_wo/article/details/78062684)
