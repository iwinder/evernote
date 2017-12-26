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
### 状态:
- pending: 初始状态，不是成功或失败状态。
- fulfilled: 意味着操作成功完成。
- rejected: 意味着操作失败。

Promise 构造函数接受一个函数作为参数，该函数的两个参数分别是 resolve 方法和 reject 方法。

如果异步操作成功，则用 resolve 方法将 Promise 对象的状态，从「未完成」变为「成功」（即从 pending 变为 resolved）；

如果异步操作失败，则用 reject 方法将 Promise 对象的状态，从「未完成」变为「失败」（即从 pending 变为 rejected）。
### 属性
- ```Promise.length```

长度属性，其值总是为 1 (构造器参数的数目).
- ```Promise.prototype```

表示 Promise 构造器的原型.
### 方法
#### Promise.all(iterable)
这个方法返回一个新的promise对象，该promise对象在iterable**参数对象里所有的promise对象都成功的时候才会触发成功**，一旦有任何一个iterable里面的promise对象失败则立即触发该promise对象的失败。

这个新的promise对象在**触发成功状态**以后，会把一个包含iterable里所有promise返回值的数组作为成功回调的返回值，**顺序跟iterable的顺序保持一致**；如果这个新的promise对象**触发了失败状态**，它会把iterable里**第一个触发失败的promise对象的错误信息作为它的失败错误信息**。Promise.all方法**常被用于处理多个promise对象的状态集合**。（可以参考jQuery.when方法---译者注）

#### Promise.race(iterable)
