---
title: Gradle系列-依赖管理
tags: Gradle
grammar_cjkRuby: true
---

## 什么是依赖管理?
粗略的讲, 依赖管理由两部分组成：项目的  **dependencies(依赖项)** 和 **publications(发布项)**。

1. Gradle  **需要了解你的项目需要构建或运行的东西, 以便找到它们**。我们称这些传入的文件为项目的 dependencies(依赖项)。
2. Gradle  **需要构建并上传你的项目产生的东西**。我们称这些传出的项目文件为 **publications(发布项)**。

### 依赖
简言为个人理解，细说为出处。若对简言不清楚，可查看理解细说部分。
#### 简言：

1. 根据配置获取依赖关系的过程为 **dependency resolution(依赖解析)** 。
2. 项目运行时寻找到其依赖关系并使其可用的过程为**dependency resolution(依赖解析)** 。

#### 细说：

**大多数项目都不是完全独立的 ，它们需要其它项目进行编译或测试等等** 。举个例子, 为了在项目中使用 Hibernate, 在编译的时候需要在 classpath 中添加一些 Hibernate 的 jar 路径. 要运行测试的时候, 需要在 test classpath 中包含一些额外的 jar, 比如特定的 JDBC 驱动或者 Ehcache jars.

这些传入的文件构成上述项目的依赖。 Gradle 允许你告诉它项目的依赖关系, 以便找到这些依赖关系, 并在你的构建中维护它们。 

依赖关系可能需要**从远程的 Maven 或者 Ivy 仓库中下载**, 也可能是**在本地文件系统中**， 或者是**通过多项目构建另一个构建**。我们称这个过程为**dependency resolution(依赖解析)** 。

通常, 一个项目本身会具有依赖性. 举个例子, 运行 Hibernate 的核心需要其他几个类库在 classpath 中. 因此, Gradle 在为你的项目运行测试的时候, 它**会找到这些依赖关系, 并使其可用** 。 我们称之为**transitive dependencies(依赖传递)** 。

### 发布

#### 简言：
**项目的主要目的是要建立一些文件，在项目之外使用。Gradle可以负责完成这一系列任务，而这一过程称为publication(发布)**。

#### 细说：

大部分项目的主要目的是要建立一些文件，在项目之外使用。比如，你的项目产生一个 Java 库，你需要构建一个jar，可能是一个 jar 和一些文档， 并将它们发布在某处。

这些传出的文件构成了项目的发布。Gradle 当然会为你负责这个重要的工作。你声明项目的发布，Gradle 会构建并发布在某处。究竟什么是"发布"取决于你想做什么。可能你希望将文件复制到本地目录， 或者将它们上传到一个远程 Maven 或者 Ivy 库.或者你可以使用这些文件在多项目构建中应用在其它的项目中。我们称这个过程为 publication(发布)。

## 声明依赖

简单的依赖声明：
```
//应用插件,这里引入了Gradle的Java插件,此插件提供了Java构建和测试所需的一切。
apply plugin: 'java'

//仓库:指明要从哪个仓库下载jar包
repositories {
    mavenCentral()
}
//定义依赖:声明项目中需要哪些依赖
dependencies {
    compile group: 'org.hibernate', name: 'hibernate-core', version: '3.6.7.Final'
    testCompile group: 'junit', name: 'junit', version: '4.+'
}
```

## Dependency configurations

在 Gradle 里, 依赖可以组合成configurations(配置).。一个Configuration（配置）简单地说就是一系列的依赖。我们称它们为(dependency configuration)依赖配置。 你可以使用它们声明项目的外部依赖。正如我们将在后面看到，它们也被用来声明项目的发布。

