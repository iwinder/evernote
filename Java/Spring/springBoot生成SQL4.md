之前的文章介绍的都是用的jpa或者Hibernate内部方法实现的，本文引入一个第三方数据库管理工具[Liquibase](http://www.liquibase.org/)，说到数据库版本管理软件还有[Flyway](https://flywaydb.org/),但其社区版无论是功能还是用法均简单至极，完全无法和Liquibase相比。

当项目中不使用Hibernate与jpa自动生成表时，完全可以用Liquibase管理SQL脚本的版本迭代，还可以对比数据库间的差异生成对应的差异log，其用来管理版本的log文件还可以与SQL脚本文件互转，具体强大到几何去其官网领教吧。

本文的目标是创建一个gradle的task来运行Liquibase生成增量脚本，这里需要引入其gradle插件[liquibase-gradle-plugin](https://github.com/liquibase/liquibase-gradle-plugin)。

## 插件基本用法
这里仅做基础介绍，详情可见其[README.md](https://github.com/liquibase/liquibase-gradle-plugin/blob/master/README.md)文档。
### 1.引入插件
要将插件包含到Gradle构建中，只需将以下内容添加到build.gradle文件中：

```gradle
plugins {
  id 'org.liquibase.gradle' version '2.0.1'
}
```

要使用较旧的Gradle 2.0样式(多模块项目好像必须用这种)，请将以下内容添加到build.gradle中：

```gradle
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

### 2.设置classpath
该插件在运行任务时需要能够在类路径上找到Liquibase，而Liquibase需要能够在类路径中找到数据库驱动程序，更改日志解析器等。

这是通过将liquibaseRuntime依赖项添加到build.gradle文件中的依赖项块来完成的。
至少，您需要包含Liquibase本身以及数据库驱动程序:

```gradle
dependencies {
  // All of your normal project dependencies would be here in addition to...
  liquibaseRuntime 'org.liquibase:liquibase-core:3.6.1'
  liquibaseRuntime 'mysql:mysql-connector-java:5.1.34'
}
```
### 3.配置插件
Liquibase命令的参数在build.gradle文件内的liquibase块中配置。

该块包含一系列“activity”，每个activity定义一系列Liquibase参数。

“activity”中的任何方法都假定为Liquibase命令行参数。例如，在活动中包含```changeLogFile'myfile.groovy'```与```--changeLogfile = myfile.groovy```在命令行上执行的操作相同。在activity中包含```difftypes'data'```与```difftypes = data```在命令行上执行的操作相同，等等.Liquibase文档详细说明了所有有效的命令行参数。

liquibase块还有一个可选的“runList”，它确定为每个任务运行哪些活动。如果没有定义runList，Liquibase插件将运行所有活动。注意：不保证没有runList时的执行顺序。

```gradle
liquibase {
  activities {
    main {
      changeLogFile 'src/main/db/main.groovy'
      url project.ext.mainUrl
      username project.ext.mainUsername
      password project.ext.mainPassword
    }
    security {
      changeLogFile 'src/main/db/security.groovy'
      url project.ext.securityUrl
      username project.ext.securityUsername
      password project.ext.securityPassword
    }
    diffMain {
      changeLogFile 'src/main/db/main.groovy'
      url project.ext.mainUrl
      username project.ext.mainUsername
      password project.ext.mainPassword
      difftypes 'data'
    }
  }
  runList = project.ext.runList
}
```

## 多模块项目中生成增量脚本

目标将生成增量脚本的task单独抽成一个liquibase.gradle文件，在build.gradle中引入。本方案是通过对比两个数据库生成增量脚本。

### 1.设置build.gradle
buildscript中dependencies包含插件:

```gradle
buildscript {
    ...
    dependencies {
        ...

        classpath "org.liquibase:liquibase-gradle-plugin:2.0.1"
    }
}
```
不然会报如下错误,原本想只在liquibase.gradle中引入，但发现在liquibase.gradle中无效：

```gradle
Plugin with id 'org.liquibase.gradle' not found.
```

configure(rootProject)中引入插件:

```gradle
configure(rootProject) {
    
    ...

    apply from: "${rootProject.projectDir}/gradle/liquibase.gradle"
    
    ...
}
```
### 2.创建liquibase.gradle
具体文件如下：
```gradle
apply plugin: 'org.liquibase.gradle'

dependencies {
    liquibaseRuntime 'org.liquibase:liquibase-core:3.6.2'
    liquibaseRuntime 'org.liquibase:liquibase-groovy-dsl:2.0.1'
    liquibaseRuntime 'mysql:mysql-connector-java:5.1.34'
}

def buildTimestamp() {
	def date = new Date()
	def formattedDate = date.format('yyyyMMddHHmmss')
	return formattedDate
}

task diffDBSQL(){
	doFirst{
		File propsFile = new File("${rootProject.projectDir}/qycms-core/common/src/main/resources/application.properties")
        Properties properties = new Properties()
        properties.load(new FileInputStream(propsFile))
        
		def changeLogOutPutFileURL = "${rootProject.projectDir}/db/liquibase/changelog/changelog-diff-master-"+ buildTimestamp()+".yaml"
    	def outputFileURL = "${rootProject.projectDir}/db/liquibase/diffSQL-"+buildTimestamp()+".sql"
    	def referenceDriverURL =  "${properties.contains('parim.datasource.referenceDriver-class-name')}" == true ? "${properties.contains('parim.datasource.referenceDriver-class-name')}" : "${properties.get('spring.datasource.driver-class-name')}"
    	
	    liquibase {
	        activities {
	            diff {
	                driver "${properties.get('spring.datasource.driver-class-name')}"
	                url "${properties.get('spring.datasource.url')}"
	                username "${properties.get('spring.datasource.username')}"
	                password "${properties.get('spring.datasource.password')}"
	                
	                referenceDriver "${referenceDriverURL}"
	                referenceUrl "${properties.get('qy.datasource.referenceUrl')}"
	                referenceUsername  "${properties.get('qy.datasource.referenceUsername')}"
	                referencePassword  "${properties.get('qy.datasource.referencePassword')}"

	                changeLogFile "${changeLogOutPutFileURL}"   
	                outputFile "${outputFileURL}"
	            }
	            runList = "diff" // 这里代表选择哪一个配置 可用参数代替
	        }
	    }
	}
	
	doLast{
		diffChangeLog.execute()
		updateSQL.execute()
	}
}

```

### 3.配置application.properties

由上面配置可知，这里将liquibase的配置属性都集中在了application.properties文件中，故在application.properties文件中配置参考的标准数据库信息,如：

```
qy.datasource.referenceUrl=数据库地址
qy.datasource.referenceUsername=数据库用户名
qy.datasource.referencePassword=数据库密码
```

若想自定义参照数据库的驱动类名可添加使用```qy.datasource.referenceDriver-class-name```属性字段，反之默认使用 ```spring.datasource.driver-class-name```字段。

文件中已默认添加MySQL和Oracle的运行时驱动，若无法满足需求可自行修改为所需版本：

```
    liquibaseRuntime 'mysql:mysql-connector-java:5.1.46'
    liquibaseRuntime 'com.oracle:ojdbc6:12.1.0.1-atlassian-hosted'
```

默认在```${rootProject.projectDir}/db/liquibase/changelog```目录下生成diff后的changelog文件```changelog-diff-master-日期.yml```，如changelog-master-20181217172416.yaml。该文件用于之后生成SQL增量脚本。

默认在```${rootProject.projectDir}/db/liquibase```目录下生成SQL增量脚本```diffSQL-日期.sql```。

### 4.使用方法
执行```gradle diffDBSQL```即可生成所需要的增量SQL脚本文件。

该脚本仅涉及表结构，执行涉及到的DROP的语句前，请确保该语句不是因重命名字段等产生的。

## 单模块项目中生成增量脚本

单模块可以如上面多模块生成方式一样对比两个数据库，也可以对比数据库与当前程序中的注解entity生成增量脚本。原因是单模块下可以直接通过配置```liquibaseRuntime sourceSets.main.output ```依赖，将entity的classpath注入给liquibase，若多模块下有大神能找到方案，也可以使用这种方案从而免去建参照库。

这里仅介绍对比数据库与当前程序中的注解entity生成增量脚本的方案，该方案需要用到liquibase-hibernate以及一大批jpa相关的依赖，具体完整文件如下：

```gradle
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath 'org.springframework.boot:spring-boot-gradle-plugin:2.0.3.RELEASE'
    }
}

plugins {
    id 'java'
    id 'idea'
    id 'io.franzbecker.gradle-lombok' version '1.14'
    id 'org.liquibase.gradle' version '2.0.1'
}

apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group 'de.bobek'
version '1.0-SNAPSHOT'

sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    compile 'org.springframework.boot:spring-boot-starter-web'
    compile 'org.springframework.boot:spring-boot-starter-data-jpa'
    compile 'org.liquibase:liquibase-core'
    compile 'com.h2database:h2'
    liquibaseRuntime 'org.liquibase:liquibase-core:3.6.2'
    liquibaseRuntime 'org.liquibase.ext:liquibase-hibernate5:3.6'
    liquibaseRuntime 'mysql:mysql-connector-java:5.1.46'
    liquibaseRuntime 'org.springframework.boot:spring-boot:2.0.1.RELEASE'
    liquibaseRuntime 'org.springframework:spring-orm:5.0.5.RELEASE'
    liquibaseRuntime sourceSets.main.output
}

