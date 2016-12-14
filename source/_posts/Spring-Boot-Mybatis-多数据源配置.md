---
title: Spring-Boot Mybatis 多数据源配置
date: 2016-12-14 17:32:36
categories:
- SpringBoot
tags:
- SpringBoot
- Java
---

1. 这篇文章涉及到的类和配置文件列表：

   - pom.xml
   - application.properties
   - MybatisProperties.java
   - AbstractDatasourceConfig.java
   - ReadDatasourceConfig.java
   - PrimaryDatasourceConfig.java


2. 添加依赖：pom.xml

   ```xml
   <dependency>
     <groupId>com.zaxxer</groupId>
     <artifactId>HikariCP</artifactId>
     <version>2.5.1</version>
   </dependency>
   <dependency>
     <groupId>org.mybatis</groupId>
     <artifactId>mybatis</artifactId>
     <version>3.4.1</version>
   </dependency>
   <dependency>
     <groupId>org.mybatis</groupId>
     <artifactId>mybatis-spring</artifactId>
     <version>1.3.0</version>
   </dependency>
   <dependency>
     <groupId>org.springframework</groupId>
     <artifactId>spring-tx</artifactId>
     <version>4.3.4.RELEASE</version>
   </dependency>
   ```
   <!-- more -->

3. 添加数据源配置项：application.properties

   ```properties
   #primary datasource setting
   datasource.primary.name=dmsdb_primary
   datasource.primary.username=devdb
   datasource.primary.password=******
   datasource.primary.jdbcUrl=jdbc:mysql://127.0.0.1:3306/dmsdb_primary?autoReconnect=true&characterEncoding=UTF-8

   #read datasource setting
   datasource.read.name=dmsdb_read
   datasource.read.username=devdb
   datasource.read.password=******
   datasource.read.jdbcUrl=jdbc:mysql://127.0.0.1:3306/dmsdb_read?autoReconnect=true&characterEncoding=UTF-8
   ```
   
4. 添加MybatisProperities的配置项：application.properties

   ```properties
   #mybatis setting
   mybatis.config-location=classpath:mybatis/mybatis-config.xml
   mybatis.mapper-locations=classpath:mybatis/mapper/**/*.xml
   mybatis.type-aliases-package=zoomus.addons.model
   mybatis.check-config-location=false
   mybatis.executor-type=reuse
   ```

5. 创建类: MybatisProperities.java

   ```java
   package zoomus.addons.config.properties;

   import org.apache.commons.lang3.builder.ToStringBuilder;
   import org.springframework.boot.context.properties.ConfigurationProperties;
   import org.apache.ibatis.session.ExecutorType;
   import org.springframework.core.io.Resource;
   /**
    * Created by lisanlai on 2016/12/13.
    */
   @ConfigurationProperties(prefix = MybatisProperties.MYBATIS_PREFIX)
   public class MybatisProperties {
       public static final String MYBATIS_PREFIX = "mybatis";

       private Resource configLocation;
       private Resource[] mapperLocations;
       private String typeAliasesPackage;
       private String typeHandlersPackage;
       private boolean checkConfigLocation = false;
       private ExecutorType executorType = ExecutorType.SIMPLE;

       public Resource getConfigLocation() {
           return configLocation;
       }

       public void setConfigLocation(Resource configLocation) {
           this.configLocation = configLocation;
       }

       public Resource[] getMapperLocations() {
           return this.mapperLocations;
       }

       public void setMapperLocations(Resource[] mapperLocations) {
           this.mapperLocations = mapperLocations;
       }

       public String getTypeHandlersPackage() {
           return this.typeHandlersPackage;
       }

       public void setTypeHandlersPackage(String typeHandlersPackage) {
           this.typeHandlersPackage = typeHandlersPackage;
       }

       public String getTypeAliasesPackage() {
           return this.typeAliasesPackage;
       }

       public void setTypeAliasesPackage(String typeAliasesPackage) {
           this.typeAliasesPackage = typeAliasesPackage;
       }

       public boolean isCheckConfigLocation() {
           return this.checkConfigLocation;
       }

       public void setCheckConfigLocation(boolean checkConfigLocation) {
           this.checkConfigLocation = checkConfigLocation;
       }

       public ExecutorType getExecutorType() {
           return this.executorType;
       }

       public void setExecutorType(ExecutorType executorType) {
           this.executorType = executorType;
       }

       @Override
       public String toString() {
           return new ToStringBuilder(this)
                   .append("configLocation", configLocation)
                   .append("mapperLocations", mapperLocations)
                   .append("typeAliasesPackage", typeAliasesPackage)
                   .append("typeHandlersPackage", typeHandlersPackage)
                   .append("checkConfigLocation", checkConfigLocation)
                   .append("executorType", executorType)
                   .toString();
       }
   }
   ```

