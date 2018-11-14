---
title: ConcurrentMap的Method 
tags: JAva
grammar_cjkRuby: true
---


| Method | 形式|描述| 实例|功能特性 |
|---|---|---|---| --- |
| ```compute``` |  ```default V compute(K key, BiFunction<? super K,? super V,? extends V> remappingFunction)``` |  尝试计算指定key及其当前映射值的映射(如果没有当前映射，则为null)。 | ```map.compute(key, (k, v) -> (v == null) ? msg : v.concat(msg))```| 新增/替换更新 |
| ```computeIfAbsent``` | ```default V computeIfAbsent(K key, Function<? super K,? extends V> mappingFunction)``` | 如果指定的key尚未与值相关联(或映射到null)，则尝试使用给定的映射函数计算其值，并将其输入到此映射中，除非为null。 |```map.computeIfAbsent(key, k -> new Object(f(k)));``` | 新增|
| ```computeIfPresent``` | ```default V computeIfPresent(K key,BiFunction<? super K,? super V,? extends V> remappingFunction)``` | 如果指定key的值存在且非空，则尝试计算给定key及其当前映射值V的新映射。  | ```map.computeIfPresent(1, (key, value) -> (key + 1) + value);``` |  替换更新 |
|```putIfAbsent```| ```V putIfAbsent(K key,V value)``` | 如果指定的key尚未与值相关联，请将其与给定值相关联，并返回新值；反之返回key关联的旧值。 | map.putIfAbsent(3, "d"); |
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