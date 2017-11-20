---
title: JPA简介 
tags: Java,JPA
grammar_cjkRuby: true

---

## JPA架构

![JPA架构图](http://www.yiibai.com/uploads/allimg/141108/192R64962-0.png)

描述：

|单元|	描述|
|---|---|
|EntityManagerFactory|	这是一个EntityManager的工厂类。它创建并管理多个EntityManager实例。|
|EntityManager|	这是一个接口，它管理的持久化操作的对象。它的工作原理类似工厂的查询实例。|
|Entity|	实体是持久性对象是存储在数据库中的记录。|
|EntityTransaction	|它与EntityManager是一对一的关系。对于每一个EntityManager，操作是由EntityTransaction类维护。|
|Persistence	|这个类包含静态方法来获取EntityManagerFactory实例。|
|Query	|该接口由每个JPA供应商，能够获得符合标准的关系对象。|

## JPAORM组件

### 常用注解

|注解|	描述|
|---|---|
|@Entity	|声明类为实体或表。|
|@Table	|声明表名。|
|@Basic	|指定非约束明确的各个字段。|
|@Embedded	|指定类或它的值是一个可嵌入的类的实例的实体的属性。|
|@Id	|指定的类的属性，用于识别（一个表中的主键）。|
|@GeneratedValue	|指定如何标识属性可以被初始化，例如自动，手动，或从序列表中获得的值。|
|@Transient	|指定的属性，它是不持久的，即，该值永远不会存储在数据库中。|
|@Column	|指定持久属性栏属性。|
|@SequenceGenerator	|指定在@GeneratedValue注解中指定的属性的值。它创建了一个序列。|
|@TableGenerator	|指定在@GeneratedValue批注指定属性的值发生器。它创造了的值生成的表。|
|@AccessType	|这种类型的注释用于设置访问类型。如果设置@AccessType（FIELD），然后进入FIELD明智的。如果设置@AccessType（PROPERTY），然后进入属性发生明智的。|
|@JoinColumn	|指定一个实体组织或实体的集合。这是用在多对一和一对多关联。|
|@UniqueConstraint	|指定的字段和用于主要或辅助表的唯一约束。|
|@ColumnResult	|参考使用select子句的SQL查询中的列名。|
|@ManyToMany	|定义了连接表之间的多对多一对多的关系。|
|@ManyToOne	|定义了连接表之间的多对一的关系。|
|@OneToMany	|定义了连接表之间存在一个一对多的关系。|
|@OneToOne	|定义了连接表之间有一个一对一的关系。|
|@NamedQueries	|指定命名查询的列表。|
|@NamedQuery	|指定使用静态名称的查询。|

#### @GeneratedValue

用于标注主键的生成策略，通过 strategy 属性指定。默认情况下，JPA 自动选择一个最适合底层数据库的主键生成策略：SqlServer 对应 identity，MySQL 对应 auto increment。

在 javax.persistence.GenerationType 中定义了以下几种可供选择的策略：

 -  IDENTITY：采用数据库 ID自增长的方式来自增主键字段，Oracle 不支持这种方式；
 -  AUTO： JPA自动选择合适的策略，是默认选项；
 - SEQUENCE：通过序列产生主键，通过 @SequenceGenerator 注解指定序列名，MySql 不支持这种方式
 - TABLE：通过表产生主键，框架借由表模拟序列产生主键，使用该策略可以使应用更易于数据库移植。



## Spring Data JPA 
站位
[使用 Spring Data JPA 简化 JPA 开发](https://www.ibm.com/developerworks/cn/opensource/os-cn-spring-jpa/index.html)
[了解 Spring Data JPA](http://www.cnblogs.com/WangJinYang/p/4257383.html)


## 参考资料

[JPA教程](http://www.yiibai.com/jpa/)



###  泛型

得到类型的泛型信息
```
ResolvableType resolvableType1 = ResolvableType.forClass(ABService.class);
```

如果类型被Spring AOP进行了CGLIB代理,使用
```
ClassUtils.getUserClass(ABService.class)
```
得到原始类型。
[Spring4新特性——更好的Java泛型操作API](http://jinnianshilongnian.iteye.com/blog/1993608)

### 动态查询

CriteriaBuilder 安全查询创建工厂

[java-jpa-criteriaBuilder使用入门](http://blog.csdn.net/id_kong/article/details/70225032)

[JPA criteria 查询:类型安全与面向对象](https://my.oschina.net/zhaoqian/blog/133500)


### @Modify的clearAutomatically=true的作用
 它说的是可以清除底层持久化上下文，就是entityManager这个类，我们知道jpa底层实现会有二级缓存，也就是在更新完数据库后，如果后面去用这个对象，你再去查这个对象，这个对象是在一级缓存，但是并没有跟数据库同步，这个时候用clearAutomatically=true,就会刷新hibernate的一级缓存了， 不然你在同一接口中，更新一个对象，接着查询这个对象，那么你查出来的这个对象还是之前的没有更新之前的状态。
 [spring boot使用jpa的@Modify的clearAutomatically=true的作用](https://www.cnblogs.com/xjz1842/p/7217393.html)