
---
title: "Redis的更新策略"
date: 2017-12-20 10:23:12
categories: Redis
tags: 
	- SpringBoot, 
	- Redis
---


**SpringBoot + Redis的缓存更新策略**
<!-- more -->

### 前言
- 这里才用的模式为Cache Aside,这是标准的design pattern.它的具体逻辑如下:
- 失效：应用程序先从cache取数据，没有得到，则从数据库中取数据，成功后，放到缓存中。
- 命中：应用程序从cache中取数据，取到后返回。
- 更新：先把数据存到数据库中，成功后，再让缓存失效。
![1.png-84.8kB][1]
- 详细资料:[缓存更新的套路](https://coolshell.cn/articles/17416.html)

### 项目具体配置
- `pom.xml`依赖配置
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>demo</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.9.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>

		<!-- 核心模块，包括自动配置支持、日志和YAML -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>

		<!-- 测试模块，包括JUnit、Hamcrest、Mockito -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>

		<!-- 引入web依赖 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<!-- 引入JDBC -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-jdbc</artifactId>
		</dependency>
		
		<!-- 引入MYSQL驱动 -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>
		
		<!-- JPA -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>

		<!--redis配置依赖关系-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>


	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>


</project>

```
### 配置数据源
```yml
#mysql
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test
    driver-class-name: com.mysql.jdbc.Driver
    username: root
    password: root
```

### 配置JPA
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

### 配置Redis
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

### 建立实体类
- 下面的开发全部基于SpringBoot + JPA + Redis
- 实体类如下(**必须实现序列化**):
```java
package com.example.demo.model;

import javax.persistence.*;
import java.io.Serializable;

@Entity
@Table(name="user")
public class SystemUser implements Serializable{
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY) //id自增
	@Column(name = "id")
	private Integer id;
	
	@Column(name="user_name")
	private String name;
	
	@Column(name="user_password")
	private String password;

	public SystemUser(String name, String password) {
		this.name = name;
		this.password = password;
	}

	@Override
	public String toString() {
		return "SystemUser{" +
				"id=" + id +
				", name='" + name + '\'' +
				", password='" + password + '\'' +
				'}';
	}

	public SystemUser() {
	}

	public Integer getId() {
		return id;
	}

	public void setId(Integer id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getPassword() {
		return password;
	}

	public void setPassword(String password) {
		this.password = password;
	}
	
	
}

```

### 缓存更新策略的具体实现
```java
package com.example.demo.controller;

import com.example.demo.model.SystemUser;
import com.example.demo.service.IUserService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.ValueOperations;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.concurrent.TimeUnit;

/**
 * 创建时间:2017/12/20 0020
 * 创建人:lq
 */
@RestController
@RequestMapping(value = "redis")
public class RedisController {

    private Logger log = LoggerFactory.getLogger("RedisDemo");

    @Autowired
    private IUserService userService;

    @Autowired
    private RedisTemplate redisTemplate;



    /**
     * 获取User逻辑
     * 如果缓存中存在就从缓存区取出
     * 不存在就从DB中取出,然后插入缓存
     * @param id
     * @return
     */
    @RequestMapping(value = "find")
    public SystemUser findOne(Integer id){
        String key = "User_id" + id;
        //如果缓存中存在就取出
        if (redisTemplate.hasKey(key)){
            SystemUser user = (SystemUser) redisTemplate.opsForValue().get(key);
            log.info("RedisController.findOne():从缓存中取出SystemUser >>>" + user.toString());
            return user;
        }
        //不存在就在DB中取出
        SystemUser user = userService.findOne(id);

        //插入缓存
        redisTemplate.opsForValue().set(key, user, 60, TimeUnit.SECONDS);
        log.info("RedisController.findOne():插入User缓存 >>>" + user.toString());
        return user;
    }

    /**
     * 更新User逻辑
     * 先更新
     * 如果缓存存在删除,不存在不操作
     */
    @RequestMapping(value = "up")
    public SystemUser update(){
        SystemUser user = new SystemUser();
        user.setId(2);
        user.setPassword("123");
        user.setName("p");

        userService.save(user);

        String key = "User_id" + user.getId();

        if (redisTemplate.hasKey(key)){
            log.info("RedisController.update():从缓存中移除SystemUser >>>" + user.toString());
            redisTemplate.delete(key);
        }
        return user;
    }
}

```
- 第一次访问`findOne()`:
![2.png-10.5kB][2]
![3.png-12.6kB][3]

- 访问修改方法`update()`:
![4.png-12.8kB][4]

- 再次访问查找`findOne()`:
![5.png-7.8kB][5]

### 总结
- 采用Cache Aside的话,并发操作的概率可能非常低，因为这个条件需要发生在读缓存时缓存失效，而且并发着有一个写操作。而实际上数据库的写操作会比读操作慢得多，而且还要锁表，而读操作必需在写操作前进入数据库操作，而又要晚于写操作更新缓存，所有的这些条件都具备的概率基本并不大。

---


  [1]: http://static.zybuluo.com/pockadmin/7tjsr1qnzewgov2vhoi701k2/1.png
  [2]: http://static.zybuluo.com/pockadmin/wgkdqh9wym4djn7c1uo2gaus/2.png
  [3]: http://static.zybuluo.com/pockadmin/uxzcweydcjli71345evqcpdu/3.png
  [4]: http://static.zybuluo.com/pockadmin/en74wy4dojp8d0pn5dll7fro/4.png
  [5]: http://static.zybuluo.com/pockadmin/bcafv1o0xal2ncarltc2s0ng/5.png