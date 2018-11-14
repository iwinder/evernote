---
title: ConcurrentMap的Method 
tags: JAva
grammar_cjkRuby: true
---


| Method | 是|
|---|---|
| default V compute(K key, BiFunction<? super K,? super V,? extends V> remappingFunction) |  |

## 等价转换

### compute
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