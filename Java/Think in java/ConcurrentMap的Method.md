---
title: ConcurrentMap的Method 
tags: JAva
grammar_cjkRuby: true
---


| Method | 形式|描述| 实例|
|---|---|---|
| ```compute``` |  ```default V compute(K key, BiFunction<? super K,? super V,? extends V> remappingFunction)``` |  尝试计算指定key及其当前映射值的映射(如果没有当前映射，则为null)。 | ```map.compute(key, (k, v) -> (v == null) ? msg : v.concat(msg))```|
| ```computeIfAbsent``` | ```default V computeIfAbsent(K key, Function<? super K,? extends V> mappingFunction)``` | 如果指定的key尚未与值相关联(或映射到null)，则尝试使用给定的映射函数计算其值，并将其输入到此映射中，除非为null。 |
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