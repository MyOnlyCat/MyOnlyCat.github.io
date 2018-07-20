
---
title: "SpringBoot启动报错The import org.springframework.data.repository.query.Param cannot be resolved"
date: 2017-12-19 10:31:31
categories: SpringBoot
tags: 
	- SpringBoot, 
	- 报错
---

**SpringBoot启动报错The import org.springframework.data.repository.query.Param cannot be resolved**
<!-- more -->

- 描述:在sprigBoot启动时遇到:
`The import org.springframework.data.repository.query.Param cannot be resolved`

- 解决方式
 `mvn clean dependency:tree` 命令看看依赖是否有问题
在运行`mvn clean compile`看是否失败,成功就没啥问题了

- 到这里重新启动就解决了问题




