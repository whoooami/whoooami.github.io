---
layout: post
title:  "Spring boot/Mybatis with multi DataSource."
date:   2017-03-28 17:130:00 +0800
categories: [java, springboot, codefights]
---

How to use Multi dataSource in spring boot?

**Solution:**
Talk is cheap, show you the code.

```java
@Configuration
@MapperScan(basePackages = "com.vipabc.vqs.mapper.jr", sqlSessionFactoryRef = "jrSqlSessionFactory")
public class JrDataSourceConfig {

    @Bean
    @Primary
    @ConfigurationProperties(prefix = "vipcoder.datasource.targets.jr")
    public DataSource jrDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name="jrSqlSessionFactory")
    @Primary
    public SqlSessionFactory jrSqlSessionFactory() throws Exception {
        SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
        sessionFactory.setDataSource(jrDataSource());
        return sessionFactory.getObject();
    }

    @Bean(name = "jrSqlSessionTemplate")
    @Primary
    public SqlSessionTemplate db1SqlSessionTemplate(SqlSessionFactory jrSqlSessionFactory) throws Exception {
        return new SqlSessionTemplate(jrSqlSessionFactory);
    }

    @Bean(name = "jrTransactionManager")
    @Primary
    public DataSourceTransactionManager jrTransactionManager() {
        return new DataSourceTransactionManager(jrDataSource());
    }
}
```
**Example:**

* how to use?

Just run it.

Done.