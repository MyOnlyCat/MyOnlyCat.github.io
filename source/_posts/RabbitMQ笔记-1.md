---
layout: post
title: RabbitMQ笔记-1
categories: RabbitMQ
tags:
  - RabbitMQ
abbrlink: 20a535df
date: 2019-01-18 09:45:19
---

**最近遇到了任务的问题,想采用一种高效成熟的消息队列去处理,所以就开始看看RabbitMQ**
<!--more-->

[TOC]

## RabbitMQ的历史

* RbitMQ是一个Erlang开发的AMQP（Advanced Message Queuing Protocol ）的开源实现。AMQP 的出现其实也是应了广大人民群众的需求，虽然在同步消息通讯的世界里有很多公开标准（如 Cobar）的 IIOP ，或者是 SOAP 等），但是在异步消息处理中却不是这样，只有大企业有一些商业实现（如微软的 MSMQ ，IBM 的 WebSphere MQ 等），因此，在 2006 年的 6 月，Cisco 、Red Hat、iMatix 等联合制定了 AMQP 的公开标准。

* * *

* RabbitMQ由RabbitMQ Technologies Ltd开发并且提供商业支持的。该公司在2010年4月被SpringSource（VMware的一个部门）收购。在2013年5月被并入Pivotal。其实VMware，Pivotal和EMC本质上是一家的。不同的是，VMware是独立上市子公司，而Pivotal是整合了EMC的某些资源，现在并没有上市。


## RabbitMQ简述

* RabbitMQ是一个实现**AMQP**的消息中间件,其服务器端使用**Erlang**语言编写

* * *

* 需要发送邮件的人,先把邮件放到**邮箱**(这里邮箱就相当于MQ中的队列),后面邮箱中的**邮件**(相当于在队列中的信息)被送到指定人的手中,然后收件人需要**确认收到**

* * *

* 需要发送邮件的人,先把邮件放到**邮箱**(这里邮箱就相当于MQ中的队列),后面邮箱中的**邮件**(相当于在队列中的信息)被送到指定人的手中,然后收件人需要**确认收到**

