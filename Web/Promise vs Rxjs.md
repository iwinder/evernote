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

这个新的promise对象在**触发成功状态**以后，会把一个包含iterable里所有promise返回值的数组作为成功回调的返回值，**顺序跟iterable的顺序保持一致**；如果这个新的promise对象**触发了失败状态**，它会把iterable里**第一个触发失败的promise对象的错误信息作为它的失败错误信息**。

Promise.all方法**常被用于处理多个promise对象的状态集合**。（可以参考jQuery.when方法---译者注）

#### Promise.race(iterable)

当iterable参数里的**任意一个**子promise被成功或失败后，父promise马上也会用子promise的成功返回值或失败详情作为参数调用父promise绑定的相应句柄，并返回该promise对象。

#### Promise.reject(reason)

返回一个状态为失败的Promise对象，并将给定的失败信息传递给对应的处理方法

#### Promise.resolve(value)

返回一个状态由给定value决定的Promise对象。

如果该值是一个Promise对象，则直接返回该对象；如果该值是thenable(即，带有then方法的对象)，返回的Promise对象的最终状态由then方法执行决定；否则的话(该value为空，基本类型或者不带then方法的对象),返回的Promise对象状态为fulfilled，并且将该value传递给对应的then方法。

通常而言，如果你不知道一个值是否是Promise对象，使用Promise.resolve(value) 来返回一个Promise对象,这样就能将该value以Promise对象形式使用。