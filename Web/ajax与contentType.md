contentType选项为false用于传递文件的multipart / form-data表单。

当将contentType选项设置为false时，它会强制jQuery不添加Content-Type标头，否则将丢失边界字符串。此外，当通过多部分/表单提交文件时，必须将processData标志设置为false，否则，jQuery将尝试将FormData转换为字符串，这将失败。

要尝试解决您的问题：

您正在使用jQuery的.serialize（）方法，该方法使用标准的URL编码表示法创建文本字符串。

使用“contentType：false”时，您需要传递未编码的数据。

尝试使用“new FormData”而不是.serialize（）：

```js
  var formData = new FormData($(this)[0]);
```

通过使用console.log（），自己查看formData如何传递到php页面的区别。

```js
  var formData = new FormData($(this)[0]);
  console.log(formData);

  var formDataSerialized = $(this).serialize();
  console.log(formDataSerialized);
```

[题 Jquery / Ajax表单提交（enctype =“multipart / form-data”）。为什么'contentType：False'导致PHP中的未定义索引？](http://landcareweb.com/questions/13695/jquery-ajaxbiao-dan-ti-jiao-enctype-multipart-form-data-wei-shi-yao-contenttype-false-dao-zhi-phpzhong-de-wei-ding-yi-suo-yin)