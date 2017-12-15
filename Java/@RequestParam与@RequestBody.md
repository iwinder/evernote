--- 
title: @RequestParam与@RequestBody
tags: Spring
grammar_cjkRuby: true
---

## @RequestParam
- A） 常用来处理简单类型的绑定，通过Request.getParameter() 获取的String可直接转换为简单类型的情况（ String--> 简单类型的转换操作由ConversionService配置的转换器来完成）；因为使用request.getParameter()方式获取参数，所以可以处理get 方式中queryString的值，也可以处理post方式中 body data的值；
- B）用来处理Content-Type: 为 ```application/x-www-form-urlencoded```编码的内容，提交方式GET、POST；
- C) 该注解有两个属性： value、required； value用来指定要传入值的id名称，required用来指示参数是否必须绑定；