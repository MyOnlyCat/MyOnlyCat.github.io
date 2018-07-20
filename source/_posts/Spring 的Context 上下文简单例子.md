
---
title: "Spring 的Context 上下文简单例子"
date: 2018-5-18 10:37:28
categories: ApplicationContext
tags: 
		- spring, 
		- Context上下文
---


**Spring 的Context 上下文简单例子(技术不够啊,现在还没理解)**
<!-- more -->

## 前言
**简单例子便于以后使用,这里Man注入是采用构造器注入,需要QQcar类,所以在XML配置的时候使用的`constructor-arg`,来指定构造器需要的东西**

## 1.基于xml的配置方式
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-3.2.xsd">
    <bean id="man" class="spring.chapter1.domain.Man">
        <constructor-arg ref="qqCar" />
    </bean>
    <bean  id="qqCar" class="spring.chapter1.domain.QQCar"/>
</beans>
```

**配置完成后**

```java
public class Test {
    public static void main(String[] args) {
        //加载项目中的spring配置文件到容器
//        ApplicationContext context = new ClassPathXmlApplicationContext("resouces/applicationContext.xml");
        //加载系统盘中的配置文件到容器(两种方式都能获取)
        ApplicationContext context = new FileSystemXmlApplicationContext("E:/Spring/applicationContext.xml");
        //从容器中获取对象实例
        Man man = context.getBean(Man.class);
        man.driveCar();
    }
}
```

## 基于注解式

```java
//同xml一样描述bean以及bean之间的依赖关系
@Configuration
public class ManConfig {
    @Bean
    public Man man() {
        return new Man(car());
    }
    @Bean
    public Car car() {
        return new QQCar();
    }
}
```

**配置完成**

```java
public class Test {
    public static void main(String[] args) {
        //从java注解的配置中加载配置到容器
        ApplicationContext context = new AnnotationConfigApplicationContext(ManConfig.class);
        //从容器中获取对象实例
        Man man = context.getBean(Man.class);
        man.driveCar();
    }
}
```





