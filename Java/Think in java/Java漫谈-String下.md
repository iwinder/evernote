上篇介绍了一些String的基础与简单的创建方式，本篇引入它的intern()方法。

## 关于intern()方法


当一个String实例str调用**intern()方法**时，如果常量池中已经有了这个字符串，那么直接返回常量池中它的引用，如果没有，那就将它的引用保存一份到字符串常量池，然后直接返回这个引用。可参考JDK中的解释或[The Java Virtual Machine Specification, Java SE 8 Edition (§5.1)](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html#jvms-5.1)，简单来说就是一个可以手动将字符串存入常量池的方法并返回其引用的方法。


#### 示例3

现在再来看另一种方式创建String的例子：

```java
public class StringDemo3 {
    public static void main(String[] args) {
        String s1 = new String("1") + new String("a"); // 1
        s1.intern();          // 2
        String s2 = "1a";    // 3
        System.out.println(s1 == s2);     // 4
        System.out.println(s1.intern() == s2); // 5
        String s3 = "1"+"a";
        System.out.println(s3 == s2); // 6
    }
}

```

**运行结果**

```shell
true
true
true
```



**解析**

语句6肯定是true，因为编译器会对"1"+"a"进行优化，使其在编译完成后成为"1a",即```String s3 = "1a";```，从而导致s3和s2均为字符串常量池中的字符串的引用，通过字节码也能看到类似情形：

```java
// ... 省略

// s2
40: ldc           #11                 // String 1a
42: astore_2

// ...省略

// s3
78: ldc           #11                 // String 1a
80: astore_3

// ...省略
```

现在我们看下```String s1 = new String("1") + new String("a");```的字节码：

```java
0: new           #2                  // class java/lang/StringBuilder
3: dup
4: invokespecial #3                  // Method java/lang/StringBuilder."<init>":()V
7: new           #4                  // class java/lang/String
10: dup
11: ldc           #5                  // String 1
13: invokespecial #6                  // Method java/lang/String."<init>":(Ljava/lang/String;)V
16: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
19: new           #4                  // class java/lang/String
22: dup
23: ldc           #8                  // String a
25: invokespecial #6                  // Method java/lang/String."<init>":(Ljava/lang/String;)V
28: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
31: invokevirtual #9                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
34: astore_1
```

通过字节码可知，常量池中存在"1","a"两个常量，字符串s1的内部实现是通过StringBuilder执行append方法拼接后执行toString()获得的。这一段代码不算常量池中的一共创建了3个对象，一个```StringBuilder```,一个```String 1```,一个```String a```,最后在仅堆中生成引用s1所指向的字符串“1a”，此时常量池中并无“1a”。

当执行``` s1.intern();```时，发现常量池中不存在“1a”，故在常量池中复制一份与s1相同的引用，即直接将s1锁指向的字符串“1a”的地址复制一份到常量池中。

此时再执行```String s2 = "1a";```，拿到的就和s1相同了，从而有了语句3和4的true。

如果将语句1和2对调，则会出现结果：

```
false
true
true
```


## 扩展

### 1. 常量池

严格来说，Java中存在着3中常量池：

- Class常量池(Class文件中的The Constant Pool)
- 字符串常量池(String Pool/string literal pool,有时也称String池/全局字符串池)
- 运行时常量池(Run-Time Constant Pool)

#### 1.1 Class常量池

这里面主要存放两大类常量：**字面量(Literal)**与**符号引用(Symbolic References)**。



[Java 中new String("字面量") 中 "字面量" 是何时进入字符串常量池的?](https://www.zhihu.com/question/55994121)