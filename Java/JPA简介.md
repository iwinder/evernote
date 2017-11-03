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