6. 创建类： AbstractDatasourceConfig.java

   ```java
   package zoomus.addons.config;

   import org.apache.ibatis.session.SqlSessionFactory;
   import org.mybatis.spring.SqlSessionFactoryBean;
   import org.slf4j.Logger;
   import org.slf4j.LoggerFactory;
   import zoomus.addons.config.properties.MybatisProperties;

   import javax.sql.DataSource;

   /**
    * Created by lisanlai on 2016/12/13.
    */
   public abstract class AbstractDatasourceConfig {

       private static final Logger logger = LoggerFactory.getLogger(AbstractDatasourceConfig.class);

       public SqlSessionFactory sqlSessionFactory(DataSource primaryDataSource, MybatisProperties properties) throws Exception {
           logger.info("Initializing sqlSessionFactory; properties = {}", properties);
           final SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
           sessionFactory.setDataSource(primaryDataSource);
           sessionFactory.setConfigLocation(properties.getConfigLocation());
           sessionFactory.setMapperLocations(properties.getMapperLocations());
           sessionFactory.setTypeAliasesPackage(properties.getTypeAliasesPackage());
           sessionFactory.setTypeHandlersPackage(properties.getTypeHandlersPackage());
           return sessionFactory.getObject();
       }
   }
   ```

7. 创建类： PrimaryDatasourceConfig.java

   ```java
   package zoomus.addons.config;

   import org.apache.ibatis.session.SqlSessionFactory;
   import org.mybatis.spring.annotation.MapperScan;
   import org.slf4j.Logger;
   import org.slf4j.LoggerFactory;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.beans.factory.annotation.Qualifier;
   import org.springframework.boot.autoconfigure.jdbc.DataSourceBuilder;
   import org.springframework.boot.context.properties.ConfigurationProperties;
   import org.springframework.boot.context.properties.EnableConfigurationProperties;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.context.annotation.Primary;
   import org.springframework.jdbc.datasource.DataSourceTransactionManager;
   import org.springframework.transaction.annotation.EnableTransactionManagement;
   import zoomus.addons.config.properties.MybatisProperties;

   import javax.sql.DataSource;

   /**
    * Created by lisanlai on 2016/12/13.
    */
   @Configuration
   @EnableTransactionManagement
   @EnableConfigurationProperties(MybatisProperties.class)
   @MapperScan(basePackages = PrimaryDatasourceConfig.PACKAGE, sqlSessionFactoryRef = "primarySqlSessionFactory")
   public class PrimaryDatasourceConfig extends AbstractDatasourceConfig{

     	/** 指定java mapper 接口的包路径 */
       static final String PACKAGE = "zoomus.addons.repository.mybatis.primary";
       private static Logger logger = LoggerFactory.getLogger(PrimaryDatasourceConfig.class);

       @Autowired
       private MybatisProperties properties;

       @Bean
       @Primary
       @ConfigurationProperties(prefix="datasource.primary")
       public DataSource primaryDataSource() {
           logger.info("Initializing primaryDataSource...");
           return DataSourceBuilder.create().build();
       }

       @Bean(name = "primaryTransactionManager")
       @Primary
       public DataSourceTransactionManager primaryTransactionManager() {
           logger.info("Initializing primaryTransactionManager...");
           return new DataSourceTransactionManager(primaryDataSource());
       }

       @Bean(name = "primarySqlSessionFactory")
       @Primary
       public SqlSessionFactory primarySqlSessionFactory() throws Exception {
           logger.info("Initializing primarySqlSessionFactory...");
           return sqlSessionFactory(primaryDataSource(), properties);
       }
   }
   ```

