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

每个Gradle构建都包括三个基本的构建块：项目(projects)、任务(tasks)和属性(properties)，每个构建至少包括一个项目，项目包括一个或者多个任务，项目和任务都有很多个属性来控制构建过程。


