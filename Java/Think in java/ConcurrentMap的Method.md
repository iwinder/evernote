---
title: ConcurrentMap的Method
tags: Java
grammar_cjkRuby: true
---
暂且仅记录方法:```compute```， ```computeIfAbsent```，```computeIfPresent```，```putIfAbsent```

## 基础

| Method | 形式|描述| 实例|功能特性 |
|---|---|---|---| --- |
| ```compute``` |  ```default V compute(K key, BiFunction<? super K,? super V,? extends V> remappingFunction)``` |  尝试计算指定key及其当前映射值的映射(如果没有当前映射，则为null)。 | ```map.compute(key, (k, v) -> (v == null) ? msg : v.concat(msg))```| 新增/替换更新 |
| ```computeIfAbsent``` | ```default V computeIfAbsent(K key, Function<? super K,? extends V> mappingFunction)``` | 如果指定的key尚未与值相关联(或映射到null)，则尝试使用给定的映射函数计算其值，并将其输入到此映射中，除非为null。 |```map.computeIfAbsent(key, k -> new Value(f(k)));``` | 新增|
| ```computeIfPresent``` | ```default V computeIfPresent(K key,BiFunction<? super K,? super V,? extends V> remappingFunction)``` | 如果指定key的值存在且非空，则尝试计算给定key及其当前映射值V的新映射。  | ```map.computeIfPresent(1, (key, value) -> (key + 1) + value);``` |  替换更新 |
|```putIfAbsent```| ```V putIfAbsent(K key,V value)``` | 如果指定的key相关联value不存在，则与给定值相关联，并返回新值，反之返回key关联的旧值。 | ```map.putIfAbsent(3, "d");``` | 新增 |

## 默认实现的类似转换
以下仅是方法的类似转换，区别是方法是原子实现，类似转换则可能存在并发问题。

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

### computeIfAbsent

```
 if (map.get(key) == null) {
     V newValue = mappingFunction.apply(key);
     if (newValue != null)
         return map.putIfAbsent(key, newValue);
 }
```

### computeIfPresent

```
 if (map.get(key) != null) {
     V oldValue = map.get(key);
     V newValue = remappingFunction.apply(key, oldValue);
     if (newValue != null)
         map.replace(key, oldValue, newValue);
     else
         map.remove(key, oldValue);
 }
 
```

### putIfAbsent

```
 if (!map.containsKey(key))
   return map.put(key, value);
 else
   return map.get(key);
```

## 参考

[Interface ConcurrentMap<K,V>](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentMap.html#compute-K-java.util.function.BiFunction-)