上一篇我们说了使用jpa配置属性直接生成SQL全量脚本文件的方式，想重新了解可以看：[springBoot+jpa根据实体类注解生成SQL文件](https://windcoder.com/springbootjpagenjushitileizhujieshengchengsqlwenjian)。
这一篇是根据Hibernate的SchemaExport实现程序建表，具体的方案可以是写在main函数中直接执行，也可以注入在springBoot中，在项目启动时自动完成。这里首先介绍第一种。

本系列环境基于 springBoot1.5.8.RELEASE+jpa+Hibernate5.0+java8+gradle
## 初步
最开始就在想既然可以通过配置```spring.jpa.hibernate.ddl-auto=update```实现自动创建和更新数据库的表结构，就应该有办法通过程序创建全量SQL和增量SQL吧，通过搜索，找到了蛛丝马迹:

- 在**Hibernate4.x**中可直接使用：
```java
Configuration cfg = new Configuration().configure();
SchemaExport se = new SchemaExport(cfg);
se.create(true,true);
```
- 在**Hibernate5.x**中```SchemExport(Configuration)```已经失效，就需要使用：

```java
ServiceRegistry serviceRegistry = new StandardServiceRegistryBuilder().configure().build();
MetadataImplementor metadata = (MetadataImplementor) new MetadataSources( serviceRegistry ).buildMetadata();
SchemaExport schemaExport =  new SchemaExport(metadata);
schemaExport.create(true, true);
```

但是这些默认方式都需要```hibernate.cfg.xml```文件，对于本系列中本身使用注解的项目而言则无法直接使用。

## 渐进
在之后的寻找中，发现可以手动配置的这些属性：
```java
 
Map<String, Object> settings = new HashMap<>();
settings.put("hibernate.dialect", DIALECT_CLASS);
settings.put("hibernate.physical_naming_strategy",  "org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy");
settings.put("hibernate.implicit_naming_strategy", "org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy");
settings.put("hibernate.id.new_generator_mappings", false);

StandardServiceRegistry standardServiceRegistry = new StandardServiceRegistryBuilder().applySettings(settings)
        .build();
MetadataSources metadata = new MetadataSources(standardServiceRegistry);
MetadataImplementor metadataImplementor = (MetadataImplementor) metadata.getMetadataBuilder().build();
SchemaExport schemaExport = new SchemaExport(metadataImplementor);
String outputFile = getOutputFilename(args);
schemaExport.setOutputFile(outputFile);
schemaExport.setDelimiter(";");
schemaExport.create(true, false);
```

但是这个目前尚无法根据entity中注解文件生成脚本，毕竟还没设置扫描的路径。现在通过反射过滤获取所需要的class列表：

```java
String pattern = getPattern(args);
List<Class<?>> classes = getClassesByAnnotation(Entity.class, pattern);
classes.forEach(metadata::addAnnotatedClass);
```

涉及到的相关方法如下：
```java
/**
* 根据运行mian函数时的输入路径参数获取扫描路径，
* 无输出时使用默认路径PATTERN
* @param args
*/
private static String getPattern(String[] args) {
    String pattern = PATTERN;
    if(args != null && args.length >= 3
            && StringUtils.hasText(args[2])) {
        pattern = args[2];
    }
    return pattern;
}
/**
* 使用流过滤获取所需类的list
*/
private static List<Class<?>> getClassesByAnnotation(Class<? extends Annotation> annotation, String pattern) {
    return getResources(pattern).stream()
            .map(r -> metadataReader(r))
            .filter(Objects::nonNull)
            .filter(mr -> mr.getAnnotationMetadata().hasAnnotation(annotation.getName()))
            .map(mr -> entityClass(mr))
            .filter(Objects::nonNull)
            .collect(Collectors.toList());
}

/**
    * 获取与模式对应的资源信息。
    * @param pattern
    * @return
    */
private static List<Resource> getResources(String pattern) {
    PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
    Resource[] resources;
    try {
        resources = resolver.getResources(pattern);
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
    return Arrays.asList(resources);
}


private static Class<?> entityClass(MetadataReader mr) {
    String className = mr.getClassMetadata().getClassName();
    Class<?> clazz;
    try {
        clazz = Class.forName(className);
    } catch (ClassNotFoundException e) {
        System.err.printf("%s Class not found", className);
        return null;
    }
    return clazz;
}

private static MetadataReader metadataReader(Resource r) {
    MetadataReader mr;
    try {
        mr = new SimpleMetadataReaderFactory().getMetadataReader(r);
    } catch (IOException e) {
        System.err.printf(e.getMessage());
        return null;
    }
    return mr;
}
/**
*根据输入生成输出路径，无输入时使用默认路径SCHEMA_SQL
*/
private static String getOutputFilename(String[] args) {
    SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMdd_HHmmss");
    String currentDate = sdf.format(Calendar.getInstance().getTime());
    if(args != null && args.length > 0
            && StringUtils.hasText(args[0])) {
        String customSchemaName = args[0];
        if(customSchemaName.contains("%s")) {
            return String.format(customSchemaName, currentDate);
        }
        return customSchemaName;
    }
    return String.format(SCHEMA_SQL, currentDate);
}
```
该方法运行期间根据扫描的路径可能会报一些类未找到等错误，但不会影响脚本的生成，如：
```
java.lang.ClassNotFoundException: org.springframework.hateoas.config.EnableHypermediaSupport$HypermediaType

 [main] DEBUG org.springframework.core.type.classreading.AnnotationAttributesReadingVisitor - Failed to classload enum type while reading annotation metadata
java.lang.ClassNotFoundException: org.jboss.logging.annotations.Message$Format

DEBUG org.springframework.core.type.classreading.AnnotationAttributesReadingVisitor - Failed to classload enum type while reading annotation metadata
java.lang.ClassNotFoundException: org.jboss.logging.annotations.Message$Format
```
以上错误是在使用默认扫描路径```PATTERN = "classpath*:**/*.class";```的情况下可能出现的，至于原因是在getClassesByAnnotation过程中扫描了项目中的jar及其类后未能匹配到一些类等。这也反映出了扫描范围的大小在于扫描路径的选取。
## 完整代码
```java
package com.windcoder.qycms.core.basis.test.Hibernate.ddl;

import org.hibernate.boot.MetadataSources;
import org.hibernate.boot.registry.StandardServiceRegistry;
import org.hibernate.boot.registry.StandardServiceRegistryBuilder;
import org.hibernate.boot.spi.MetadataImplementor;
import org.hibernate.dialect.Dialect;
import org.hibernate.dialect.MySQL5InnoDBDialect;
import org.hibernate.tool.hbm2ddl.SchemaExport;
import org.springframework.core.io.Resource;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.core.type.classreading.MetadataReader;
import org.springframework.core.type.classreading.SimpleMetadataReaderFactory;
import org.springframework.util.FileCopyUtils;
import org.springframework.util.StreamUtils;
import org.springframework.util.StringUtils;


import javax.persistence.Entity;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.lang.annotation.Annotation;
import java.nio.charset.Charset;
import java.text.SimpleDateFormat;
import java.util.*;
import java.util.stream.Collectors;

/**
 * 可以使用JPA Entity类生成DDL查询的类
 *
 * 生成成功，但DIALECT_CLASS获取不友好。
 * https://gist.github.com/sbcoba/e4264f4b4217746767e682c61f9dc3a6
 */
public class JpaEntityDdlExport {
    /**
     * 要创建的文件名
     */
    private static final String SCHEMA_SQL = "schema_%s.sql";
 
    /**
     * 域类路径位置（如果范围很宽，则只能找到带有@Entity的类）
     */
    private final static String PATTERN = "classpath*:**/*.class";



    /**
     * 定义DB的DDL
     * org.hibernate.dialect.* 包参考*
     *
     * - Oracle  Oracle10gDialect.class
     * - H2 H2Dialect.class
     * ...
     *
     */
    private final static Class<? extends Dialect> DIALECT_CLASS = MySQL5InnoDBDialect.class;

    public static void main(String[] args) {
        createData(args);
    }


    /**
     * 生成全量SQL脚本
     * @param args
     */
    public static void createData(String[] args){
        Map<String, Object> settings = new HashMap<>();
        settings.put("hibernate.dialect", DIALECT_CLASS);
        settings.put("hibernate.physical_naming_strategy","org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy");
        settings.put("hibernate.implicit_naming_strategy","org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy");
        settings.put("hibernate.id.new_generator_mappings", false);

        StandardServiceRegistry standardServiceRegistry = new StandardServiceRegistryBuilder()
                .applySettings(settings)
                .build();

        MetadataSources metadata = new MetadataSources(standardServiceRegistry);
        String pattern = getPattern(args);
        List<Class<?>> classes = getClassesByAnnotation(Entity.class, pattern);
        classes.forEach(metadata::addAnnotatedClass);
        MetadataImplementor metadataImplementor = (MetadataImplementor) metadata.getMetadataBuilder().build();
        SchemaExport schemaExport = new SchemaExport(metadataImplementor);
        String outputFile = getOutputFilename(args);
        schemaExport.setOutputFile(outputFile);
        schemaExport.setDelimiter(";");
        schemaExport.create(true, false);
    }

    
    private static String getPattern(String[] args) {
        String pattern = PATTERN;
        if(args != null && args.length >= 3
                && StringUtils.hasText(args[2])) {
            pattern = args[2];
        }
        return pattern;
    }

    private static void appendMetaData(String outputFile, Map<String, Object> settings) {
        String charsetName = "UTF-8";
        File ddlFile = new File(outputFile);
        try {
            StringBuilder sb = new StringBuilder();
            sb.append("/* Generate Environment\n");
            for (Map.Entry<String, Object> entry : settings.entrySet()) {
                sb.append(entry.getKey().toString() + ": " + entry.getValue() + "\n");
            }
            sb.append("*/\n");
            String ddlFileContents = StreamUtils.copyToString(new FileInputStream(ddlFile), Charset.forName(charsetName));
            sb.append(ddlFileContents);
            FileCopyUtils.copy(sb.toString().getBytes(charsetName), ddlFile);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }


    private static List<Class<?>> getClassesByAnnotation(Class<? extends Annotation> annotation, String pattern) {
        return getResources(pattern).stream()
                .map(r -> metadataReader(r))
                .filter(Objects::nonNull)
                .filter(mr -> mr.getAnnotationMetadata().hasAnnotation(annotation.getName()))
                .map(mr -> entityClass(mr))
                .filter(Objects::nonNull)
                .collect(Collectors.toList());
    }

    /**
     * 获取与模式对应的资源信息。
     * @param pattern
     * @return
     */
    private static List<Resource> getResources(String pattern) {
        PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
        Resource[] resources;
        try {
            resources = resolver.getResources(pattern);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        return Arrays.asList(resources);
    }


    private static Class<?> entityClass(MetadataReader mr) {
        String className = mr.getClassMetadata().getClassName();
        Class<?> clazz;
        try {
            clazz = Class.forName(className);
        } catch (ClassNotFoundException e) {
            System.err.printf("%s Class not found", className);
            return null;
        }
        return clazz;
    }

    private static MetadataReader metadataReader(Resource r) {
        MetadataReader mr;
        try {
            mr = new SimpleMetadataReaderFactory().getMetadataReader(r);
        } catch (IOException e) {
            System.err.printf(e.getMessage());
            return null;
        }
        return mr;
    }

    private static String getOutputFilename(String[] args) {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMdd_HHmmss");
        String currentDate = sdf.format(Calendar.getInstance().getTime());
        if(args != null && args.length > 0
                && StringUtils.hasText(args[0])) {
            String customSchemaName = args[0];
            if(customSchemaName.contains("%s")) {
                return String.format(customSchemaName, currentDate);
            }
            return customSchemaName;
        }
        return String.format(SCHEMA_SQL, currentDate);
    }
}

```

