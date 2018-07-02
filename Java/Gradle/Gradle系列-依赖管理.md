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

**大多数项目都不是完全独立的 ，它们需要其它项目进行编译或测试等等** 。举个例子, 为了在项目中使用 Hibernate, 在编译的时候需要在 classpath 中添加一些 Hibernate 的 jar 路径. 要运行测试的时候, 需要在 test classpath 中包含一些额外的 jar, 比如特定的 JDBC 驱动或者 Ehcache jars.

这些传入的文件构成上述项目的依赖。 Gradle 允许你告诉它项目的依赖关系, 以便找到这些依赖关系, 并在你的构建中维护它们。 

依赖关系可能需要**从远程的 Maven 或者 Ivy 仓库中下载**, 也可能是**在本地文件系统中**， 或者是**通过多项目构建另一个构建**。我们称这个过程为**dependency resolution(依赖解析)** 。

通常, 一个项目本身会具有依赖性. 举个例子, 运行 Hibernate 的核心需要其他几个类库在 classpath 中. 因此, Gradle 在为你的项目运行测试的时候, 它**会找到这些依赖关系, 并使其可用** 。 我们称之为**transitive dependencies(依赖传递)** 。

