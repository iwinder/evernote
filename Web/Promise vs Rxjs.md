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
- pending: 初始状态，不是成功或失败状态。
- fulfilled: 意味着操作成功完成。
- rejected: 意味着操作失败。

Promise 构造函数接受一个函数作为参数，该函数的两个参数分别是 resolve 方法和 reject 方法。

如果异步操作成功，则用 resolve 方法将 Promise 对象的状态，从「未完成」变为「成功」（即从 pending 变为 resolved）；

如果异步操作失败，则用 reject 方法将 Promise 对象的状态，从「未完成」变为「失败」（即从 pending 变为 rejected）。

