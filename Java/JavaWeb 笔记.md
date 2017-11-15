---
title: JavaWeb 笔记
tags: Java
grammar_cjkRuby: true
---


## 获取所有请求参数

HttpServletRequest request

```
		/* 
         * 获取所有请求参数，封装到Map中 
         */  
            Map<String,String[]> map = (Map<String,String[]>)request.getParameterMap();  
            for(String name:map.keySet()){  
                String[] values = map.get(name);  
                System.out.println(name+"="+Arrays.toString(values));  
            } 
```