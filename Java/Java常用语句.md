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

