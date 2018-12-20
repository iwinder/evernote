之前两篇介绍了使用Hibernate生成SQL全量脚本文件的方式，若需要生成增量脚本进行版本维护呢？想到的对于生成增量脚本的方案可归为：

- 全量脚本文件与全量脚本文件对比生成
- 全量脚本文件与数据库对比生成
- 数据库与数据库对比生成

经过实际查询，第一种方案实现基本为零，暂未找到相关实现；第二种方案可以通过Hibernate的SchemaUpdate实现，也可以通过Liquibase实现；第三种方案可以通过Liquibase实现。

本次介绍通过Hibernate的SchemaUpdate生成SQL增脚本文件的方式，与SchemaExport生成全量脚本一样也可以通过两种方式生成。不同之处在于**生成全量脚本时可以不配置数据库连接信息**,，但**生成增量脚本时必须配置数据库连接信息**，从而连接数据库，不然只有程序中的注解，缺少参照的从而无法生成增量。

## 单独main函数生成
这个和之前的SchemaExport一样，只是createData方法换成了updatData方法。之前说可能会产生报错提示的情况，当精确到类似路径时```classpath*:**/**/entity/*.class```（即所有含注解的class均在entity包中）即可避免。这里不再细谈，具体的注释可见之前的文章。

```java
package com.windcoder.qycms.core.basis.test.Hibernate.ddl;

import org.hibernate.boot.MetadataSources;
import org.hibernate.boot.registry.StandardServiceRegistry;
import org.hibernate.boot.registry.StandardServiceRegistryBuilder;
import org.hibernate.boot.spi.MetadataImplementor;
import org.hibernate.dialect.Dialect;
import org.hibernate.dialect.MySQL5InnoDBDialect;
import org.hibernate.tool.hbm2ddl.SchemaExport;
import org.hibernate.tool.hbm2ddl.SchemaUpdate;
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
    private static final String SCHEMA_UPDATE_SQL = "schema-update_%s.sql";
    /**
     * 域类路径位置（如果范围很宽，则只能找到带有@Entity的类）
     */
    private final static String PATTERN = "classpath*:**/**/entity/.class";
    private final static String PATH = "com.windcoder";


    /**
     * DB类型生成DDL
     * org.hibernate.dialect.* 包参考*
     *
     * - Oracle  Oracle10gDialect.class
     * - H2 H2Dialect.class
     * ...
     *
     */
    private final static Class<? extends Dialect> DIALECT_CLASS = MySQL5InnoDBDialect.class;

    public static void main(String[] args) {
        updatData(args);
    }

    /**
     * 生成全量SQL脚本
     * @param args
     */
    public static void updatData(String[] args){
        File propsFile = new File("./qycms-core/common/src/main/resources/application.properties");

        Properties properties = new Properties();
        try {
            properties.load(new FileInputStream(propsFile));
            Map<String, Object> settings = new HashMap<>();
            settings.put("hibernate.dialect", DIALECT_CLASS);
            settings.put("hibernate.connection.driver", properties.getProperty("spring.datasource.driver-class-name"));
            settings.put("hibernate.connection.url", properties.getProperty("spring.datasource.url"));
            settings.put("hibernate.connection.username", properties.getProperty("spring.datasource.username"));
            settings.put("hibernate.connection.password", properties.getProperty("spring.datasource.password"));
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
            SchemaUpdate schemaUpdate = new SchemaUpdate(metadataImplementor);
            schemaUpdate.setOutputFile(getOutputFilename(args));

            schemaUpdate.setDelimiter(";");
            schemaUpdate.setFormat(false);
            schemaUpdate.execute(true, false);
        } catch (IOException e) {
            e.printStackTrace();
        }
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
        return String.format(SCHEMA_UPDATE_SQL, currentDate);
    }
}

```
## 构建项目期间生成

这个和之前创建全量脚本的类似，区别之处是全量脚本的StandardServiceRegistry是通过StandardServiceRegistryBuilder直接创建的，增量的是通过
```java
LocalContainerEntityManagerFactoryBean->EntityManagerFactoryImpl(由getNativeEntityManagerFactory()强转得到)->
SessionFactoryImpl(由getSessionFactory()强转得到)->StandardServiceRegistry（由getSessionFactoryOptions().getServiceRegistry()得到）
```
完整文件如下：
```java
package com.windcoder.qycms.core.basis.test.Hibernate.ddl;


import org.hibernate.boot.MetadataSources;

import org.hibernate.boot.registry.StandardServiceRegistry;

import org.hibernate.boot.spi.MetadataImplementor;


import org.hibernate.internal.SessionFactoryImpl;
import org.hibernate.jpa.internal.EntityManagerFactoryImpl;

import org.hibernate.tool.hbm2ddl.SchemaUpdate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;

import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.stereotype.Component;


import java.text.SimpleDateFormat;
import java.util.Calendar;

import java.util.List;


/**
 * 参考资料：
 * https://stackoverflow.com/questions/34612019/programmatic-schemaexport-schemaupdate-with-hibernate-5-and-spring-4
 */
@Component
public class JPASchemaSchemaUpdate implements ApplicationRunner {
    private static final String SCHEMA_SQL2 = "db/base/update-ddl_2_%s.sql";

    @Autowired
    LocalContainerEntityManagerFactoryBean fb;


    @Override
    public void run(ApplicationArguments args) throws Exception {

        EntityManagerFactoryImpl emf=(EntityManagerFactoryImpl)fb.getNativeEntityManagerFactory();

        SessionFactoryImpl sf= (SessionFactoryImpl) emf.getSessionFactory();
        StandardServiceRegistry serviceRegistry = sf.getSessionFactoryOptions().getServiceRegistry();
        MetadataSources metadata = new MetadataSources(serviceRegistry);
        List<String> managedClassNames = fb.getPersistenceUnitInfo().getManagedClassNames();
        for (String managedClassName : managedClassNames) {
            metadata.addAnnotatedClassName(managedClassName);
        }
        MetadataImplementor metadataImplementor = (MetadataImplementor) metadata.getMetadataBuilder().build();

        SchemaUpdate schemaUpdate = new SchemaUpdate(metadataImplementor);

        schemaUpdate.setOutputFile(getOutputFilename());

        schemaUpdate.setDelimiter(";");
        schemaUpdate.setFormat(false);
        schemaUpdate.execute(true, false);

    }


    private static String getOutputFilename() {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMdd_HHmmss");
        String currentDate = sdf.format(Calendar.getInstance().getTime());

        return String.format(SCHEMA_SQL2, currentDate);
    }

}

```