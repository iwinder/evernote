---
title: Git命令小记 
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


## 克隆分支

```
git clone -b 分支名 仓库地址
```
[使用git克隆指定分支的代码](https://www.cnblogs.com/nylcy/p/6569284.html)

## 查看分支
```
//查看本地和远程所有分支
git branch -a
//查看远程所有分支
git branch -ｒ
//查看本地分支
git branch
```

##  git 切换远程分支

```


//检出并切换分支
git checkout -b training origin/feature/training
```
## 查看git配置
```
git config -l
```
## 设置全局参数
```
git config --global core.autocrlf input 
git config --global core.diff auto
git config --global core.syslinks false
```

[[git] warning: LF will be replaced by CRLF | fatal: CRLF would be replaced by LF](
http://blog.csdn.net/feng88724/article/details/11600375)

## 快速打开git config
```
git config [--global] --edit
```
[Win10下修改git全部配置文件方法](http://blog.csdn.net/shrimpcolo/article/details/49302619)
[git config命令使用](http://blog.csdn.net/zxncvb/article/details/22153019)
## 删除远程分支
```
git branch -r -d origin/branch-name  
git push origin :branch-name  
```
[git命令行删除远程分支](http://blog.csdn.net/furzoom/article/details/53002699)

