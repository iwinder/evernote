<!-- ---
title: NodeJS
tags: Nodejs
grammar_cjkRuby: true
--- -->

## 修改全局路径

```
/**设置全局路径**/
npm config set prefix “D:\SoftWare\NodeJS_Redis\NodeJS\node_modules\node_global”
/**设置全局cache路径**/
npm config set cache “D:\SoftWare\NodeJS_Redis\NodeJS\node_modules\node_cache”
```
## 版本升级
```
npm i -g npm to update
```
查看配置
```
npm config ls
```

## nrm管理npm源

### 一：安装nrm
```
npm install -g nrm
```
### 二：常用命令： 

- 1.nrm ls 查看已有的源 

- 2.nrm add <源名称> <源地址>  //新增源 

- 3.nrm use <源名称>.  //切换源



```
npm install console-ui-ng@0.1.83
```