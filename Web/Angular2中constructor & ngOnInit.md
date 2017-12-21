---
title:  Angular2中constructor & ngOnInit
tags: Angular2
grammar_cjkRuby: true
---
这两天学Angular2，本想写个带参数的路由，结果运行时一直报```Error: No provider for String```的错误。对了好久没找到错误在哪，直到将```uname: string;```从构造函数```constructor```拿出来。身为java出身的程序员，竟然忽然无视 “构造函数”的含义，也不知道当时怎么想的。出于好奇，开始了详细搜索之旅。
## constructor

在 ES6 中就引入了类，constructor(构造函数) 是类中的特殊方法，主要用来做初始化操作，在进行类实例化操作时，会被自动调用。执行顺序在ngOnInit之前。
constructor是ES6引入类的概念后新出现的东东，是类的自身属性，并不属于Angular的范畴，所以Angular没有办法控制constructor。
本身：
```
  constructor(name) {
    console.log('Constructor initialization');
    this.name = name;
  }
```
然而程序中使用了如下写法，一时没看清，当成在那里面定义了。。
```
    constructor(
        private bookMineService: BookMineService,
        private route: ActivatedRoute,
        private location: Location,
    ) { }
```
其实下面这种是涉及到依赖注入的问题，这种写法时AngularJS会自动寻找相应关系并注入对象，比如Service需要@Injectable()装饰器标识一个类可以被注入器实例化，然后通过provider配置注入器。由于string本身等问题，不可能提供依赖注入的条件，AngularJS找不到自然报开始时的错误。

### 应用场景

在 Angular 2 中，构造函数一般用于依赖注入或执行一些简单的初始化操作。

## ngOnInit
ngOnInit 是 Angular 2 组件生命周期中的一个钩子，用于在 Angular 获取输入属性后初始化组件，该钩子方法会在第一次 ngOnChanges 之后被调用。
```
import { Component, ElementRef } from '@angular/core';

@Component({
  selector: 'my-app',
  template: `
    <h1>Welcome to Angular World</h1>
    <p>Hello {{name}}</p>
  `,
})
export class AppComponent {
  name: string = '';

  constructor(public elementRef: ElementRef) { // 使用构造注入的方式注入依赖对象
    this.name = 'Semlinker'; // 执行初始化操作
  }
}
```

### 应用场景
在项目开发中我们要尽量保持构造函数简单明了，让它只执行简单的数据初始化操作，因此我们会把其他的初始化操作放在 ngOnInit 钩子中去执行。如在组件获取输入属性之后，需执行组件初始化操作等。

## 总结
在 Angular 2 中 constructor 一般用于依赖注入或执行简单的数据初始化操作，ngOnInit 钩子主要用于执行组件的其它初始化操作或获取组件输入的属性值。

## 参考资料
[Angular 2 constructor & ngOnInit](https://segmentfault.com/a/1190000008685752)
[Angular之constructor和ngOnInit差异及适用场景](http://blog.csdn.net/u010730126/article/details/64486997)