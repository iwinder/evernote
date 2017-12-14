---
title: JavaWeb 笔记
tags: Java
grammar_cjkRuby: true
---


## 获取所有请求参数

HttpServletRequest request

```java
			/* *
			 * 获取所有请求参数，封装到Map中 
			*/  
            Map<String,String[]> map = (Map<String,String[]>)request.getParameterMap();  
            for(String name:map.keySet()){  
                String[] values = map.get(name);  
                System.out.println(name+"="+Arrays.toString(values));  
            } 
```
[request请求获取参数(post和get两种方式)](http://blog.csdn.net/u012110719/article/details/44672111)

