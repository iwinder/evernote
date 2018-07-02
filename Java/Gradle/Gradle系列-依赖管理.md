---
title: Gradle系列-依赖管理
tags: Gradle
grammar_cjkRuby: true
---

## 什么是依赖管理?
粗略的讲, 依赖管理由两部分组成：项目的  **dependencies(依赖项)** 和 **publications(发布项)**。

1. Gradle  **需要了解你的项目需要构建或运行的东西, 以便找到它们**。我们称这些传入的文件为项目的 dependencies(依赖项)。
2. Gradle  **需要构建并上传你的项目产生的东西**。我们称这些传出的项目文件为 **publications(发布项)**。

### 依赖
简言为个人理解，细说为出处。若对简言不清楚，可查看理解细说部分。
#### 简言：

1. 根据配置获取依赖关系的过程为 **dependency resolution(依赖解析)** 。
2. 项目运行时寻找到其依赖关系并使其可用的过程为**dependency resolution(依赖解析)** 。

#### 细说：

**大多数项目都不是完全独立的 ，它们需要其它项目进行编译或测试等等** 。举个例子, 为了在项目中使用 Hibernate, 在编译的时候需要在 classpath 中添加一些 Hibernate 的 jar 路径. 要运行测试的时候, 需要在 test classpath 中包含一些额外的 jar, 比如特定的 JDBC 驱动或者 Ehcache jars.

这些传入的文件构成上述项目的依赖。 Gradle 允许你告诉它项目的依赖关系, 以便找到这些依赖关系, 并在你的构建中维护它们。 

依赖关系可能需要**从远程的 Maven 或者 Ivy 仓库中下载**, 也可能是**在本地文件系统中**， 或者是**通过多项目构建另一个构建**。我们称这个过程为**dependency resolution(依赖解析)** 。

通常, 一个项目本身会具有依赖性. 举个例子, 运行 Hibernate 的核心需要其他几个类库在 classpath 中. 因此, Gradle 在为你的项目运行测试的时候, 它**会找到这些依赖关系, 并使其可用** 。 我们称之为**transitive dependencies(依赖传递)** 。

### 发布

#### 简言：
**项目的主要目的是要建立一些文件，在项目之外使用。Gradle可以负责完成这一系列任务，而这一过程称为publication(发布)**。

#### 细说：

大部分项目的主要目的是要建立一些文件，在项目之外使用。比如，你的项目产生一个 Java 库，你需要构建一个jar，可能是一个 jar 和一些文档， 并将它们发布在某处。

这些传出的文件构成了项目的发布。Gradle 当然会为你负责这个重要的工作。你声明项目的发布，Gradle 会构建并发布在某处。究竟什么是"发布"取决于你想做什么。可能你希望将文件复制到本地目录， 或者将它们上传到一个远程 Maven 或者 Ivy 库.或者你可以使用这些文件在多项目构建中应用在其它的项目中。我们称这个过程为 publication(发布)。

## 声明依赖

简单的依赖声明：
```
//应用插件,这里引入了Gradle的Java插件,此插件提供了Java构建和测试所需的一切。
apply plugin: 'java'

//仓库
repositories {
    mavenCentral()
}
//定义依赖
dependencies {
    compile group: 'org.hibernate', name: 'hibernate-core', version: '3.6.7.Final'
    testCompile group: 'junit', name: 'junit', version: '4.+'
}
```