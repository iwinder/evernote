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

    @ConditionalOnMissingBean({StandardServiceRegistry.class})
    @Bean
    public StandardServiceRegistry getStandardServiceRegistry(JpaProperties jpaProperties) {
        StandardServiceRegistryBuilder ssrb = new StandardServiceRegistryBuilder();
        Map<String, String> properties = jpaProperties.getProperties();
        ssrb.applySettings(properties);

        StandardServiceRegistry ssr = ssrb.build();

        return ssr;
    }

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