8. 创建类： ReadDatasourceConfig.java

   ```java
   package zoomus.addons.config;

   import org.apache.ibatis.session.SqlSessionFactory;
   import org.mybatis.spring.annotation.MapperScan;
   import org.slf4j.Logger;
   import org.slf4j.LoggerFactory;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.autoconfigure.jdbc.DataSourceBuilder;
   import org.springframework.boot.context.properties.ConfigurationProperties;
   import org.springframework.boot.context.properties.EnableConfigurationProperties;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.jdbc.datasource.DataSourceTransactionManager;
   import org.springframework.transaction.annotation.EnableTransactionManagement;
   import zoomus.addons.config.properties.MybatisProperties;

   import javax.sql.DataSource;

   /**
    * Created by lisanlai on 2016/12/13.
    */
   @Configuration
   @EnableTransactionManagement
   @EnableConfigurationProperties(MybatisProperties.class)
   @MapperScan(basePackages = ReadDatasourceConfig.PACKAGE, sqlSessionFactoryRef = "readSqlSessionFactory")
   public class ReadDatasourceConfig extends AbstractDatasourceConfig{

       /** 指定java mapper 接口的包路径 */
       static final String PACKAGE = "zoomus.addons.repository.mybatis.read";
       private static Logger logger = LoggerFactory.getLogger(ReadDatasourceConfig.class);

       @Autowired
       private MybatisProperties properties;

       @Bean
       @ConfigurationProperties(prefix="datasource.read")
       public DataSource readDataSource() {
           logger.info("Initializing readDataSource...");
           return DataSourceBuilder.create().build();
       }

       @Bean(name = "readTransactionManager")
       public DataSourceTransactionManager readTransactionManager() {
           logger.info("Initializing readTransactionManager...");
           return new DataSourceTransactionManager(readDataSource());
       }

       @Bean(name = "readSqlSessionFactory")
       public SqlSessionFactory readSqlSessionFactory() throws Exception {
           logger.info("Initializing readSqlSessionFactory...");
           return sqlSessionFactory(readDataSource(), properties);
       }
   }
   ```

9. 添加Mapper类到相对应的package路径

   > - Primary mapper package
   >
   >   static final String PACKAGE = "zoomus.addons.repository.mybatis.primary";
   >
   > - Read mapper package
   >
   >   static final String PACKAGE = "zoomus.addons.repository.mybatis.read";

10. 使用演示

   ```java
   @Service
   public class DemoServiceImpl implements DemoService {
   	....
   	private final DemoPrimaryMapper demoPrimaryMapper;
   	private final DemoReadMapper demoReadMapper;

   	@Autowired
       public DemoServiceImpl(DemoPrimaryMapper demoPrimaryMapper,
                              demoReadMapper demoReadMapper) {
           Assert.notNull(demoPrimaryMapper, "demoPrimaryMapper must not be null!");
           Assert.notNull(demoReadMapper, "demoReadMapper must not be null!");
           this.demoPrimaryMapper = demoPrimaryMapper;
           this.demoReadMapper = demoReadMapper;
       }

   	@Override
       public void saveDemoObject(DemoObject demoObject) {
       	demoPrimaryMapper.save(demoObject);
       }

   	@Override
       public DemoObject getDemoObject(String id) {
       	return demoReadMapper.get(id);
       }

       //如果需要使用注解事务，需要指定事务管理器，如：
       @Transactional("primaryTransactionManager")

       ....

   }
   ```

   ​
