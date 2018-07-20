
---
title: "Spring Cloud自己学习的时候遇到的一些问题"
date: 2017-12-22 18:30:42
categories: Spring Cloud
tags: 
	- 疑难杂症
---



**Spring Cloud自己学习的时候遇到的一些问题**
<!-- more -->

### Feign
- `pom.xml`引起问题的配置部分如下:

```xml
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

- 报错内容

```java
Attribute 'value' in annotation [org.springframework.cloud.netflix.feign.FeignClient] must be declared as an @AliasFor [serviceId], not [name].
```

- 解决方法

将Spring Cloud版本改为 `Brixton.SR5` 或 `Camden.RELEASE` ，即可解决此问题。




