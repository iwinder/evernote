## 不可变

String对象是不可变的。每次修改都是创建了一个全新的String对象，以包含修改后的字符串内容，最初的String对象在原处丝毫未动。

对一个方法而言，参数是为该方法提供信息的，而不是想让该方法改变自己的。

- String类是final的，不可被继承。
- String类的本质是字符数组char[], 并且其值不可改变。即：```private final char value[];```
- JVM存在一个String Pool（String池/字符串常量池/全局字符串池,也有叫做string literal pool）,1.7之前处于方法区中，之后被分离出来放在了堆中。
- 两个有用的
## 重载“+”

内部并不是创建n个String对象，而是创建了一个StringBuilder对象，通过其append()方法连接，最后调用toStrong()方法返回。

当为类似```String s = "a" + "b" + "c";```的单行操作时，编译器会执行优化，在编译时直接合成一个“abc”。

该操作适用于单行“+”操作，不适用于循环（如for等）。因为在循环中，每次循环会生成一个新的个StringBuilder对象。

循环时的手动优化：在外创建StringBuilder对象，在循环内部执行append()方法拼接字符串。

javap反编译：
```Java
javap -c Concatenation
```

StringBuilder 是JavaSE5引入的，之前都是StringBuffer。后者是线程安全的，因此开销会大些，所以在javaSE5及以后中，字符串操作应该还会更快一点。

## 创建




http://imdalai.com/2017/11/05/JDK1-8-String%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/

https://www.cnblogs.com/quyanhui/p/3781663.html
https://www.cnblogs.com/jaylon/p/5721571.html


https://www.cnblogs.com/liangyueyuan/p/9796992.html
https://blog.csdn.net/axela30w/article/details/79167361

https://blog.csdn.net/Xlyxcar/article/details/78704768


**对象+引用**
https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html
**引用**
https://javaranch.com/journal/200409/ScjpTipLine-StringsLiterally.html