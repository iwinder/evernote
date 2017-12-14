---
title: Git命令小记 
tags: Git,代码管理工具,
grammar_cjkRuby: true
---

## 配置

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

## 分支

### 查看分支
```
//查看本地和远程所有分支
git branch -a
//查看远程所有分支
git branch -ｒ
//查看本地分支
git branch
```
[git branch用法总结，查看、新建、删除、重命名](http://blog.csdn.net/afei__/article/details/51567155)

### 重命名分支
#### 重命名本地分支
```
git branch -m old_local_branch_name new_local_branch_name
```
#### 重命名远程分支
```
Step1：重命名远程分支对应的本地分支
git branch -m old_local_branch_name new_local_branch_name

step2：删除远程分支
git push origin :old_local_branch_name

step3：重新推送新命名的本地分支
git push origin new_local_branch_name
```
[git分支重命名 & 删除tag & 删除远程分支后本地依然存在的解决办法](http://blog.csdn.net/sunny05296/article/details/65449791)

### 克隆分支

```
git clone -b 分支名 仓库地址
```
[使用git克隆指定分支的代码](https://www.cnblogs.com/nylcy/p/6569284.html)


###  git 切换远程分支

```
//检出并切换分支
git checkout -b feature/training origin/feature/training
```

## 删除远程分支
```
git branch -r -d origin/branch-name  
git push origin :branch-name  
```
[git命令行删除远程分支](http://blog.csdn.net/furzoom/article/details/53002699)

