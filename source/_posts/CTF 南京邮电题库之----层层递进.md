---
layout: post
title: CTF 南京邮电题库之----层层递进
categories: CTF
tags:
  - CTF
abbrlink: f2aca5e2
date: 2018-07-30 19:07:22
---


**从现在开始每日一篇博客来记录CTF,今日题目----层层递进**
<!-- more -->


## 0X00 题目概要

> ![image_1cjle1ti3s5bub01j5codq4hr9.png-26.1kB][1]

## 0X01 题目网站内容
![image_1cjle4dcm1nve17u715carte2mm.png-545kB][2]

## 0X02 解题过程

### 0X00 找!

- 浏览了一遍F12,没法现什么有用的东西,我一度以为地址变了,怀疑的一B

### 0X01 Network发现奇怪的东西?

- 源码不管用了,我就看看网站加载了什么东西,在Network发现加载了一个`404.html`,内容也有点奇怪:
![image_1cjlecqcncve10hsr0he921tim3j.png-86.2kB][3]

- 加载这么多的jQuery干嘛?????,定眼一看,jQuery的名字好奇怪哦什么-n -c 的哦,??????,在一看这不是flage吗?**每个jQuery后面的-后面的字母拼起来就是答案**
>![image_1cjlepcpl7oob3n1bdf1jeq11f240.png-12.5kB][4]

## 0X03 总结

- 感觉没有层层递进啊..........


  [1]: http://static.zybuluo.com/pockadmin/6ynreb8cw5v72nszgmch0b0w/image_1cjle1ti3s5bub01j5codq4hr9.png
  [2]: http://static.zybuluo.com/pockadmin/kartkd275ybx1g1zrafpu6wp/image_1cjle4dcm1nve17u715carte2mm.png
  [3]: http://static.zybuluo.com/pockadmin/u9xxqiwzmc8sj0j79ue7inwx/image_1cjlecqcncve10hsr0he921tim3j.png
  [4]: http://static.zybuluo.com/pockadmin/32dsc5dlspvb99miw5apmiqq/image_1cjlepcpl7oob3n1bdf1jeq11f240.png