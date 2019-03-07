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

### volatile写-读建立的happens-before关系

从JSR-133开始（即JDK5开始），volatile变量的写-读可以实现线程之间的通信。

从内存语义的角度来说，volatile的写-读与锁的释放-获取有相同的内存效果：

- volatile写和锁的释放有相同的内存语义。
- volatile读与锁的获取有相同的内存语义。

下面我们通过相关示例来分析volatile写-读建立的happens-before关系：
```java
class VolatileTest {
    int i = 0;
    volatile boolean flag = false;

    // Thread A
    public void writer(){
        i = 2;               // 1   
        flage = true;       // 2
    }
    // Thread B
    public void reader() {
        if(flage) {                                         //3
            System.out.println("---windocdr.com----: " + i); // 4
        }
    }
}
```
假设线程A执行writer()方法后，线程B执行reader()方法。根据happens-before规则，上面的程序得到如下关系：

- 已久程序次序规则：1 happens-before 2；3 happens-before 4。
- 根据volatile规则：2 happens-before 3。
- 根据happens-before的传递性规则： 1 happens-before 4；

A线程在写volatile变量之前所有可见的共享变量，在B线程读取同一个volatile之后，将立即变得对B线程可见。

### volatile写-读的内存语义

volatile的内存语义如下：
- 写的内存语义：当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量刷新到主内存。
- 读的内存语义：当读一个volatile变量时，JMM会把该线程对应的本地内存置为无效。线程接下来将从主内存中读取共享变量。

线程A写一个volatile变量，实质上是线程A向接下来将要读这个volate变量的某个线程发出了（其对共享变量所做修改的）消息。

线程B读一个volate变量，实质上是线程B接收了之前某个线程发出的（在写这个volatile变量之前对共享变量所做修改）消息。

线程A写一个volatile变量，随后线程B读取该变量，这个过程实质是线程A通过主内存向线程B发送消息。

### volatile内存语义的实现

[前文](https://windcoder.com/bingfaxuexibiji05-zhongpaixu)提到重排序分为编译器重排序和处理器重排序。