### Java插件
Java插件定义了一些标准配置，形成了插件本身的类路径库。下面列一下，你可以自己去这看：[Table 23.5, “Java plugin - dependency configurations”.](http://www.gradle.org/docs/current/userguide/java_plugin.html#tab:configurations)

|configuration|含义|
|---|---|
|compile|用来编译项目源代码的依赖.|
| runtime | 在运行时被生成的类使用的依赖. 默认的, 也包含了编译时的依赖.|
|testCompile|编译测试代码的依赖. 默认的, 包含生成的类运行所需的依赖和编译源代码的依赖.|
|testRuntime|运行测试所需要的依赖. 默认的, 包含上面三个依赖.|

简单实例：

```
dependencies {
    compile module(":compile:1.0") {
        dependency ":compile-transitive-1.0@jar"
        dependency ":providedCompile-transitive:1.0@jar"
    }
    providedCompile "javax.servlet:servlet-api:2.5"
    providedCompile module(":providedCompile:1.0") {
        dependency ":providedCompile-transitive:1.0@jar"
    }
	compile `org.springframework:spring-web:4.3.4.RELEASE`
    runtime ":runtime:1.0"
    providedRuntime ":providedRuntime:1.0@jar"
    testCompile 'junit:junit:4.11'
}
```



###  War 插件

做web开发时需要servlet的依赖，但是只是编译阶段，运行时servlet依赖由servlet容器来提供。

所以Gradle的War插件也提供了两个configuration，分别是providedCompile和providedRuntime，它们对依赖的使用范围定义和compile以及runtime一致，只不过依赖的Jar包不会被加到War包里面。

如上面示例中的
```
  providedCompile "javax.servlet:servlet-api:2.5"
  providedRuntime ":providedRuntime:1.0@jar"
```


### 排除依赖


**传递依赖特性可以轻松地通过transitive参数进行开启或关闭**，上面的示例中如果要忽略spring-web的传递性依赖可以采用指定 transitive = false 的方式来关闭依赖传递特性，**也可以采用添加@jar的方式忽略该依赖的所有传递性依赖**。
```
compile("org.springframework:spring-web:4.3.4.RELEASE") {
    transitive = false
}
```
下面的语句，则可以**全局性的关闭依赖传递特性**。

```
configurations.all {
   transitive = false
}
```

#### 局部排除模块

可能需要排除一些传递性依赖中的某个模块，这时需要exclude.

**如果说@jar彻底的解决了传递问题，那么exclude则是部分解决了传递问题**。

此外，exclude还可用于但不限于以下几种情况：

1. 依赖冲突时，如果有两个依赖引用了相同jar包的不同版本时，默认情况下gradle会采用最新版本的jar包，此时可以通过排除选项来排除。

2. 运行期无需此模块的。

3. 无法正常获取到此传递依赖，远程仓库都不存在的。

4. 版权原因需要排除的。

5. 其他原因。

可以通过configuration配置或者在依赖声明时添加exclude的方式来排除指定的引用。

exclude可以接收group和module两个参数，这两个参数可以单独使用也可以搭配使用。

其中module可以理解为对应GAV中的artifactId，也就是compile group: 'org.gradle.test.classifiers', name: 'service', version: '1.0'中的中间name部分。
  
```
//方法 1.直接在configuration中排除
configurations {
    //编译期排除commons模块
    compile.exclude module: 'commons'
    //在整个构建过程中排除pkaq.tiger：share
    all*.exclude group: 'pkaq.tiger', module: 'share'
}
//方法 2.在具体的某个dependency中排除
dependencies {
    compile("pkaq.tiger:web:1.0") {
        exclude module: 'share'
    }       
}
```


## 参考资料

[跟我学Gradle-3.2:快速入门,Gradle的脚本结构](https://www.jianshu.com/p/a36389eb81a2)

[什么是依赖管理?](http://wiki.jikexueyuan.com/project/GradleUserGuide-Wiki/dependency_management_basics/what_is_dependency_management.html)

[Gradle深入与实战(转)](https://www.cnblogs.com/zdfjf/p/5262037.html)

[翻译：Gradle之依赖管理](http://somefuture.iteye.com/blog/2003535)

