---
title: Gradle系列-依赖管理
tags: Gradle
grammar_cjkRuby: true
---

## 什么是依赖管理?
粗略的讲, 依赖管理由两部分组成：项目的  **dependencies(依赖项)** 和 **publications(发布项)** 。

1. Gradle  **需要了解你的项目需要构建或运行的东西, 以便找到它们** 。 我们称这些传入的文件为项目的 dependencies(依赖项)。
2. Gradle  **需要构建并上传你的项目产生的东西** 。我们称这些传出的项目文件为 **publications(发布项)** . 