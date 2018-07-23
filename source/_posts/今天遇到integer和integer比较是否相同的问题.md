---
title: integer和integer比较是否相同用==的问题
categories: java开发遇到的问题
tags:
  - 疑难杂症
abbrlink: 79bb4db8
date: 2017-12-25 14:06:56
---

**integer和integer比较是否相同的问题**
<!-- more -->

- 今天在继续开发商城项目时,遇到一个问题,我根据地址取出用户ID(地址唯一),代码如下:

```java
// 处理转账到自己的账户
Integer userIdByAddress = getUserIdByAddress(address);
if (userIdByAddress == Integer.valueOf(memberId)) {
	return JsonResultUtil.getErrorJson("无法转账给自己");
}
```

- 这里我测试的时候输入的自己的`address`也进入不了

- 在网上查了一下原因,大概是因为integer范围的问题

```java
static final Integer cache[] = new Integer[-(-128) + 127 + 1];

        static {
            for(int i = 0; i < cache.length; i++)
        cache[i] = new Integer(i - 128);
        }

```

- 总结:这是源码中的，也就是说cache[]中已有-128到127，不在这范围的会新new ，这时可以理解比较是内存地址，
也就是是不是同一对象.
所以说当Integer的值不在-128到127的时候使用==方法判断是否相等就会出错，在这个范围之内的就会没有问题！
以后最好用equals


