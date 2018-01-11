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
>类中的成员变量——称为字段（亦即 “域”）
>一个方法或代码块中的变量——称为局部变量（亦即 “本地变量”）
>在方法声明中的变量——称为参数

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
