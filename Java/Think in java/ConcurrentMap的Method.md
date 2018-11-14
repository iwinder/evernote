---
title: ConcurrentMap的Method 
tags: JAva
grammar_cjkRuby: true
---


| Method | 形式|描述|
|---|---|---|
| ```compute``` |  ```default V compute(K key, BiFunction<? super K,? super V,? extends V> remappingFunction)``` | 旧映射value为null，保存并返回新值（若新值为空，则移除该key对应的键值对）；旧值非空，则替换并返回新值（若新值为空，则移除该key对应的键值对）。 |
| ```computeIfAbsent``` | ```default V computeIfAbsent(K key, Function<? super K,? extends V> mappingFunction)``` |旧映射value为null，则存储并返回新映射的非null的值，反之返回null|
| ```computeIfPresent``` | ```default V computeIfPresent(K key,BiFunction<? super K,? super V,? extends V> remappingFunction)``` | |
## 等价转换

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