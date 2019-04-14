上篇介绍了一些String的基础与简单的创建方式，本篇引入它的intern()方法。

## 关于intern()方法


当一个String实例str调用**intern()方法**时，如果常量池中已经有了这个字符串，那么直接返回常量池中它的引用，如果没有，那就将它的引用保存一份到字符串常量池，然后直接返回这个引用。可参考JDK中的解释或[The Java Virtual Machine Specification, Java SE 8 Edition (§5.1)](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html#jvms-5.1)，简单来说就是一个可以手动将未存在常量池的字符串存入常量池并返回其引用的方法。


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

通过字节码可知，字符串常量池中存在"1","a"两个常量，字符串s1的内部实现是通过StringBuilder执行append方法拼接后执行toString()获得的。这一段代码不算常量池中的一共创建了3个对象，一个```StringBuilder```,一个```String 1```,一个```String a```,最后在仅堆中生成引用s1所指向的字符串“1a”，此时常量池中并无“1a”。

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
- 运行时常量池(Run-Time Constant Pool)
- 字符串常量池(String Pool/string literal pool,有时也称String池/全局字符串池)


#### 1.1 Class常量池

Class文件中的除了有类的版本、字段、方法、接口等描述信息，还有一项时常量池（Constant Pool Table），这里面主要存放两大类常量：**字面量(Literal)**与**符号引用(Symbolic References)**。

字面量比较接近于Java语言层面的常量概念，如：

- 文本字符串 
- 声明为final的常量值等。

符号引用属于编译原理方面的概念，包含下面三类常量：

- 类和接口的全限定名（Fully Qualified  Name）
- 字段的名称和描述（Descriptor）
- 方法的名称和描述

可以通过上一节中的```javap```命令查看，以上面的程序反编译为例，格式类似如下：

```java
Constant pool:
   #1 = Methodref          #15.#37        // java/lang/Object."<init>":()V
   #2 = Class              #38            // java/lang/StringBuilder
   #3 = Methodref          #2.#37         // java/lang/StringBuilder."<init>":()V
   #4 = Class              #39            // java/lang/String
   #5 = String             #40            // 1
   #6 = Methodref          #4.#41         // java/lang/String."<init>":(Ljava/lang/String;)V
   #7 = Methodref          #2.#42         // java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
   #8 = String             #43            // a
   #9 = Methodref          #2.#44         // java/lang/StringBuilder.toString:()Ljava/lang/String;
  #10 = Methodref          #4.#45         // java/lang/String.intern:()Ljava/lang/String;
  #11 = String             #46            // 1a
  #12 = Fieldref           #47.#48        // java/lang/System.out:Ljava/io/PrintStream;
  #13 = Methodref          #49.#50        // java/io/PrintStream.println:(Z)V
  #14 = Class              #51            // Others/base/StringDemo4
  #15 = Class              #52            // java/lang/Object
  #16 = Utf8               <init>
  #17 = Utf8               ()V
  #18 = Utf8               Code
  #19 = Utf8               LineNumberTable
  #20 = Utf8               LocalVariableTable
  #21 = Utf8               this
  #22 = Utf8               LOthers/base/StringDemo4;
  #23 = Utf8               main
  #24 = Utf8               ([Ljava/lang/String;)V
  #25 = Utf8               args
  #26 = Utf8               [Ljava/lang/String;
  #27 = Utf8               s1
  #28 = Utf8               Ljava/lang/String;
  #29 = Utf8               s2
  #30 = Utf8               s3
  #31 = Utf8               StackMapTable
  #32 = Class              #26            // "[Ljava/lang/String;"
  #33 = Class              #39            // java/lang/String
  #34 = Class              #53            // java/io/PrintStream
  #35 = Utf8               SourceFile
  #36 = Utf8               StringDemo4.java
  #37 = NameAndType        #16:#17        // "<init>":()V
  #38 = Utf8               java/lang/StringBuilder
  #39 = Utf8               java/lang/String
  #40 = Utf8               1
  #41 = NameAndType        #16:#54        // "<init>":(Ljava/lang/String;)V
  #42 = NameAndType        #55:#56        // append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
  #43 = Utf8               a
  #44 = NameAndType        #57:#58        // toString:()Ljava/lang/String;
  #45 = NameAndType        #59:#58        // intern:()Ljava/lang/String;
  #46 = Utf8               1a
  #47 = Class              #60            // java/lang/System
  #48 = NameAndType        #61:#62        // out:Ljava/io/PrintStream;
  #49 = Class              #53            // java/io/PrintStream
  #50 = NameAndType        #63:#64        // println:(Z)V
  #51 = Utf8               Others/base/StringDemo4
  #52 = Utf8               java/lang/Object
  #53 = Utf8               java/io/PrintStream
  #54 = Utf8               (Ljava/lang/String;)V
  #55 = Utf8               append
  #56 = Utf8               (Ljava/lang/String;)Ljava/lang/StringBuilder;
  #57 = Utf8               toString
  #58 = Utf8               ()Ljava/lang/String;
  #59 = Utf8               intern
  #60 = Utf8               java/lang/System
  #61 = Utf8               out
  #62 = Utf8               Ljava/io/PrintStream;
  #63 = Utf8               println
  #64 = Utf8               (Z)V
```
#### 1.2 运行时常量池

方法区的一部分。Class文件的常量池（上面的1.1）中的内容将在类加载后进入方法区的运行时常量池中存放。

#### 1.3 字符串常量池
HotSpot VM里，记录interned string的一个全局表叫做StringTable，它本质上就是个HashSet<String>。这是个纯运行时的结构，而且**是惰性（lazy）维护的**。注意它只存储对java.lang.String实例的引用，而不存储String对象的内容。

一般我们说一个字符串进入了全局的字符串常量池其实是说在这个StringTable中保存了对它的引用，反之，如果说没有在其中就是说StringTable中没有对它的引用。


###  字面量进入字符串常量池的时机


### 再谈ldc指令

[Java 中new String("字面量") 中 "字面量" 是何时进入字符串常量池的?](https://www.zhihu.com/question/55994121)