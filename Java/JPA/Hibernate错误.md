---
title: Hibernate错误
tags: Hibernate
grammar_cjkRuby: true
---
## save the transient instance before flushing
在Hibernate多表操作的“一对多|多对一”中，尤其是同时再遇上存在懒加载，没准什么时候会遇上这种问题。如果本身在执行添加或更新时很容易定位，把对应的非持久化更新成原本的持久化就好了。

但有时明明只是查询，也报了该错误，而且当你发现报的那个错误对象完全跟你查询中没有任何关系的时候，不要怀疑程序在误报。

因为，你**肯定在某个地方对该对象做了非持久化的赋值操作**。要在整个Session下寻找做改变的地方，而不仅仅是保存所在的查询，如果方法上使用了@Transactional，就要从最开始使用@Transactional地方找起。

当在抛出异常的位置打上断点时，发现下面两个方法中执行了n次对Session中存储的对象的查询与懒加载存储，而我们被抛出异常的对象就隐藏在这个Session存储的对象中的某个懒加载：
```
public void noCascade(）
void org.hibernate.engine.spi.new BaseCascadingAction() {...}.cascade(EventSource session, Object child, String entityName, Object anything, boolean isCascadeDeleteEnabled) throws HibernateException
```
