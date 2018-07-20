
---
title: "Spring Cloud初体验"
date: 2017-12-25 15:08:21
categories: Spring Cloud
tags: 
		- Spring,
		- Cloud
---

**上周星期五的时候,完成了一次springcloud的小体验,现在把流程和想法记下来**
<!-- more -->

### 前言

- 按照网上的教程来写的demo
- 首先需要一个服务端,还需要一个服务提供者,而且服务提供者需要把自己注册到服务端,另外还需要一个服务消费者,服务提供者也可以是服务消费者
- 熔断器Hystrix的话,就相当于短路机制.熔断器在什么情况先需要呢?
我的理解是,比如在A调用B的服务表明上是直接调用的B,其实是A请求服务端,服务端在把B服务提供给A,如果发生A调用B,B在调用C,这个过程就是A请求服务端发送给了B,B在请求服务端,服务端在把C服务给B,B在给A;
这种情况下如果C服务出现问题,或者延迟的话,B就会一直等待C的响应,然后A就会一直等待B的响应,要不到多久这种连锁反应就会让集群崩溃
在这种情况下熔断器的作用就体现出来了
熔断器会在,请求服务,服务出现多次错误或者延迟的情况下把出现错误或者延迟的服务关闭掉,让请求这个服务的其它程序回调(`fallback`),这样就不会出现集群的连锁反应

### 建立服务端

- 在最基本的springboot项目的`pom.xml`文件里面加入如下依赖:

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-eureka-server</artifactId>
</dependency>

<dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-dependencies</artifactId>
			<version>Brixton.RELEASE</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>

```

- **注意添加依赖的方式**

![2.png-47.9kB][1]

- 修改配置文件

```yml
server:
  port: 1111

eureka:
  client:
    # 这里把服务端注册自己到服务里改为false
    register-with-eureka: false
    fetch-registry: false
    serviceUrl:
      defaultZone: //localhost:${server.port}/eureka/
```

- 修改启动类

```java
@EnableEurekaServer
@SpringBootApplication
public class SpringCloudDemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringCloudDemoApplication.class, args);
	}
}
```

加入 `@EnableEurekaServer`

- 启动效果如下(**我启动了两个服务提供方,设定了不同的端口,一个有say方法,一个没有**):

![1.png-79.3kB][2]


### 创建服务提供者

- 在基本的springboot项目上添加依赖

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>



<dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-dependencies</artifactId>
			<version>Brixton.RELEASE</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>

```

- 修改配置

```xml
spring.application.name=compute-service

server.port=2222

eureka.client.serviceUrl.defaultZone=http://localhost:1111/eureka/
```

- 编写服务controller

```java
/**
 * 创建时间:2017/12/22 0022
 * 创建人:lq
 */
@RestController
public class MyController {

    private final Logger logger = Logger.getLogger(getClass());

    @Autowired
    private DiscoveryClient client;

    @RequestMapping(value = "/add", method = RequestMethod.GET)
    public Integer add(@RequestParam Integer a, @RequestParam Integer b){

        ServiceInstance instance = client.getLocalServiceInstance();

        Integer c = a + b;

        logger.info("/add, host:" + instance.getHost() + ", service_id:" + instance.getServiceId() + ", result:" + c);

        return c;
    }
}

```

- 在启动器上加入: `@EnableDiscoveryClient`注解,注册到服务中心

- 启动过后如图表明以及注册成功:
![3.png-35.2kB][3]


### 编写消费服务方

- 这里使用的 Feign

- 在最基本的springboot项目上添加依赖(**注意添加依赖和上面的类型**):

```xml

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-feign</artifactId>
</dependency>

<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
		
		
<dependencyManagement>
<dependencies>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-dependencies</artifactId>
		<version>Brixton.SR5</version>
		<type>pom</type>
		<scope>import</scope>
	</dependency>
</dependencies>
</dependencyManagement>

```

- 修改配置

```yml

server:
  port: 3333

spring:
  application:
    # 显示在服务中心的名字,以及调用时的名字?
    name: compute-service
eureka:
  client:
    serviceUrl:
        #指定服务中心
      defaultZone: http://localhost:1111/eureka/

```

- 编写调用接口

```java

@FeignClient("compute-service") // 指定调用的服务名
public interface ComputeClient {
    //这里访问的路径,请求方式,参数都必须和服务提供端的一样
    @RequestMapping(value = "/add", method = RequestMethod.GET)
    Integer add(@RequestParam(value = "a") Integer a, @RequestParam(value = "b") Integer b);
    
    //测试方法
    @RequestMapping(value = "/say", method = RequestMethod.GET)
    String say();
}

```

- 编写Controller进行测试

```java

@RestController
public class My {
    
    //注入接口
    @Autowired
    ComputeClient computeClient;

    @RequestMapping(value = "/add", method = RequestMethod.GET)
    public Integer add() {
        return computeClient.add(10, 20);
    }

    @RequestMapping(value = "/say", method = RequestMethod.GET)
    public String say(){
        return computeClient.say();
    }
}

```

- 修改启动类

```java

@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class RibbonDemoApplication {


	public static void main(String[] args) {
		SpringApplication.run(RibbonDemoApplication.class, args);
	}
}

```

- 启动过后如图:

![4.png-63.8kB][4]

### 测试

- 这里我启动的两个服务提供方,端口指定不一样

- 访问服务消费者的add方法

![5.png-12.3kB][5]

- 第一次访问其中一个服务提供日志打印如下

![6.png-76.5kB][6]

- 再次访问add,另一个服务端打印结果

![7.png-25.3kB][7]


### 结论

通过`Feign`以接口和注解配置的方式，轻松实现了对compute-service服务的绑定，这样我们就可以在本地应用中像本地服务一下的调用它，并且做到了客户端均衡负载。

但是如果访问3333的say方法,出现第一次访问如果是成功,第二次访问肯定404,这是因为启动了两个服务端,一个有say方法一个没有say方法,在`Feign`实现的客户端均衡负载的时候,是采用的遍历的方式应该,类似于上面的`add`方法,第一次访问第一个服务端,第二次访问第三个服务端.


  [1]: http://static.zybuluo.com/pockadmin/pcsordtocbx8xvfzovexg7nt/2.png
  [2]: http://static.zybuluo.com/pockadmin/m4olwzpp5kniu2uljz22171f/1.png
  [3]: http://static.zybuluo.com/pockadmin/mbwkcgihkrsyy4fwwms0z9bq/3.png
  [4]: http://static.zybuluo.com/pockadmin/adymuks68c2x5129uv1wdw3r/4.png
  [5]: http://static.zybuluo.com/pockadmin/vnmw6vvnygsro67d3jqv3ix1/5.png
  [6]: http://static.zybuluo.com/pockadmin/4qnc50hgvaenh2scj9ah4rn7/6.png
  [7]: http://static.zybuluo.com/pockadmin/euk7pz9w5lft9rnbpsz7ir3p/7.png