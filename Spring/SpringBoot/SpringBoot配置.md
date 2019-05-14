---
title: SpringBoot配置
categories: SpringBoot
tags: [SpringBoot]
---

## Spring方式

@Configuration 声明一个配置类相当于xml

@Bean 将方法的返回值加入Bean容器，代理bean标签

@Value属性注入

@PropertySource指定外部属性文件

```
@Configuration
@PropertySource("classpath:jdbc.properties")
public class DruidConfig {

    @Value("${jdbc.url}")
    String url;
    @Value("${jdbc.driver-class-name}")
    String driverClassName;
    @Value("${jdbc.username}")
    String username;
    @Value("${jdbc.password}")
    String password;

    @Bean
    public DataSource dataSource(){
        DruidDataSource dataSource=new DruidDataSource();
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        dataSource.setUrl(url);
        dataSource.setDriverClassName(driverClassName);
        return dataSource;
    }

}
```

<!--more-->

## SpringBoot 方式

适合多个使用

```
@ConfigurationProperties(prefix = "jdbc")
@Data
public class ConfigProperties {
    String url;
    String driverclassname;
    String username;
    String password;
}
```

```
@Configuration
@EnableConfigurationProperties(ConfigProperties.class)
public class DruidConfig {

    @Bean
    public DataSource dataSource(ConfigProperties configProperties){
        DruidDataSource dataSource=new DruidDataSource();
        dataSource.setUsername(configProperties.getUsername());
        dataSource.setPassword(configProperties.getPassword());
        dataSource.setUrl(configProperties.getUrl());
        dataSource.setDriverClassName(configProperties.getDriverclassname());
        return dataSource;
    }

}
```

适合单个使用

```
@Configuration
public class DruidConfig {

    @Bean
    @ConfigurationProperties(prefix = "jdbc")
    public DataSource dataSource(){
        return new DruidDataSource();
    }
}
```

