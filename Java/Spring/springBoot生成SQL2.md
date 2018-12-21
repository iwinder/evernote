在上篇中已经知道从Hibernate5.0.x开始通过程序生成SQL的方式已变成：
```
ServiceRegistry serviceRegistry = new StandardServiceRegistryBuilder().configure().build();
MetadataImplementor metadata = (MetadataImplementor) new MetadataSources( serviceRegistry ).buildMetadata();
SchemaExport schemaExport =  new SchemaExport(metadata);
schemaExport.create(true, true);
```
下面我们看下在springBoot中如何在启动过程中生成SQL。这里通过两种方式实现，第一种为最初版本，第二种是第一种的精简版，两种套餐可酌情使用。

## 初版
初版中通过手动注入关键been实现获取Hibernate的Config配置。分成了两个文件HibernateJavaConfig.java和GenerateDDLApplicationRunner.java

### HibernateJavaConfig.java
这个文件用于实现Hibernate的配置，类似hibernate.cfg.xml。通过显式创建been手动获取了如下对象：

- ```org.hibernate.boot.Metadata```
- ```org.hibernate.boot.registry.StandardServiceRegistry```
- ```javax.persistence.spi.PersistenceUnitInfo```

完整文件如下：

```java
package com.windcoder.qycms.core.basis.test.Hibernate.ddl;

import org.hibernate.boot.Metadata;
import org.hibernate.boot.MetadataSources;
import org.hibernate.boot.registry.StandardServiceRegistry;
import org.hibernate.boot.registry.StandardServiceRegistryBuilder;
import org.springframework.boot.autoconfigure.AutoConfigureAfter;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.domain.EntityScan;
import org.springframework.boot.autoconfigure.domain.EntityScanPackages;
import org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration;
import org.springframework.boot.autoconfigure.orm.jpa.JpaProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.orm.jpa.persistenceunit.DefaultPersistenceUnitManager;

import javax.persistence.spi.PersistenceUnitInfo;
import java.util.List;
import java.util.Map;

@Configuration
@EntityScan("com.windcoder.qycms.*")
@AutoConfigureAfter({HibernateJpaAutoConfiguration.class})
public class HibernateJavaConfig {

    /**
    *  生成元数据Metadata
    *  在这里将要解析的类加在进Metadata中
    * @param standardServiceRegistry
    * @param persistenceUnitInfo
    * @return
    */
    @ConditionalOnMissingBean({Metadata.class})
    @Bean
    public Metadata getMetadata(StandardServiceRegistry standardServiceRegistry,
                                PersistenceUnitInfo persistenceUnitInfo) {
        MetadataSources metadataSources = new MetadataSources(standardServiceRegistry);

        List<String> managedClassNames = persistenceUnitInfo.getManagedClassNames();
        for (String managedClassName : managedClassNames) {
            metadataSources.addAnnotatedClassName(managedClassName);
        }

        Metadata metadata = metadataSources.buildMetadata();
        return metadata;
    }
    /**
    *   该实例将配置信息合并到一组工作服务
    * @param jpaProperties
    * @return
    */
    @ConditionalOnMissingBean({StandardServiceRegistry.class})
    @Bean
    public StandardServiceRegistry getStandardServiceRegistry(JpaProperties jpaProperties) {
        StandardServiceRegistryBuilder ssrb = new StandardServiceRegistryBuilder();
        Map<String, String> properties = jpaProperties.getProperties();
        ssrb.applySettings(properties);

        StandardServiceRegistry ssr = ssrb.build();

        return ssr;
    }
    /**
     * PersistenceUnitInfo接口由容器实现并由创建一个javax.persistence.EntityManagerFactory时的persistence提供者使用，
     * 这里用于生成PersistenceUnitInfo的Been,用于代替persistence.xml
     * @param entityScanPackages
     * @return
     */
    @ConditionalOnMissingBean({PersistenceUnitInfo.class})
    @Bean
    public PersistenceUnitInfo getPersistenceUnitInfo(EntityScanPackages entityScanPackages) {
        List<String> packagesToScan = entityScanPackages.getPackageNames();

        DefaultPersistenceUnitManager persistenceUnitManager = new DefaultPersistenceUnitManager();

        String[] packagesToScanArr = (String[]) packagesToScan.toArray(new String[packagesToScan.size()]);
        persistenceUnitManager.setPackagesToScan(packagesToScanArr);
        persistenceUnitManager.afterPropertiesSet();

        PersistenceUnitInfo persistenceUnitInfo = persistenceUnitManager.obtainDefaultPersistenceUnitInfo();
        return persistenceUnitInfo;
    }
}

```
这里需要```@EntityScan("com.windcoder.qycms.*")```设置扫描范围，以便entityScanPackages可以自动生成默认been，不添加可能会报错找不到可用的entityScanPackages。
## GenerateDDLApplicationRunner.java
```java
package com.windcoder.qycms.core.basis.test.Hibernate.ddl;

import org.hibernate.boot.Metadata;
import org.hibernate.boot.spi.MetadataImplementor;
import org.hibernate.tool.hbm2ddl.SchemaExport;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;


import java.io.File;
import java.text.MessageFormat;
import java.text.SimpleDateFormat;
import java.util.Calendar;

/**
 * 最终用于生成SQL的类
 * Hibernate基础可参考：   
 * http://docs.jboss.org/hibernate/orm/5.0/quickstart/html/
 */
@Component
public class GenerateDDLApplicationRunner  implements ApplicationRunner {
    private Metadata metadata;


    private static final String SCHEMA_SQL2 = "db/base/update-2-ddl_%s.sql";
    public GenerateDDLApplicationRunner(Metadata metadata) {
        this.metadata = metadata;
    }
    /**
    *
    */
    public void run(ApplicationArguments args) throws Exception {
        File dropAndCreateDdlFile = new File(getOutputFilename());
        deleteFileIfExists(dropAndCreateDdlFile);


        SchemaExport schemaExport = new SchemaExport((MetadataImplementor) metadata);

        schemaExport.setDelimiter(";");
        schemaExport.setFormat(false);
        schemaExport.setOutputFile(dropAndCreateDdlFile.getAbsolutePath());

        schemaExport.execute(true, false, false, false);

    }
    /**
    *   检测输出路径将要生成的文件，若存在则删除
    */
    private void deleteFileIfExists(File dropAndCreateDdlFile) {
        if (dropAndCreateDdlFile.exists()) {
            if (!dropAndCreateDdlFile.isFile()) {
                String msg = MessageFormat.format("File is not a normal file {0}", dropAndCreateDdlFile);
                throw new IllegalStateException(msg);
            }

            if (!dropAndCreateDdlFile.delete()) {
                String msg = MessageFormat.format("Unable to delete file {0}", dropAndCreateDdlFile);
                throw new IllegalStateException(msg);
            }
        }
    }

    /**
    * 格式化输出路径
    */
    private static String getOutputFilename() {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMdd_HHmmss");
        String currentDate = sdf.format(Calendar.getInstance().getTime());

        return String.format(SCHEMA_SQL2, currentDate);
    }
}

```

