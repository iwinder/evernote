--- 
title: Java中的域与变量
tags: Java,基础
grammar_cjkRuby: true
---
Java中的Field译为“字段”，也译为“域”，Field和成员变量（Member Variable）是相同的。所以域是变量中的一种。

关于Java中的变量，官方文档中如是说：
>There are several kinds of variables:
> - Member variables in a class—these are called fields.
> - Variables in a method or block of code—these are called local variables.
> - Variables in method declarations—these are called parameters.

翻译过来即：
>Java中有如下几种变量：
>- 类中的成员变量——称为字段（亦即 “域”）
>- 一个方法或代码块中的变量——称为局部变量（亦即 “本地变量”）
>- 在方法声明中的变量——称为参数

### 成员变量

包含：类变量（也称静态变量、静态域）和实例变量（也称实例域、非静态域）。

#### 类变量

由static修饰，每个类的实例共享一个类变量，它位于内存中的一个固定位置。任何对象都可以改变类变量的值，但是也可以在不创建类的实例的情况下操作类变量。

#### 实例变量
当一个类实例化多个对象时，它们都有自己独立的实例变量副本。每个对象都有自己的这些变量的值，存储在不同的内存位置。

## 参考资料

主要是以下这两篇官方文档

[Declaring Member Variables](https://docs.oracle.com/javase/tutorial/java/javaOO/variables.html)

[Understanding Class Members](https://docs.oracle.com/javase/tutorial/java/javaOO/classvars.html)

最开始搜到这篇

[JAVA中的域，静态域，实例域](https://www.cnblogs.com/jerry007/archive/2013/01/18/java%E4%B8%AD%E5%9F%9F.html)
排版有点乱看不下去，里面有个“域的初始化”可以看看。

然后看到这篇
[Java中字段、域与成员变量关系](http://blog.csdn.net/u013632190/article/details/50662643)
联想到其他变量，忽然又感觉哪里不对劲了。

看到这篇
[ java中的域是什么？](http://blog.csdn.net/iaiti/article/details/38794007)
里面翻译的例子可以参考看下，但最后括号中关于类变量和实例变量的理解可以无视，因为是错的。。

同时看到
[域与变量的区别是什么](http://bbs.csdn.net/topics/390488364)
里面有一句“域是变量的一种”。

然后同时看到
[java中字段（也叫域）、成员变量和属性有什么区别，请前辈指教。我觉得起不一样的名字 肯定会有所区别的?](https://www.zhihu.com/question/23675337)
从里面找到官方文档中的出处，最终解惑
