<!-- ---
title: Git命令小记 
tags: Git
grammar_cjkRuby: true
--- -->

## 配置

### 查看git配置
```
git config -l
```
### 设置全局参数
```
git config --global core.autocrlf input 
git config --global core.diff auto
git config --global core.syslinks false
```

[[git] warning: LF will be replaced by CRLF | fatal: CRLF would be replaced by LF](
http://blog.csdn.net/feng88724/article/details/11600375)

### 快速打开git config
```
git config [--global] --edit
```
[Win10下修改git全部配置文件方法](http://blog.csdn.net/shrimpcolo/article/details/49302619)
[git config命令使用](http://blog.csdn.net/zxncvb/article/details/22153019)

### 追其他新的远程仓库地址
```
 git remote set-url --add origin GiteeProfileUrl
```
[Git配置同时推送到GitHub和码云](https://blog.csdn.net/qq_28975017/article/details/78976827)

## 仓库初始化
###  创建新的本地仓库并提交
```
//create a new repository on the command line
echo "# qy-console" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/iwinder/qy-console.git
git push -u origin master
```
### 将已存在的仓库向远程提交
```
// push an existing repository from the command line
git remote add origin https://github.com/iwinder/qy-console.git
git push -u origin master
```

## 修复 工程远端库地址
```
# 方案一
git remote set-url origin \
https://github.com/iwinder/nightbook2.0.git \
https://gitee.com/windcoderqy/nightbook2.0.git

# 方案二
git remote set-url origin git@gitee.com:windcoderqy/nightbook2.0.git


// 更新分支等
git fetch -ap

// 查看更新结果
git remote -v

```

## 篡改 name and email
```
git filter-branch -f --env-filter "GIT_AUTHOR_NAME='Newname'; GIT_AUTHOR_EMAIL='newemail'; GIT_COMMITTER_NAME='Newname'; GIT_COMMITTER_EMAIL='newemail';" HEAD
```

[Git 今天终于用了一下 git filter-branch ，爽死了](https://ruby-china.org/topics/7820)
[Git 修改 commit 的作者信息](https://segmentfault.com/a/1190000008828569)


## Git提交记住用户名和密码
```
git config --global credential.helper store
```
[Git提交记住用户名和密码](https://blog.csdn.net/youanyyou/article/details/78992990)


## 分支

### 查看分支
```
//查看本地和远程所有分支
git branch -a
//查看远程所有分支
git branch -r
//查看本地分支
git branch
```
[git branch用法总结，查看、新建、删除、重命名](http://blog.csdn.net/afei__/article/details/51567155)

### 更新分支
#### 本地的远程分支列表
```
git remote update origin --prune
```
#### 更新分支
```
git fetch
```
上面命令将某个远程主机的更新，全部取回本地。默认情况下，git fetch取回所有分支的更新。

切分支前需常 
```
git fetch -ap
```

### 克隆分支

```
git clone -b 分支名 仓库地址
```
[使用git克隆指定分支的代码](https://www.cnblogs.com/nylcy/p/6569284.html)


###  切换分支
#### 本地
```
git checkout feature/training
```

#### 远程

```
//检出并切换分支
git checkout -b feature/training origin/feature/training
```
### 合并分支
#### 合并远程
```
 git merge origin/release/v0.6.0.alpha
```
#### 合并本地
同远程
```
 git merge release/v0.6.0.alpha
```

### 撤销合并
```
git merge --abort
```
该命令仅仅在合并后导致冲突时才使用。git merge --abort将会抛弃合并过程并且尝试重建合并前的状态。但是，当合并开始时如果存在未commit的文件，git merge --abort在某些情况下将无法重现合并前的状态。（特别是这些未commit的文件在合并的过程中将会被修改时）

[git-merge完全解析](https://blog.csdn.net/t3/article/details/77069523)

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

### 删除分支

#### 本地
```
git branch -d branch-name
```
若提示：
```
error: The branch 'newTesting' is not fully merged.  
If you are sure you want to delete it, run 'git branch -D newTesting'.  
```
由于这些分支中还包含着尚未合并进来的工作成果，所以简单地用 Git branch -d 删除该分支会提示错误，因为那样做会丢失数据。

若确实想要删除该分支上的改动，可以用**大写的删除选项 -D 强制执行**，即：
```
git branch -D branch-name
```
[Git之（四）分支管理](https://blog.csdn.net/w372426096/article/details/78518259)

#### 远程

1. 方案一：

```
git branch -r -d origin/branch-name  
git push origin :branch-name  
```

2. 方案二：

```
 git push origin --delete bug_xzx
```
[git命令行删除远程分支](http://blog.csdn.net/furzoom/article/details/53002699)

### 回退分支
#### 还原某个特定的文件到之前的版本
```
//获得文件对应的commit历史记录
 git log com/windcoder.md
//获取需要回退版本的hash，checkout 对应版本，如hash为d1bd8c35ff58c19ecdc5238e076ed468fa323a9b，则执行:
git checkout d1bd8c35ff58c19ecdc5238e076ed468fa323a9b com/windcoder.md
// commit更改
git commit -m "revert to previous version"
//push 提交
git push
```
[git还原某个特定的文件到之前的版本](https://blog.csdn.net/l_yangliu/article/details/53197706)

#### 回退到某个历史版本
```
//获取历史记录,获取hash,如d1bd8c35ff58c19ecdc5238e076ed468fa323a9b
git log 
//回退
git reset --hard 139dcfaa558e3276b30b6b2e5cbbb9c00bbdca96
//把修改推到远程服务器
git push -f -u origin master  
```
[ git回退到某个历史版本](https://blog.csdn.net/newjueqi/article/details/49098123)

### 提交本地分支到远程分支 
- 创建本地分支：$ git branch [name] ----注意新分支创建后不会自动切换为当前分支

- 切换分支：$ git checkout [name]

- 创建新分支并立即切换到新分支：$ git checkout -b [name]

- 删除分支：$ git branch -d [name] ---- -d选项只能删除已经参与了合并的分支，对于未有合并的分支是无法删除的。如果想强制删除一个分支，可以使用-D选项

- 合并分支：$ git merge [name] ----将名称为[name]的分支与当前分支合并

- 创建远程分支(本地分支push到远程)：$ git push origin [name]

[git提交本地分支到远程分支](https://www.cnblogs.com/springbarley/archive/2012/11/03/2752984.html)

## git add小结
|命令|作用|
|---|---|
|git add -A|  提交所有变化|
|git add -u | 提交被修改(modified)和被删除(deleted)文件，不包括新文件(new)|
|git add .  |提交新文件(new)和被修改(modified)文件，不包括被删除(deleted)文件|

[git add -A 和 git add . 的区别](https://www.cnblogs.com/skura23/p/5859243.html)




## Git提交分支
```
 git push --set-upstream origin feature/v0.1.1
```

## 问题及解决
### git签出远程分支问题解决

使用命令
```
git checkout -b develop origin/develop  
```
签出远程分支，出现以下错误：

```
fatal: Cannot update paths and switch to branch 'develop' at the same time.  
Did you intend to checkout 'origin/develop' which can not be resolved as commit?  
```

解决方法：
```
$ git fetch  
$ git checkout -b develop origin/develop  
```
[git签出远程分支问题解决](http://blog.csdn.net/wmzy1067111110/article/details/13512763)


[Git远程操作详解](http://www.ruanyifeng.com/blog/2014/06/git_remote.html)

### push问题

```
fatal: The current branch develop has no upstream branch.
To push the current branch and set the remote as upstream, use

 git push --set-upstream origin develop

```

### 修改git提交的注释(commit message)的方法
提交并且没有push到远程分支，可用以下命令直接修改：
```
git commit --amend -m "your new message"
```
[修改git提交的注释(commit message)的方法](https://www.cnblogs.com/zhangjiali/p/7150523.html)


### pull失败，解决fatal: refusing to merge unrelated histories
#### 问题：
```
Everything up-to-date
To https://github.com/iwinder/nightbook-java.git
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'https://github.com/iwinder/nightbook-java.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.

```
#### 解决方案
```
git pull origin master --allow-unrelated-histories
```

#### 原因：
先pull，因为两个仓库不同，发现refusing to merge unrelated histories，无法pull

因为他们是两个不同的项目，要把两个不同的项目合并，git需要添加一句代码，在git pull，

这句代码是在git 2.9.2版本发生的，最新的版本需要添加--allow-unrelated-histories

[如何去解决fatal: refusing to merge unrelated histories](https://blog.csdn.net/m0_37402140/article/details/72801372)

### dst refspec xxx matches more than one.

#### 问题

```
 dst refspec xxx matches more than one.
```

#### 解决方案


1.删除分支

```
git push origin :heads/xxx

```
2.删除tag

```git
git push origin :tags/xxx
```

#### 原因：

这是Git提交时，远端出现了 xxx 同名的branch 或者 tag。删除即可解决~

### 

```shell
Auto packing the repository in background for optimum performance.
See "git help gc" for manual housekeeping.
Counting objects: 132138

```

[git操作提示 Auto packing the repository for optimum performance. You may also run "git gc" manually. See](https://blog.csdn.net/ronnyjiang/article/details/54893190)
