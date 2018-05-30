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
此处以ArrayList的contains为例
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