diff.dependsOn compileJava
diffChangeLog.dependsOn compileJava

liquibase {
    activities {
        main {
            driver 'com.mysql.jdbc.Driver'
            url 'jdbc:mysql://127.0.0.1:3306/my_database'
            username 'root'
            password ''
            changeLogFile 'src/main/resources/db/changelog/changelog-master.xml'
            referenceUrl 'hibernate:spring:de.bobek' +
                    '?dialect=org.hibernate.dialect.MySQL5Dialect' +
                    '&hibernate.physical_naming_strategy=org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy' +
                    '&hibernate.implicit_naming_strategy=org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy'
        }
    }
}

```

### 使用
```gradle
gradle diffChangeLog
```

该方案参考资料：

- 1.[Unable to perform diff with Spring Boot #44](https://github.com/liquibase/liquibase-gradle-plugin/issues/44)
- 2.[完整demo：spring-liquibase](https://github.com/Kr0oked/spring-liquibase)

## liquibase插件内置任务

多模块项目的解决方案下liquibase.gradle文件的doLast中的diffChangeLog和updateSQL均属于liquibase插件中的内置任务。

| 命令 | 描述 |
| --- | --- |
| update    | Updates the database to the current version. |
| updateSQL | Writes SQL to update the database to the current version to STDOUT. |
| futureRollbackSQL | Writes SQL to roll back the database to the current state after the changes in the changeslog have been applied. |
| updateTestingRollback | Updates the database, then rolls back changes before updating again. |
| listLocks | Lists who currently has locks on the database changelog. |
| dropAll | Drops all database objects owned by the user. Note that functions, procedures and packages are not dropped (Liquibase limitation) |
| releaseLocks | Releases all locks on the database changelog. |
| validate | Checks the changelog for errors. |
| diff | Writes description of differences to standard out. |
| diffChangeLog | Writes Change Log to update the database to the reference database to standard out |
| snapshot | Writes the current state of the database to standard out |
| snapshotReference | Writes the current state of the referenceUrl database to standard out |
| clearChecksums | Removes all saved checksums from the database. On next run checksums will be recomputed.  Useful for "MD5Sum Check Failed" errors. |
| changelogSync | Mark all changes as executed in the database. |
| changelogSyncSQL | Writes SQL to mark all changes as executed in the database to STDOUT. |
| markNextChangesetRan | Mark the next change set as executed in the database. |
| markNextChangesetRanSQL | Writes SQL to mark the next change set as executed in the database to STDOUT. |
| status | Outputs count (list if liquibaseCommandValue is "--verbose") of unrun change sets. |
| generateChangelog | Writes Change Log groovy to copy the current state of the database to standard out. |  
| unexpectedChangeSets | Outputs count (list if liquibaseCommandValue is "--verbose") of changesets run in the database that do not exist in the changelog. |