## 化繁为简
通过上面两个文件的配置与实现，实现原则了解的也差不多了，现在开始做精简，整个文件：

```java
package com.windcoder.qycms.core.basis.test.Hibernate.ddl;

import org.hibernate.boot.MetadataSources;
import org.hibernate.boot.registry.StandardServiceRegistry;
import org.hibernate.boot.registry.StandardServiceRegistryBuilder;
import org.hibernate.boot.spi.MetadataImplementor;
import org.hibernate.tool.hbm2ddl.SchemaExport;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.stereotype.Component;

import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.List;
@Component
public class JPASchemaExtractor2 implements ApplicationRunner {
    private static final String SCHEMA_SQL2 = "db/base/create-ddl_2_%s.sql";

    @Autowired
    LocalContainerEntityManagerFactoryBean fb;

    @Override
    public void run(ApplicationArguments args) throws Exception {


        StandardServiceRegistry standardServiceRegistry = new StandardServiceRegistryBuilder()
                .applySettings(fb.getJpaPropertyMap())
                .build();

        MetadataSources metadata = new MetadataSources(standardServiceRegistry);
        List<String> managedClassNames = fb.getPersistenceUnitInfo().getManagedClassNames();
        for (String managedClassName : managedClassNames) {
            metadata.addAnnotatedClassName(managedClassName);
        }

        MetadataImplementor metadataImplementor = (MetadataImplementor) metadata.getMetadataBuilder().build();
        SchemaExport schemaExport = new SchemaExport(metadataImplementor);
        String outputFile = getOutputFilename();
        schemaExport.setOutputFile(outputFile);
        schemaExport.setDelimiter(";");
        schemaExport.setFormat(false);
        schemaExport.execute(true, false, false, false);
    }



    private static String getOutputFilename() {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMdd_HHmmss");
        String currentDate = sdf.format(Calendar.getInstance().getTime());

        return String.format(SCHEMA_SQL2, currentDate);
    }
}
```
通过对比可知，LocalContainerEntityManagerFactoryBean代替了之前的显式代码实现。

根据官方定义可知：

该FactoryBean根据JPA的标准容器引导程序约定创建JPA EntityManagerFactory。

这是在Spring应用程序上下文中设置共享JPA EntityManagerFactory的最强大的方法;之后可以通过依赖注入将EntityManagerFactory传递给基于JPA的DAO。

注意：可以通过配置，切换到JNDI查找或切换到LocalEntityManagerFactoryBean definition。

与LocalEntityManagerFactoryBean一样，配置设置通常根据常规JPA配置约定从驻留在类路径中的META-INF / persistence.xml配置文件中读取。

但是，这个FactoryBean更灵活，你可以覆盖persistence.xml文件的位置，指定要链接的JDBC DataSources等。此外，它允许通过Spring的LoadTimeWeaver抽象实现可插入的类检测，而不是绑定到
JVM启动时指定的特殊VM代理。

在内部，此FactoryBean解析persistence.xml文件本身并创建相应的PersistenceUnitInfo对象（包含其他配置，例如JDBC DataSources和Spring LoadTimeWeaver），以传递给选定的JPA PersistenceProvider。
这是一个完全支持标准JPA容器约定的本地JPA容器。

公开的EntityManagerFactory对象将实现如下两个接口：
- PersistenceProvider返回的底层（underlying ）原生（native ）EntityManagerFactory的所有接口；
- 由此FactoryBean组装的其他元数据的EntityManagerFactoryInfo接口；

[Class LocalContainerEntityManagerFactoryBean](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/orm/jpa/LocalContainerEntityManagerFactoryBean.html)
