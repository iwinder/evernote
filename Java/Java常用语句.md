---
title: Java常用语句 
tags: Java
grammar_cjkRuby: true
---

判断不为空
```
StringUtils.isNotBlank(searchText)
```
Jpa条件查询
```
public Page<Award> allAwards(Award award,String searchText,Pageable pageable){
		return super.findAllWithDataRule((root,query,cb)->{
			Predicate predicate = cb.equal(root.get("activity").get("id"), award.getActivity().getId());
			if(StringUtils.isNotBlank(searchText)) {
				Predicate predicateOr = cb.or(cb.like(cb.lower(root.get("createdBy").get("username")), "%"+StringUtils.trim(searchText).toLowerCase()+"%"),
						cb.like(cb.lower(root.get("createdBy").get("displayName")), "%"+StringUtils.trim(searchText).toLowerCase()+"%"));
				predicate = cb.and(predicate,predicateOr);
			}
			return predicate; 
		}, pageable);
	}
```

