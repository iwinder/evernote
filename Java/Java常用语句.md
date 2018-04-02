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
### 多表查询

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
