---
title: Hibernate注解
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

## Enumerated
- 实体Entity中通过@Enumerated标注枚举类型。
- @Enumerated(value=EnumType.ORDINAL)采用枚举类型的序号值与数据库进行交互， 当我们将对象gt保存到数据库中的时候，数据库中存储的数值是BLUE在Color枚举 
定义中的序号1(序号从零开始)；
- @Enumerated(value=EnumType.STRING)采用枚举类型与数据库进行交互， 
此时数据库的数据类型需要是NVACHAR2等字符串类型，例如在实际操作中 

## @Transient
表示该属性并非一个到数据库表的字段的映射,ORM框架将忽略该属性.

如果一个属性并非数据库表的字段映射，就务必将其标示为@Transient，否则ORM框架默认其注解为@Basic；

[Hibernate JPA中@Transient、@JsonIgnoreProperties、@JsonIgnore、@JsonFormat、@JsonSerialize等注解解释](http://blog.csdn.net/kevinxxw/article/details/51381544)
## @Formula
动态语句
## @DynamicInsert
设置为true,设置为true,表示insert对象的时候,生成动态的insert语句,如果这个字段的值是null就不会加入到insert语句当中.默认false。
## @DynamicUpdate属性

设置为true,设置为true,表示update对象的时候,生成动态的update语句,如果这个字段的值是null就不会被加入到update语句中,默认false。

## @ColumnDefault
默认值


