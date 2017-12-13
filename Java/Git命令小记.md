---
title: Git命令小记 
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


## 克隆分支

git clone -b 分支名 仓库地址

[使用git克隆指定分支的代码](https://www.cnblogs.com/nylcy/p/6569284.html)

##  git 切换远程分支

```
//列出远程所有分支
git branch -a

//检出并切换分支
git checkout -b training origin/feature/training
```
## 查看git配置
```
git config -l
```