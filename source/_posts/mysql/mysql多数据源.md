---
title: springboot + mybatisplus 配置多数据源
date: 2021-05-19 20:27:30
tags: mysql
---

## springboot 使用mybatisplus 配置多数据源
> springboot项目某个模块可能使用多个数据源, 那么就需要在配置文件中生成多个不同名称的数据源


### 1. yml配置文件
```xml
    spring:
      datasource:
        master:
          driver-class-name: com.mysql.cj.jdbc.Driver
          url: jdbc:mysql://ip:3306/数据库?serverTimezone=GMT%2B8&characterEncoding=utf-8&autoReconnect=true&allowMultiQueries=true
          username: 
          password: 
          type: com.alibaba.druid.pool.DruidDataSource
          druid:
            initial-size: 1
            min-idle: 1
            max-active: 5
            max-wait: 10000
            pool-prepared-statements: true
            max-open-prepared-statements: 60
            validation-query: select 'x' from dual
            test-while-idle: true
            time-between-eviction-runs-millis: 60000
            min-evictable-idle-time-millis: 300000
        slave:
          driver-class-name: com.mysql.cj.jdbc.Driver
          url: jdbc:mysql://ip:3306/数据库?serverTimezone=GMT%2B8&characterEncoding=utf-8&autoReconnect=true&allowMultiQueries=true
          username: 
          password: 
          type: com.alibaba.druid.pool.DruidDataSource
          druid:
            initial-size: 1
            min-idle: 1
            max-active: 5
            max-wait: 10000
            pool-prepared-statements: true
            max-open-prepared-statements: 60
            validation-query: select 'x' from dual
            test-while-idle: true
            time-between-eviction-runs-millis: 60000
            min-evictable-idle-time-millis: 300000
          

```
### 2 数据源的配置文件 (有几个数据源就写几个配置文件 我这边是两个)
```xml

    import com.alibaba.druid.pool.DruidDataSource;
    import com.baomidou.mybatisplus.extension.spring.MybatisSqlSessionFactoryBean;
    import org.apache.ibatis.session.SqlSessionFactory;
    import org.mybatis.spring.SqlSessionFactoryBean;
    import org.mybatis.spring.annotation.MapperScan;
    import org.springframework.beans.factory.annotation.Qualifier;
    import org.springframework.beans.factory.annotation.Value;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
    import org.springframework.jdbc.datasource.DataSourceTransactionManager;
    
    
    @Configuration
    @MapperScan(basePackages = MasterDataSourceConfig.PACKAGE, sqlSessionFactoryRef = "masterSqlSessionFactory")
    public class MasterDataSourceConfig {
        /**
         * 配置多数据源 关键就在这里 这里配置了不同的数据源扫描不同mapper
         */
        static final String PACKAGE = "第一个数据源的mapper所在的目录";

        /**
        *
        */ 第一个数据源所在的xml 配置文件        
        static final String MAPPER_LOCATION = "classpath*:com/**/*.xml";
    
        /**
         * 连接数据库信息 这个其实更好的是用配置中心完成
         */
        @Value("${spring.datasource.master.url}")
        private String url;
    
        @Value("${spring.datasource.master.username}")
        private String username;
    
        @Value("${spring.datasource.master.password}")
        private String password;
    
        @Value("${spring.datasource.master.driver-class-name}")
        private String driverClassName;
    
    
        @Bean("masterDataSource")
        @Primary
        public DataSource masterDataSource(){
            DruidDataSource dataSource = new DruidDataSource();
            dataSource.setUrl(url);
            dataSource.setUsername(username);
            dataSource.setPassword(password);
            dataSource.setDriverClassName(driverClassName);
            return dataSource;
        }
    
        @Bean(name = "masterTransactionManager")
        @Primary
        public DataSourceTransactionManager masterTransactionManager() {
            return new DataSourceTransactionManager(masterDataSource());
        }
    
        @Bean(name = "masterSqlSessionFactory")
        @Primary
        public SqlSessionFactory masterSqlSessionFactory(@Qualifier("masterDataSource") DataSource masterDataSource)
                throws Exception {
            final SqlSessionFactory sessionFactory = new SqlSessionFactory();  // mybatis 使用这个去生成sql工厂
            final MybatisSqlSessionFactoryBean sessionFactory = new MybatisSqlSessionFactoryBean();  // mybatisplus 是用这个生成sql工厂 不然会报找不到对应的方法
            sessionFactory.setDataSource(masterDataSource);
            sessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(MasterDataSourceConfig.MAPPER_LOCATION));
            return sessionFactory.getObject();
        }
    }
    
    
    
    import com.alibaba.druid.pool.DruidDataSource;
    import com.baomidou.mybatisplus.extension.spring.MybatisSqlSessionFactoryBean;
    import org.apache.ibatis.session.SqlSessionFactory;
    import org.mybatis.spring.SqlSessionFactoryBean;
    import org.mybatis.spring.annotation.MapperScan;
    import org.springframework.beans.factory.annotation.Qualifier;
    import org.springframework.beans.factory.annotation.Value;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
    import org.springframework.jdbc.datasource.DataSourceTransactionManager;
    
    import javax.sql.DataSource;
    
    @Configuration
    @MapperScan(basePackages = SlaveDataSourceConfig.PACKAGE, sqlSessionFactoryRef = "slaveSqlSessionFactory")
    public class SlaveDataSourceConfig {
           /**
                 * 配置多数据源 关键就在这里 这里配置了不同的数据源扫描不同mapper
                 */
           static final String PACKAGE = "第一个数据源的mapper所在的目录";        
           static final String MAPPER_LOCATION = "classpath*:com/**/*.xml";
    
         * 连接数据库信息 这个其实更好的是用配置中心完成
         */
        @Value("${spring.datasource.slave.url}")
        private String url;
    
        @Value("${spring.datasource.slave.username}")
        private String username;
    
        @Value("${spring.datasource.slave.password}")
        private String password;
    
        @Value("${spring.datasource.slave.driver-class-name}")
        private String driverClassName;
    
    
        @Bean("slaveDataSource")
        public DataSource slaveDataSource(){
            DruidDataSource dataSource = new DruidDataSource();
            dataSource.setUrl(url);
            dataSource.setUsername(username);
            dataSource.setPassword(password);
            dataSource.setDriverClassName(driverClassName);
            return dataSource;
        }
    
        @Bean(name = "slaveTransactionManager")
        public DataSourceTransactionManager slaveTransactionManager() {
            return new DataSourceTransactionManager(slaveDataSource());
        }
    
        @Bean(name = "slaveSqlSessionFactory")
        public SqlSessionFactory slaveTransactionManager(@Qualifier("slaveDataSource") DataSource slaveDataSource)
                throws Exception {
            final SqlSessionFactory sessionFactory = new SqlSessionFactory();  // mybatis 使用这个去生成sql工厂
            final MybatisSqlSessionFactoryBean sessionFactory = new MybatisSqlSessionFactoryBean();  // mybatisplus 是用这个生成sql工厂 不然会报找不到对应的方法
            sessionFactory.setDataSource(slaveDataSource);
            sessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(SlaveDataSourceConfig.MAPPER_LOCATION));
            return sessionFactory.getObject();
        }
    }

```

### 3 包的路径
```xml

   mapper 和 xml 的包的路径分别配置
  
   
```
![](/../../static/mysql/mysql多数据源.png)

### 4 总结
多数据源配置与mybatis结合使用，最主要的问题是区分是mybatis还是mybatisplus。使用的sqlSessionFactory 不同，其他的其实没啥区别


