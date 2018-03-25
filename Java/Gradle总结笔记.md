---
title: Gradle笔记 
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

## buildscript

官方解释为：

> Configures the build script classpath for this project.
> 
> The given closure is executed against this project's ScriptHandler.
> 
> The [ScriptHandler](https://docs.gradle.org/current/javadoc/org/gradle/api/initialization/dsl/ScriptHandler.html) is passed to the closure as the closure's delegate.

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