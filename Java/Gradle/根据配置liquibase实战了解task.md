

[LiquiBase](http://www.liquibase.org/)是一个用于数据库重构和迁移的开源工具，通过日志文件的形式记录数据库的变更，然后执行日志文件中的修改，将数据库更新或回滚到一致的状态。支持多种运行方式，如命令行、Spring集成、Maven插件、Gradle插件等。今天我们通过配置Gradle的task来实现运行，使用其将项目中的所有sql文件整合生成一个新的sql文件。
## 准备工作
### 引入插件

首先需要在Gradle中引入liquibase的插件：[liquibase-gradle-plugin](https://github.com/liquibase/liquibase-gradle-plugin)。
由于项目使然，这里使用旧的Gradle 2.0的格式。

```
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath "org.liquibase:liquibase-gradle-plugin:2.0.1"
    }
}
apply plugin: 'org.liquibase.gradle'
```
新版Gradle 2.0+的引入格式为：
```
plugins {
  id 'org.liquibase.gradle' version '2.0.1'
}
```

设置类路径

该插件在运行任务时需要能够在类路径上找到Liquibase，而Liquibase需要能够找到数据库驱动程序，更改日志解析器等。该需求在```build.gradle```文件中通过在```dependencies```块中添加```liquibaseRuntime``` 依赖项实现。 至少需要包含Liquibase本身以及数据库驱动程序。
```
    dependencies {
        // All of your normal project dependencies would be here in addition to...
        // 必选依赖
        liquibaseRuntime 'org.liquibase:liquibase-core:3.6.2'
        liquibaseRuntime 'mysql:mysql-connector-java:5.1.34'
        // 可选依赖，当使用Groovy DSL 代替xml时才需要添加。
        liquibaseRuntime 'org.liquibase:liquibase-groovy-dsl:2.0.1'
    }
```

###  必要文件
#### **ChangeLog**
这个是数据库更改日志文件，用于记录数据库所有的更改操作，LiquiBase通过读取该文件配置生成最终SQL以及修改数据库。文件格式具体含义可参考[官方文档](http://www.liquibase.org/documentation/index.html)，这里不过多解释。
```yml
databaseChangeLog:
  - changeSet:
      id: initData 
      author: windcoder 
      changes:
        - sqlFile:
            encoding: utf8
            path: db/mysql/test1.sql
```
#### **SQL文件**
默认之前项目中无SQL文件，若有可忽略该步骤。


```
Exception in thread "main" org.hibernate.internal.util.config.ConfigurationException: Could not locate cfg.xml resource [hibernate.cfg.xml]
	at org.hibernate.boot.cfgxml.internal.ConfigLoader.loadConfigXmlResource(ConfigLoader.java:53)
	at org.hibernate.boot.registry.StandardServiceRegistryBuilder.configure(StandardServiceRegistryBuilder.java:163)
	at org.hibernate.cfg.Configuration.configure(Configuration.java:259)
	at org.hibernate.cfg.Configuration.configure(Configuration.java:245)
	at com.windcoder.qycms.utils.SchemaExportUtil.main(SchemaExportUtil.java:8)
```