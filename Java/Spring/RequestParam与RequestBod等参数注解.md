--- 
title:  RequestParam与RequestBod等参数注解
tags: Spring
grammar_cjkRuby: true
---

## @RequestParam
- A） 常用来处理简单类型的绑定，通过Request.getParameter() 获取的String可直接转换为简单类型的情况（ String--> 简单类型的转换操作由ConversionService配置的转换器来完成）；因为使用request.getParameter()方式获取参数，所以可以处理get 方式中queryString的值，也可以处理post方式中 body data的值；

- B）用来处理Content-Type: 为 ```application/x-www-form-urlencoded```和```multipart/form-data```编码的内容，提交方式GET、POST；

- C) 该注解有两个属性： value、required； value用来指定要传入值的id名称，required用来指示参数是否必须绑定；

## @RequestBody

该注解常用来处理Content-Type: 不是```application/x-www-form-urlencoded```和```multipart/form-data```编码的内容，例如```application/json```, ```application/xml```等；

它是通过使用HandlerAdapter 配置的HttpMessageConverters来解析post data body，然后绑定到相应的bean上的。

因为配置有FormHttpMessageConverter，所以也可以用来处理 ```application/x-www-form-urlencoded```的内容，处理完的结果放在一个MultiValueMap<String, String>里，这种情况在某些特殊需求下使用，详情查看FormHttpMessageConverter api;

非```application/x-www-form-urlencoded```和```multipart/form-data```等协议时@RequestParam获取不到值的原因要追溯到tomcat的request请求处理中。