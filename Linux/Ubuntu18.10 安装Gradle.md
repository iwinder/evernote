---
title: Ubuntu18.10 安装Gradle
tags: Ubuntu
grammar_cjkRuby: true
---
## 下载
前往https://gradle.org/releases/下载最新版本。这里用的gradle-4.10.2-all.zip
可通过浏览器或用wget命令
```
wget https://gradle.org/next-steps/?version=4.10.2&format=all
```
## 解压
```
unzip -d /home/wind/apps/Java  gradle-4.10.2-all.zip
```

## 配置环境变量
```
sudo vi /etc/profile
```
在最后添加
```
# SET PATH FOR GRADLE
export GRADLE_HOME=/home/wind/apps/Java/gradle-4.10.2
export PATH=$PATH:$GRADLE_HOME/bin
# 修改Gradle缓存文件夹路径(可选项)
export GRADLE_USER_HOME=$GRADLE_HOME/Cache/.gradle
```
保存退出， 使配置生效：
```
source /etc/profile
```