---
title: Java笔记 
tags: Java
grammar_cjkRuby: true
---

## 保留两位小数
```
Double z =12.22222d;
Math.round(z*100)/100.0;
```

## Long判等源码

```
    public boolean equals(Object obj) {
        if (obj instanceof Long) {
            return value == ((Long)obj).longValue();
        }
        return false;
    }
```

可见Long类型的判等本身就是对值的判等，故不需要对Long做手动拆箱（即b.longValue()）操作：
```
Long a = 1L;
Long b = 3L;
a.equals(b); //这样既可
a.equals(b.longValue()); //无需这样
```

## list.contains(o)源码
此处以ArrayList的contains为例，可见当为Long时，该方法调用是equals作对比，而equals已自动拆箱，故无需再手动拆箱。
```
 public boolean contains(Object o) {
    return indexOf(o) >= 0;
 }

public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

## String 重载“+”
内部会自动优化，创建一个StringBuilder对象，通过其append()方法连接，最后调用toStrong()方法返回。适用于简单拼接，不适用于for循环中的拼接。

## java.lang.Math.ceil(double a)
java.lang.Math.ceil(double a) 返回最小的(最接近负无穷大)double值，该值大于或等于参数，并等于某个整数。特殊情况：

如果参数值已经等于某个整数，那么结果是一样的说法。

如果参数为NaN或无穷大，正零或负零，那么结果是一样的说法。

如果参数值是大于零，但大于-1.0，则结果为负零。

需要注意的是Math.ceil(x)的值是完全-Math.floor(-x)的值一样。

### 实例
```
package com.windcoder;

import java.lang.*;

public class MathDemo {

   public static void main(String[] args) {

      // get two double numbers
      double x = 125.9;
      double y = 0.4873;

      // call ceal for these these numbers
      System.out.println("Math.ceil(" + x + ")=" + Math.ceil(x));
      System.out.println("Math.ceil(" + y + ")=" + Math.ceil(y));
      System.out.println("Math.ceil(-0.65)=" + Math.ceil(-0.65));

   }
}
```
#### 结果
```
Math.ceil(125.9)=126.0
Math.ceil(0.4873)=1.0
Math.ceil(-0.65)=-0.0
```
[java.lang.Math.ceil(double a)方法实例](https://www.yiibai.com/java/lang/math_ceil.html)
#### 扩展
用于手动分页
```
//countUser为总条数，pages为页数
int pages = (int) Math.ceil(Double.valueOf(countUser) / 1000);
```