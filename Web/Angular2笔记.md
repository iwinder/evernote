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

#### Angular 模块库
Angular 提供了一组 JavaScript 模块。可以把它们看做库模块。

每个 Angular 库的名字都带有@angular前缀。

用 npm 包管理工具安装它们，用 JavaScript 的import语句导入其中某些部件。

## 组件(component.ts)
组件负责控制屏幕上的一小块区域，我们称之为视图。

我们在类中定义组件的应用逻辑，为视图提供支持。 组件通过一些由属性和方法组成的 API 与视图交互。

如：hero-list.component.ts

### 模板

我们通过组件的自带的模板来定义组件视图。模板以 HTML 形式存在，告诉 Angular 如何渲染组件。

如：hero-list.component.html

### 元数据
元数据告诉 Angular 如何处理一个类。

要告诉 Angular HeroListComponent是个组件，只要把元数据附加到这个类。

在TypeScript中，我们用装饰器 (decorator) 来附加元数据。 

如：
```
@Component({
  selector:    'app-hero-list',
  templateUrl: './hero-list.component.html',
  providers:  [ HeroService ]
})
export class HeroListComponent implements OnInit {
/* . . . */
}
```

@Component装饰器，它把紧随其后的类标记成了组件类。

@Component装饰器能接受一个配置对象， Angular 会基于这些信息创建和展示组件及其视图。

@Component的配置项包括：

- selector： CSS 选择器，它告诉 Angular 在父级 HTML 中查找<hero-list>标签，创建并插入该组件。 例如，如果应用的 HTML 包含<hero-list></hero-list>， Angular 就会把HeroListComponent的一个实例插入到这个标签中。
- templateUrl：组件 HTML 模板的模块相对地址，如前所示。
- providers - 组件所需服务的依赖注入提供商数组。 这是在告诉 Angular：该组件的构造函数需要一个HeroService服务，这样组件就可以从服务中获得英雄数据。

@Component里面的元数据会告诉 Angular 从哪里获取你为组件指定的主要的构建块。

模板、元数据和组件共同描绘出这个视图。

其它元数据装饰器用类似的方式来指导 Angular 的行为。 例如@Injectable、@Input和@Output等是一些最常用的装饰器。

