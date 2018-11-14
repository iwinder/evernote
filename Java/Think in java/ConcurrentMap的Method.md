---
title: ConcurrentMap的Method 
tags: JAva
grammar_cjkRuby: true
---


| Method | 形式|描述|
|---|---|---|
| ```compute``` |  ```default V compute(K key, BiFunction<? super K,? super V,? extends V> remappingFunction)``` |  尝试计算指定键及其当前映射值的映射(如果没有当前映射，则为null)。 |
| ```computeIfAbsent``` | ```default V computeIfAbsent(K key, Function<? super K,? extends V> mappingFunction)``` |  |
| ```computeIfPresent``` | ```default V computeIfPresent(K key,BiFunction<? super K,? super V,? extends V> remappingFunction)``` |   |
## 默认实现的等价转换

### compute

```
default V compute(K key,
                  BiFunction<? super K,? super V,? extends V> remappingFunction)
```
等价于
```
 V oldValue = map.get(key);
 V newValue = remappingFunction.apply(key, oldValue);
 if (oldValue != null ) {
    if (newValue != null)
       map.replace(key, oldValue, newValue);
    else
       map.remove(key, oldValue);
 } else {
    if (newValue != null)
       map.putIfAbsent(key, newValue);
    else
       return null;
 }
```