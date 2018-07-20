
---
title: "SpringBoot依赖整理"
date: 2017-12-19 14:09
categories: SpringBoot
tags: 
		- SpringBoot, 
		- 依赖
---

**整理下吧**
<!--more-->

### 前言
- 以下的配置全部在`application.yml`文件中配置
- 均为在学习过程中的总结
### 1.核心模块
- 核心模块，包括自动配置支持、日志和YAML
```xml
<!-- 核心模块，包括自动配置支持、日志和YAML -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter</artifactId>
</dependency>
```
### 2.测试模块
- 测试模块，包括JUnit、Hamcrest、Mockito
```xml
<!-- 测试模块，包括JUnit、Hamcrest、Mockito -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>
```
### 3.web依赖
- web依赖
```xml
<!-- 引入web依赖 -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
### 4.JDBC依赖
- JDBC依赖
```xml
<!-- 引入JDBC -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```
- 参数配置
```yml
#mysql
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test
    driver-class-name: com.mysql.jdbc.Driver
    username: root
    password: root
```
### 5.MYSQL驱动
- MYSQL驱动
```xml
<!-- 引入MYSQL驱动 -->
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
</dependency>
```
### 6.引入JPA
- 引入JPA
```xml
<!-- JPA -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```
- 参数配置
```yml
#jpa
  jpa:
    #指定JPA数据库
    database: mysql
    #showSql
    show-sql: true
   
    #hibernate.hbm2ddl.auto节点的值有几个create、create-drop、update、validate、none

    #create：每次加载hibernate会自动创建表，以后启动会覆盖之前的表，所以这个值基本不用，严重会导致的数据的丢失。

    #create-drop ： 每次加载hibernate时根据model类生成表，但是sessionFactory一关闭，表就自动删除，下一次启动会重新创建。

    #update：加载hibernate时根据实体类model创建数据库表，这是表名的依据是@Entity注解的值或者@Table注解的值，sessionFactory关闭表不会删除，且下一次启动会根据实体model更新结构或者有新的实体类会创建新的表。

    #validate：启动时验证表的结构，不会创建表

    #none：启动时不做任何操作
    hibernate:
      ddl-auto: none
      #命名策略
      naming:
        strategy: org.hibernate.cfg.ImprovedNamingStrategy
```
### 7.引入Redis
```xml
<!--redis配置依赖关系-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
- Redis单机参数配置
```yml
#redis
  redis:
    #Redis数据库索引（默认为0）
    database: 0
    # Redis服务器地址
    host: 192.168.3.58
    # Redis服务器连接端口
    port: 6379
    # Redis服务器连接密码(默认为空)
    password:
    pool:
      # 连接池最大连接数（使用负值表示没有限制）
      maxActice: 8
      # 连接池最大阻塞等待时间（使用负值表示没有限制）
      maxWait: -1
      # 连接池中的最大空闲连接
      maxIdle: 8
      # 连接池中的最小空闲连接
      minIdle: 0
      # 连接超时时间（毫秒）
    timeout: 0
```

### 引入mybatis

```xml
<dependency>
	<groupId>org.mybatis.spring.boot</groupId>
	<artifactId>mybatis-spring-boot-starter</artifactId>
	<version>1.2.0</version>
</dependency>
```

### 引入cassandra数据库?

```xml
<!--引入cassandra数据库?-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-cassandra</artifactId>
</dependency>
```

### springboot热部署

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
```



