
---
title: "IDEA新建项目时，没有Spring Initializr选项"
date: 2017-12-22 16:17:16
categories: IDEA
tags: 
	- 疑难杂症
---

**IDEA新建项目时，没有Spring Initializr选项**
<!-- more -->

最近开始使用IDEA作为开发工具，然后也是打算开始学习使用spring cloud。
看着博客来进行操作上手spring cloud，很多都是说
创建一个新项目(Create New Project)

选择 Spring Initializr。然而我发现我的IDEA上面没有Spring Initializr这个选项。解决办法如下：

在settings -> Plugins 里面搜索spring boot，勾选上，然后再重启下idea，就可以了。如果Plugins里面没有spring boot的话，先安装下，再勾选




