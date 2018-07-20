
---
title: "SoringBoot正常启动,但是页面Whitelabel Error Page"
date: 2017-12-22 11:13:12
categories: SoringBoot
tags: 
	- SpringBoot, 
	- 启动报错
---

**SoringBoot正常启动,但是页面Whitelabel Error Page**
<!-- more -->

- **环境:** idea + springboot


- **项目结构如下:**
![1.png-10.7kB][1]


- **情况描述:** 
启动`DemoApplication`正常启动没报任何错误,访问`mybatisDemo`下的`controller`无法访问,但是访问`demo`下的`controller`可以访问.

- **问题原因:**
我觉得spring boot只会扫描启动类当前包和以下的包,如果将启动类放到demo下,就只会扫描demo下的

- **解决方法:**
将启动类放到mybatisDemo下


  [1]: http://static.zybuluo.com/pockadmin/gu1cdubl29bjzb8oppqktvcy/1.png