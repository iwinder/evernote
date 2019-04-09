
由于具体关注的内容的特殊性，如无特殊注明，本文讨论均基于Java8。

## 不可变

String对象是不可变的。每次修改都是创建了一个全新的String对象，以包含修改后的字符串内容，最初的String对象在原处丝毫未动。

对一个方法而言，参数是为该方法提供信息的，而不是想让该方法改变自己的。

- String类是final的，不可被继承。
- String类的本质是字符数组char[], 并且其值不可改变。即：```private final char value[];```
- String类对象有个特殊的创建的方式，直接赋值，如'''String x = "abc"```, 字面量（String Literals）"abc" 就表示一个字符串对象，变量 x 指向其该字符串对象的地址，即是一个引用。
- JVM存在一个String Pool（String池/字符串常量池/全局字符串池,也有叫做string literal pool）,1.7之前处于方法区中，之后被分离出来放在了堆中。
- 两个有用的类StringBuffer和StringBuilder。前者线程安全，但比后者速度较慢。
- 1.8新出了一个StringJoiner类，，用于构造由分隔符分隔的字符序列，并可选择性地从提供的前缀开始和以提供的后缀结尾。



## 重载“+”

内部并不是创建n个String对象，而是创建了一个StringBuilder对象，通过其append()方法连接，最后调用toStrong()方法返回。

当为类似```String s = "a" + "b" + "c";```的单行操作时，编译器会执行优化，在编译时直接合成一个“abc”。

该操作适用于单行“+”操作，不适用于循环（如for等）。因为在循环中，每次循环会生成一个新的个StringBuilder对象。

循环时的手动优化：在外创建StringBuilder对象，在循环内部执行append()方法拼接字符串。

StringBuilder 是JavaSE5引入的，之前都是StringBuffer。后者是线程安全的，因此开销会大些，所以在javaSE5及以后中，字符串操作应该还会更快一点。



## 创建

### 创建方式

创建字符串的方式很多，归纳起来有三类：

- 使用new关键字创建字符串，比如String s1 = new String("abc");
- 直接指定。比如String s2 = "abc";
- 使用串联生成新的字符串。比如String s3 = "ab" + "c"; 

### 分析创建

下面一起看下在创建与运行时内部具体发生了些什么。

#### 示例1

```java
public class StringDemo1 {
    public static void main(String[] args) {
        String s1 = new String("123");
    }
}
```

当仅运行这段代码期间，**涉及用户声明的几个String变量？**

*答案很简单*：

```
一个，就是String s。
```

**涉及的实例/对象呢？**

先说*答案*：

```
两个，一个是字符串字面量"123"所对应的、驻留（intern）在一个全局共享的字符串常量池中的实例，另一个是通过new String(String)创建并初始化的、内容与"123"相同的实例。
```

至于*原因*，要从StringDemo1类的编译说起：

当编译完成，会生成StringDemo1.class文件，该文件中，"123"会被提取并放置在class常量池中，当**JVM加载类时会通过读取该class常量池创建并驻留一个String实例**作为常量来对应"123"字面量（其引用存储在String Pool中，以下均称“字符串池”），这是一个**全局共享**的，**只有当字符串池中没有相同内容的字符串时才需要创建**。

当执行main方法中的new语句时，JVM会执行的字节码类似：

```java
0: new           #2                  // class java/lang/String
3: dup
4: ldc           #3                  // String 123
6: invokespecial #4                  // Method java/lang/String."<init>":(Ljava/lang/String;)V
9: astore_1
```

这之中出现过多少次new java/lang/String就是创建了多少个String对象，即代码```String s1 = new String("123");```每**执行一次只会创建一个实例对象**。

```dup``` 复制栈顶数值(数值不能是long或double类型的)并将复制值压入栈顶

```ldc``` 将int, float或String型常量值从常量池中推送至栈顶。

```invokespecial``` 调用实例构造器```<init>```方法， 私有方法和父类方法

>在JVM里，“new”字节码指令只负责把实例创建出来（包括分配空间、设定类型、所有字段设置默认值等工作），并且把指向新创建对象的引用压到操作数栈顶。此时该引用还不能直接使用，处于未初始化状态（uninitialized）；
>
>如果某方法a含有代码试图通过未初始化状态的引用来调用任何实例方法，那么方法a会通不过JVM的字节码校验，从而被JVM拒绝执行。
>
>能对未初始化状态的引用做的唯一一种事情就是通过它调用实例构造器，在Class文件层面表现为特殊初始化方法```<init>```。
>
>实际调用的指令是**invokespecial**，而在实际调用前要把需要的参数按顺序压到操作数栈上。
>
>在上面的字节码例子中，**压参数的指令包括dup和ldc两条**，分别把隐藏参数（新创建的实例的引用，对于实例构造器来说就是“this”）与显式声明的第一个实际参数（"123"常量的引用）压到操作数栈上

#### 示例2

```java
public class StringDemo2 {
    public static void main(String[] args) {
        String s1 = new String("123");
        String s2 = "123";
    }
}
```

### 反编译指令

javap反编译指令可查看编译后的.class文件的字节码信息，这里是做简单的使用记录，不做过多讨论：

```shell
javap -c Concatenation
```

若想查看更详细的常量池等信息，可添加```-verbose```选项，即：

```shell
javap -c -verbose Concatenation
```

```-c``` 输出类中各方法的未解析的代码，即构成 Java 字节码的指令。

```-verbose``` 输出堆栈大小、各方法的 locals 及 args 数,以及class文件的编译版本。

## StringJoiner用法简介

StringJoiner类是Java8的一个新类（还有一个新类Optional可用来解决空指针的问题），可以通过指定分隔符拼接字符串，功能与```String.join```方法类似，同时可选择性地从提供的前缀开始和以提供的后缀结尾。这里简单展示用法，不做过多讨论。

```java
StringJoiner sj = new StringJoiner(":", "[", "]");
sj.add("www").add("windcoder").add("com");
String desiredString = sj.toString();
PrintUtill.println(desiredString);
```

执行结果：

```java
[www:windcoder:com]
```

String.join()内部实现则用了StringJoiner，其源码如下：

```java
    public static String join(CharSequence delimiter, CharSequence... elements) {
        Objects.requireNonNull(delimiter);
        Objects.requireNonNull(elements);
        // Number of elements not likely worth Arrays.stream overhead.
        StringJoiner joiner = new StringJoiner(delimiter);
        for (CharSequence cs: elements) {
            joiner.add(cs);
        }
        return joiner.toString();
    }
```


## 参考资料

1. [请别再拿“String s = new String("xyz");创建了多少个String实例”来面试了吧](https://rednaxelafx.iteye.com/blog/774673)
2. [The SCJP Tip Line Strings, Literally](https://javaranch.com/journal/200409/ScjpTipLine-StringsLiterally.html)
3. [JEP 122：删除永久世代](http://openjdk.java.net/jeps/122)
4. [JDK 8  里程碑](http://openjdk.java.net/projects/jdk8/milestones)
5. [JVM指令详解（上）](https://blog.csdn.net/hudashi/article/details/7062675)
6. [jvm 几个invoke 指令](https://www.cnblogs.com/selfchange/p/7489821.html)

## 其他










http://imdalai.com/2017/11/05/JDK1-8-String%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/

https://www.cnblogs.com/quyanhui/p/3781663.html
https://www.cnblogs.com/jaylon/p/5721571.html


https://www.cnblogs.com/liangyueyuan/p/9796992.html
https://blog.csdn.net/axela30w/article/details/79167361

https://blog.csdn.net/Xlyxcar/article/details/78704768


**对象+引用**
https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html
