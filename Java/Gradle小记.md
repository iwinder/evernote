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


