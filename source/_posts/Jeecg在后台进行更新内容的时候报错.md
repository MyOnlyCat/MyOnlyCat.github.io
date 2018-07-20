
---
title: "Jeecg在后台进行更新内容的时候报错"
date: 2018-6-4 15:33:27
categories: Jeecg
tags: 
		- Jeecg

---

**部署到Linux服务器在后台进行更新案例内容的时候出现下面这个错误**
<!-- more -->

## 0.环境

- 本机是win10,问题出现在部署到Linux服务器在后台进行更新案例内容的时候出现下面这个错误:

![QQ截图20180604153240.png-7.9kB][1]

## 1.解决

- 百度了一下这个问题说的好像是在`3.6`的jeecg版本中出现过,个人感觉是windows和Linux时间格式化的问题,具体的解决方式在下面的目录中:

![QQ截图20180604153922.png-13.6kB][2]

- 在这个两个bean中

```xml
<bean id="freemarker" class="freemarker.template.Configuration">
<bean id="freemarkerWord" class="freemarker.template.Configuration">
```

继续添加

```xml
<property name="numberFormat" value="0"></property>
<property name="dateFormat" value="yyyy-MM-dd"></property>
<property name="timeFormat" value="HH:mm:ss"></property>
<property name="dateTimeFormat" value="yyyy-MM-dd HH:mm:ss"></property>
```

- 举一个添加完成的例子

```xml
<bean id="freemarkerWord" class="freemarker.template.Configuration">
		<property name="templateLoader" ref="templetLoaderWord" />
		<property name="defaultEncoding" value="UTF-8"></property>
		<property name="numberFormat" value="0"></property>
		<property name="dateFormat" value="yyyy-MM-dd"></property>
		<property name="timeFormat" value="HH:mm:ss"></property>
		<property name="dateTimeFormat" value="yyyy-MM-dd HH:mm:ss"></property>
</bean>
```

## 2.疑问

- 貌似这样操作过后,以后比如我新增一个案例,或者新闻的时候,creatTime这个字段会录不进去

  [1]: http://static.zybuluo.com/pockadmin/b86mtv7r8mwxh65cb82w595f/QQ%E6%88%AA%E5%9B%BE20180604153240.png
  [2]: http://static.zybuluo.com/pockadmin/i6tvqnflv7hygyqpr8qtde75/QQ%E6%88%AA%E5%9B%BE20180604153922.png