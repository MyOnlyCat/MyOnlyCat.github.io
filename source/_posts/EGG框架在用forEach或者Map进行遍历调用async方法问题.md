
---
title: "EGG框架在用forEach或者Map进行遍历调用async方法问题"
date: 2018-7-4 10:20:50
categories: EGG
tags: 
		- EGG
---

EGG框架在用forEach或者Map进行遍历调用async方法问题
<!-- more -->


## 0X00 出现的情况

```js
    //假设 newList为一个array类型,这段代码处于async中
    newList.forEach(x => x{
        //这里的的await 会报错,如果不用await这里返回的y就是一个promise对象对于我这种入门的人不太好处理
        var y = await this.model.show();
    })

```

## 0X01 我的解决方法

```js
    //假设 newList为一个array类型,这段代码处于async中
    for(let i = 0; i < newList.length; i++) {
        //这里再来调用async方法,这里await方法不会报错,返回的y也是对的
        var y = await this.model.show();
    }
```