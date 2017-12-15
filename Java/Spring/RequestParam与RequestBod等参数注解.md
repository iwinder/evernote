--- 
title:  RequestParam与RequestBod等参数注解简析
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

### 为空的@RequestParam
非```application/x-www-form-urlencoded```和```multipart/form-data```等协议时@RequestParam获取不到值的原因要追溯到tomcat的request请求处理中。

此处所用源码为apache-tomcat-9.0.2-src。

上面说了@RequestParam实际调用的是Request.getParameter()取值，在tomcat中执行的是
```java
	/**
     * @return the value of the specified request parameter, if any; otherwise,
     * return <code>null</code>.  If there is more than one value defined,
     * return only the first one.
     *
     * @param name Name of the desired request parameter
     */
    @Override
    public String getParameter(String name) {

        if (!parametersParsed) {
            parseParameters();
        }

        return coyoteRequest.getParameters().getParameter(name);

    }
```
parametersParsed是一个为false的全局变量，由于是线程的问题。parseParameters()针对这个请求应该只会执行一次。这里关键代码有两个，其一如下：
```java
			String contentType = getContentType();
            if (contentType == null) {
                contentType = "";
            }
            int semicolon = contentType.indexOf(';');
            if (semicolon >= 0) {
                contentType = contentType.substring(0, semicolon).trim();
            } else {
                contentType = contentType.trim();
            }

            if ("multipart/form-data".equals(contentType)) {
                parseParts(false);
                success = true;
                return;
            }

            if (!("application/x-www-form-urlencoded".equals(contentType))) {
                success = true;
                return;
            }
```
这段代码不难理解，首先会得到请求类型。从上述代码可以得出：

- 当contenttype有多种时，只会取第一种。比如contenttype为```application/json;application/x-www-form-urlencoded```，最终是```application/json```。

- 当contenttype不为```application/x-www-form-urlencoded```，直接return掉了。也就是说 ***关键代码二不执行*** 了。

- 当contenttype为```multipart/form-data```时，parseParts()方法里使用的解析文件的框架是apache自带的fileupload。

好了，以下贴出关键二的代码：
```java
 readPostBody(formData, len)
 parameters.processParameters(formData, 0, len);
 ```
readChunkedPostBody方法，其实就是request.getInputStream().read()方法，将请求的数据通过流的方式读取出来。
 
processParameters()是在Parameters类里面的方法，做的工作就是对请求的数据，做key与value的拆分，然后存放进一个名叫paramHashValues的Map中。后续的request.getParameter取的就是paramHashValues里面的数据。

由于上述分析的contenttype不为form-data的和x-www-form-urlencoded的不会执行关键二的代码，所以对于请求类型为application/json通过request.getParameter得到的数据为空。

## 参考资料
[tomcat源码---->request的请求参数分析](http://www.cnblogs.com/huhx/p/baseusewebparameter1.html)
[解析Spring中的ResponseBody和RequestBody](https://www.cnkirito.moe/2017/08/30/%E8%A7%A3%E6%9E%90Spring%E4%B8%AD%E7%9A%84ResponseBody%E5%92%8CRequestBody/)