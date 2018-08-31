---
layout: post
title: Json Web Token 学习
categories: JWT
tags:
  - JWT
abbrlink: '54405304'
date: 2018-08-02 09:46:38
---

**Java重构项目中需要用到Token认证,在EGG框架中是阿里整合了的,在新项目中需要单独写,由于使用OAutch2.0的协议看起来有点麻烦......所以被我弃用选择使用Json Web Token来实现,下面是学习记录了,资料来至[会飞的污熊](http://ju.outofmemory.cn/entry/341269)**
<!-- more -->

## 0X00 RESTful API 认证的安全考虑

- 在目前的项目中只有登录的接口试用了`Token`,但是是采用的`OAutch2.0`实现的,虽然是采用较为简单的`password模式`但是感觉实现起来还是较为复杂,在普通项目中我个人还是不推荐使用,推荐使用`JTW`实现`Token认证`

- [理解OAuth 2.0](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)
- [JWT](https://jwt.io)

### 0X01 JWT 的构成

![image_1cm4dte4pi08m6nk511gsv17jv9.png-122.8kB][1]

- 如图所示,JWT解析完成过后分为3部分 **HEADER**,**PAYLOAD**,**SIGNATURE**

- **第一部分-HEADER**
 - 包括两部分信息,第一部分**alg**: 声明加密的算法,这里为HS256
 - 第二部分**typ**: 声明类型, 这里为JWT

- **第二部分-PAYLOAD**
 - **PAYLOAD**部分是用base64对称加密进行加密**PAYLOAD**里面的信息,这里的信息为`sub`,`name`,`iat`,由于属于对称加密所以**PAYLOAD**一般不建议存放敏感信息

- **第三部分-SIGNATURE**
 - **SIGNATURE**是一个签证信息，这个签证信息由三部分组成：
>   header (base64后的)
    payload (base64后的)
    secret

    这个部分需要base64加密后的header和base64加密后的payload使用.连接组成的字符串，然后通过header中声明的加密方式进行加盐secret组合加密，然后就构成了jwt的第三部分。
    
- **组成JWT**

```js
// javascript
var encodedString = base64UrlEncode(header) + '.' + base64UrlEncode(payload);
var signature = HMACSHA256(encodedString, 'secret'); // TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

将这三部分用.连接成一个完整的字符串,构成了最终的jwt
<br>
**注意：secret是保存在服务器端的，jwt的签发生成也是在服务器端的，secret就是用来进行jwt的签发和jwt的验证，所以，它就是你服务端的私钥，在任何场景都不应该流露出去。一旦客户端得知这个secret, 那就意味着客户端是可以自我签发jwt了。**
   
### 请求里面应用JWT

- 一般是在请求头里加入Authorization，并加上Bearer标注：

```js
fetch('api/user/1', {
  headers: {
    'Authorization': 'Bearer ' + token
  }
})
```

### 解析大致流程

![image_1cm4h5qg01l6s1vfrh6m19211cqbm.png-73.7kB][2]

### JWT安全相关

- JWT协议本身不具备安全传输功能，所以必须借助于SSL/TLS的安全通道，所以建议如下：

    1. 不应该在jwt的payload部分存放敏感信息，因为该部分是客户端可解密的部分。
    2. 保护好secret私钥，该私钥非常重要。
    3. 如果可以，请使用https协议


  [1]: http://static.zybuluo.com/pockadmin/8b6rl0lb7tg6m1hnbtqjqarm/image_1cm4dte4pi08m6nk511gsv17jv9.png
  [2]: http://static.zybuluo.com/pockadmin/2u04gk7ewih33004cnpvb6zt/image_1cm4h5qg01l6s1vfrh6m19211cqbm.png