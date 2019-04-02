## 使用方式
Java SE5之前，在协调对共享对象的访问时可用的机制只有synchronized和volatile。Java 5.0之后，并发包中新增了Lock接口及其相关实现类。

使用synchronized关键字将隐式获取锁，简化了同步的管理。

Lock提供了一种无条件的、可轮询的、定时的以及可中断的锁获取操作，所有加锁和解锁的方法都是显式的，使用方式如下：
```java
Lock lock = new ReentrantLock();
lock.lock();
try {

} finally{
    lock.unlock();
}
```
finally块中释放锁，目的是保证在获取到锁之后，最终能释放锁。

**不要将获取锁的过程卸载try块中**，因为**如果在获取锁（自定义锁的实现）时发生了异常，异常抛出的同时，也会导致锁无故释放**。

## Lock接口提供的synchronized所不具备的特性

- **尝试非阻塞地获取锁**：当前线程尝试获取锁，如果这一时刻锁没有被其他线程获取到，则成功获取并持有锁。
- **能被中断地获取锁**：与synchronized不同，获取到锁的线程能够响应中断，当获取到锁的线程被中断时，中断异常将会被抛出，同时锁会被释放。
- **超时获取锁**：在指定的截止时间之前获取锁，若截止时间到了仍旧无法获取锁，则返回。

## Lock的API
Lock是个接口，定义了锁获取和释放的基本操作，API如下表：

| 方法名称 | 描述 |
| --- | --- |
| void lock() | 获取锁，调用该方法当前线程会获取锁，当锁获取后，从该方法返回，|
| void LockInterruptribly() throws InterruptedException | 可中断地获取锁，和lock()方法的不同之处在于该方法会响应中断，即在锁的获取中可以中断当前线程。 |
| boolean tryLock() |尝试非阻塞的获取锁，调用该方法后立即返回，如果能够获取则返回true，否则返回false。 |
| boolean tryLock(long time, TimeUnit unit) throws InterruptedExceptions | 超时的获取锁，当前线程在以下3种情况下会返回：1.当前线程在超时时间内获得了锁。2.当前线程在超时时间内被中断。3.超时时间结束，返回false。 |
| void unlock() | 释放锁。 |
| Condition newConition() | 获取等待通知组件，该组件和当前的锁绑定，当前线程只有获取了锁，才能调用该组件的eait()方法，而调用后，当前线程将释放锁。|

Lock接口的实现基本都是通过聚合了一个同步器的子类来完成线程访问控制的。