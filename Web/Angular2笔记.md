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

## 数据绑定

```
<li>{{hero.name}}</li>
<app-hero-detail [hero]="selectedHero"></app-hero-detail>
<li (click)="selectHero(hero)"></li>
<input [(ngModel)]="hero.name">
```
- {{hero.name}}插值表达式在<li>标签中显示组件的hero.name属性的值。
- [hero]属性绑定把父组件HeroListComponent的selectedHero的值传到子组件HeroDetailComponent的hero属性中。
- (click) 事件绑定在用户点击英雄的名字时调用组件的selectHero方法。
- [(ngModel)]="hero.name" 双向绑定 双向数据绑定是重要的第四种绑定形式，它使用ngModel指令组合了属性绑定和事件绑定的功能。

## 指令 (directive)
Angular 模板是动态的。当 Angular 渲染它们时，它会根据指令提供的操作对 DOM 进行转换。

组件是一个带模板的指令；@Component装饰器实际上就是一个@Directive装饰器，只是扩展了一些面向模板的特性。

虽然严格来说组件就是一个指令，但是组件非常独特，并在 Angular 中位于中心地位，所以在架构概览中，我们把组件从指令中独立了出来。

还有两种其它类型的指令：结构型指令和属性 (attribute) 型指令。

它们往往像属性 (attribute) 一样出现在元素标签中， 偶尔会以名字的形式出现，但多数时候还是作为赋值目标或绑定目标出现。

### 结构型指令

通过在 DOM 中添加、移除和替换元素来修改布局。

*ngFor告诉 Angular 为heroes列表中的每个英雄生成一个<li>标签。
*ngIf表示只有在选择的英雄存在时，才会包含HeroDetail组件。

### 属性型指令

修改一个现有元素的外观或行为。 在模板中，它们看起来就像是标准的 HTML 属性，故名。

ngModel指令就是属性型指令的一个例子，它实现了双向数据绑定。 ngModel修改现有元素（一般是```<input>```）的行为：设置其显示属性值，并响应 change 事件。


## 服务

服务是一个广义范畴，包括：值、函数，或应用所需的特性。

几乎任何东西都可以是一个服务。 典型的服务是一个类，具有专注的、明确的用途。它应该做一件特定的事情，并把它做好。

服务没有什么特别属于 Angular 的特性。 Angular 对于服务也没有什么定义。 它甚至都没有定义服务的基类，也没有地方注册一个服务。

即便如此，服务仍然是任何 Angular 应用的基础。组件就是最大的服务消费者。

服务无处不在。

组件类应保持精简。组件本身不从服务器获得数据、不进行验证输入，也不直接往控制台写日志。 它们把这些任务委托给服务。

组件的任务就是提供用户体验，仅此而已。它介于视图（由模板渲染）和应用逻辑（通常包括模型的某些概念）之间。 设计良好的组件为数据绑定提供属性和方法，把其它琐事都委托给服务。

Angular  不会强制要求我们遵循这些原则。

Angular 帮助我们遵循这些原则 —— 它让我们能轻易地把应用逻辑拆分到服务，并通过依赖注入来在组件中使用这些服务。

## 依赖注入

“依赖注入”是提供类的新实例的一种方式，还负责处理好类所需的全部依赖。大多数依赖都是服务。 Angular 使用依赖注入来提供新组件以及组件所需的服务。

Angular 通过查看构造函数的参数类型得知组件需要哪些服务。

当 Angular 创建组件时，会首先为组件所需的服务请求一个注入器 (injector)。

注入器维护了一个服务实例的容器，存放着以前创建的实例。 如果所请求的服务实例不在容器中，注入器就会创建一个服务实例，并且添加到容器中，然后把这个服务返回给 Angular。 当所有请求的服务都被解析完并返回时，Angular 会以这些服务为参数去调用组件的构造函数。 这就是依赖注入 。

如果注入器还没有HeroService，它怎么知道该如何创建一个呢？

简单点说，我们必须先用注入器（injector）为HeroService注册一个提供商（provider）。 提供商用来创建或返回服务，通常就是这个服务类本身（相当于new HeroService()）。

我们可以在模块中或组件中注册提供商。

但通常会把提供商添加到根模块上，以便在任何地方都使用服务的同一个实例。

或者，也可以在@Component元数据中的providers属性中把它注册在组件层。

把它注册在组件级表示该组件的每一个新实例都会有一个服务的新实例。

需要记住的关于依赖注入的要点是：

- 依赖注入渗透在整个 Angular 框架中，被到处使用。
- 注入器 (injector) 是本机制的核心。
- 注入器负责维护一个容器，用于存放它创建过的服务实例。
- 注入器能使用提供商创建一个新的服务实例。
- 提供商是一个用于创建服务的配方。
- 把提供商注册到注入器。

##  angular-cli修改端口号【angular2】
### 方案一

找到node_modules/angular-cli/lib/config/schema.json

default值就是默认的端口

### 方案二
项目中使用如下启动：
```
ng serve --port 4201
```
### 方案三
在.angular-cli.json文件中，加入serve，如下：
```
    "defaults": {
        "styleExt": "scss",
        "component": {},
        "serve": {
            "port": 4201
        }
    }
```

## 表单
### 表单校验的值
|状态|含义|
|---|---|
|valid|表单元素有效|
|invalid|表单元素无效|
|pristine|表单元素是纯净的，用户未操作过|
|dirty|表单元素是已被用户操作过|

[表单校验的值valid、invalid、pristine和dirty](http://blog.csdn.net/lvjianyu2007/article/details/48246155)

### 自定义校验
```
  sendTimeValidator = (control: FormControl): { [s: string]: boolean } => {

    if (control.value != null && control.value < Date.now()) {
      return { sendTimebefore: true, error: true };
    }
  };
```

## 路由与导航

```
this.router.navigate(['../../'], { relativeTo: this.route });
```

[angular2系统学习 - 路由与导航(3)](http://blog.csdn.net/qq451354/article/details/53998280)

## 编译

```
 npm run build && npm run clean 
```
类似严格模式，可用于大型修改后的代码检测，一般合并之前和合并之后都用此试下工程编译是否能通过

### 一行代码过长
当出现
```
[tslint] Exceeds maximum line length of 140 (max-line-length)
```
时添加如下即可：
```
// tslint:disable-next-line:max-line-length
```
## 错误

### Expression has changed after it was checked.
[ERROR : ExpressionChangedAfterItHasBeenCheckedError
](https://stackoverflow.com/questions/44816495/error-expressionchangedafterithasbeencheckederror)

## ng-container
```
        <ng-container *ngIf="! currentUser || !isOperation(row)">
          <a href="javascript:;" class="text-secondary">管理</a>
          <span>|</span>
          <a href="javascript:;" class="text-secondary">删除</a>
        </ng-container>
```


[Angular 6集成Spring Boot 2,Spring Security,JWT和CORS](http://blog.51cto.com/7308310/2072364)