---
title: Angular2笔记
tags: Angular2
grammar_cjkRuby: true
---

alexa 后添加一个问号(?)表示可选字段。

```
export class Site {
  constructor(
    public id: number,
    public name: string,
    public url: string,
    public alexa?: number
  ) {  }
}
```
[Angular 2 表单](http://www.runoob.com/angularjs2/angularjs2-forms.html)


## 模块 （module.ts ） 
每个 Angular 应用至少有一个模块（根模块），习惯上命名为AppModule，即app.module.ts

Angular 模块（无论是根模块还是特性模块）都是一个带有@NgModule装饰器的类。

### NgModule模块
NgModule是一个装饰器函数，它接收一个用来描述模块属性的元数据对象。其中最重要的属性是：


- declarations - 声明本模块中拥有的视图类。Angular 有三种视图类：组件、指令和管道。
- exports - declarations 的子集，可用于其它模块的组件模板。
- imports - 本模块声明的组件模板需要的类所在的其它模块。
- providers - 服务的创建者，并加入到全局服务列表中，可用于应用任何部分。
- bootstrap - 指定应用的主视图（称为根组件），它是所有其它视图的宿主。只有根模块才能设置bootstrap属性。

AppComponent的export语句只是用于演示如何导出的，它在这个例子中并不是必须的。根模块不需要导出任何东西，因为其它组件不需要导入根模块。

### JavaScript 模块

JavaScript 也有自己的模块系统，用来管理一组 JavaScript 对象。 它与 Angular 的模块系统完全不同且完全无关。

JavaScript 中，每个文件是一个模块，文件中定义的所有对象都从属于那个模块。 通过export关键字，模块可以把它的某些对象声明为公共的。 其它 JavaScript 模块可以使用import 语句来访问这些公共对象。