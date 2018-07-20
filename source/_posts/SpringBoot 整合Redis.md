
---
title: "SpringBoot整合Redis"
date: 2017-12-19 17:48:12
categories: SpringBoot
tags: 
		- SpringBoot, 
		- Redis
---

**记录下第一整合过程**
<!-- more -->

[TOC]
### 1前言

- Redis服务器在虚拟机中搭建,系统为Ubuntu,Redis版本为4.0.6;文件配置均在`application.yml`中完成

### 2.springBoot添加Redis依赖
- 在`pom.xml`添加依赖:
```xml
<!--redis配置依赖关系-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

### 3.Redis单机配置
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

### 4.编写Redis测试用例

这个用例中完成了Redis对字符的存储,和对对象的存储
**注意:** `SystemUser`这个对象必须实现`Serializable`接口,否则会报错

- `RedisDemo.java`

```java
/**
 * 创建时间:2017/12/19 0019
 * 创建人:lq
 * Redis简单存取操作
 */
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = DemoApplication.class)
public class RedisDemo {
    @Autowired
    private RedisTemplate redisTemplate; 

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    private Logger log = LoggerFactory.getLogger("RedisDemo");

    @Test
    public void test(){
        stringRedisTemplate.opsForValue().set("name","pock");
        SystemUser user = new SystemUser("admin","123");
        redisTemplate.opsForValue().set("user",user);
        log.info("success");
    }

    @After
    public void testGet(){
        SystemUser user = (SystemUser) redisTemplate.opsForValue().get("user");
        log.info(user.getName());
    }

}
```
- `RedisTemplate`:会使用`JdkSerializationRedisSerializer`，这意味着`key`和`value`都会通过Java进行序列化。 
- `StringRedisTemplate`默认会使用`StringRedisSerializer`

### 结束
到这里就实现了一个最简单的springboot整合Redis的demo






