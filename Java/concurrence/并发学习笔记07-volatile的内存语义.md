## volatile的特性

对volatile变量的单个读/写，可看成是使用同一个锁对这些单个读/写操作做了同步。如示例：

```java
class VolatileFeaturesExample {
    // 使用volatile声明64位的long型变量。
    volatile long vl = 0L;

    public void set(long l) {
        // 单个volatile变量的写
        vl = l;
    }

    public void getAndIncrement() {
        // 复合（多个）volatile变量的读/写
        vl++;
    }

    public long get() {
        // 单个volatile变量的读
        return vl;
    }
}
```
当有多个线程分别调用上面程序中的3个方法，这个程序在语义上和下面的程序等价；
```java
class VolatileFeaturesExample {
    // 64位的long型普通变量
    long vl = 0L;

    /**
    * 对单个的普通变量的写用同一个锁同步
    */
    public synchronized void set(long l){
        vl = l;
    }

    /**
    * 普通方法调用
    */
    public void getAndIncrement() {
        // 调用已同步的读方法。
        long temp = get();
        // 普通鞋操作
        temp +=1L;
        // 调用已同步的写方法
        set(temp);
    }

    /**
    * 对单个的普通变量的读用同一个锁同步
    */
    public synchronized long get() {
        return vl;
    }
}
```

由此可见，**一个volatile变量的单个读/写操作，与一个普通变量的读/写操作都是使用同一个锁来同步，它们之间的执行效果相同**。

锁的happens-before规则保证释放锁和获取锁的两个线程之间的内存可见性。

锁的语义决定了临界区代码的执行具有原子性。

简而言之，volatile变量自身具有以下特性：
- **可见性**：对一个volatile变量的读，总是能够看到（任意线程）对这个volatile变量最后的写入。
- **原子性**：对任意单个volatile变量的读/写操作具有原子性，但类似于volatile++这样的复合操作不具有原子性。


