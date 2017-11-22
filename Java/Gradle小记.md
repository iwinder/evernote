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

