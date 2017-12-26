---
title: Promise vs Rxjs 
tags: Angular2,Rxjs,Promise
grammar_cjkRuby: true
---

## Promise
### 语法
```
new Promise( function(resolve, reject) {...} /* executor */  );
```
变形：
```
new Promise( (resolve, reject)=>{...} )
```
### 一个 Promise有以下几种状态:
pending: 初始状态，不是成功或失败状态。
fulfilled: 意味着操作成功完成。
rejected: 意味着操作失败。