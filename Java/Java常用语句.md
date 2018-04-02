---
title: Java常用语句 
tags: Java
grammar_cjkRuby: true
---
## 基础篇
### 判断不为空
```
import org.apache.commons.lang3.StringUtils;

StringUtils.isNotBlank(searchText)
```
### try-with-resources
JDK1.7的新语法，这种try语句可以自动执行资源关闭过程，无需再在finally中显式关闭流。
只有所有实现AutoCloseable接口的类的对象才可以由这种带资源的try语句进行管理。

从JDK7开始，Closeable扩展了AutoCloseable。因此，在JDK7中，所有实现了Closeable接口的类也都实现了AutoCloseable接口。

```
try (FileInputStream fis = new FileInputStream("windcoer_com.txt")) {
	.....
}
```
## JPA
### @PageableDefault
#### 默认排序
```
@PageableDefault(direction=Direction.DESC,sort={"displayOrder"}) Pageable pageable
```
### 条件查询
涉及翻页、动态查询、字符串小写化后模糊查询
```
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;


public Page<Award> allAwards(Award award,String searchText,Pageable pageable){
		return super.findAllWithDataRule((root,query,cb)->{
			Predicate predicate = cb.equal(root.get("isDeleted"), false);
			predicate = cb.and(predicate,cb.equal(root.get("activity").get("id"), ugcAward.getActivity().getId()));			
			if(StringUtils.isNotBlank(searchText)) {
				Predicate predicateOr = cb.or(cb.like(cb.lower(root.get("createdBy").get("username")), "%"+StringUtils.trim(searchText).toLowerCase()+"%"),
						cb.like(cb.lower(root.get("createdBy").get("displayName")), "%"+StringUtils.trim(searchText).toLowerCase()+"%"));
				predicate = cb.and(predicate,predicateOr);
			}
			return predicate; 
		}, pageable);
}
```
### 多表联查

```
  Subquery<Long> subquery = query.subquery(Long.class);
	                    Root<CategoryTreeXref> cateXrefRoot = subquery.from(CategoryTreeXref.class);
	                    subquery.select(cateXrefRoot.get("childId"));
	                    subquery.where(cb.equal(cateXrefRoot.get("parentId"), caseInfo.getCategory().getId()));
	                    predicate = cb.and(predicate,cb.in(root.get("category").get("id")).value(subquery));
}
```
### 虚拟列（@Formula）
```
import org.hibernate.annotations.Formula;

@Formula("(select count(distinct ae.user_id )  from activity_enrollment ae where ae.activity_id = id and ae.is_deleted = 0)")
	private Long enrollmentCount;
```
### 持久化前自动赋值（@PrePersist）
```
import javax.persistence.PrePersist;


@PrePersist
	public void preInsert() {
		 if(this.box == null) {
			  Box box = new Box();
			   this.setBox(box);
		  }
	}
```
## Mybatis
### ${}-属性property 

不同的属性值通过包含的实例变化. 

属性值也可以**被用在 include 元素的 refid 属性里**或者 **include 内部语句中**，例如：
```
<sql id="sometable">
  ${prefix}Table
</sql>

<sql id="someinclude">
  from
    <include refid="${include_target}"/>
</sql>

<select id="select" resultType="map">
  select
    field1, field2, field3
  <include refid="someinclude">
    <property name="prefix" value="Some"/>
    <property name="include_target" value="sometable"/>
  </include>
</select>
```
当一段sql语句仅某些几个字段等不同时，为了重用sql语句，可用此方式解决。
[Mapper XML 文件](http://www.mybatis.org/mybatis-3/zh/sqlmap-xml.html)

### 模糊查询
#### MySQL
```
 lower(name) like  concat('%',lower(#{course.name}),'%')
```
#### Oracle
```
lower(name) like  '%'||lower(#{course.name})||'%'
```


