
---
layout: post
title: "EGG 入门笔记"
date: 2018-6-4 16:13:37
tags: 
			- EGG
---

EGG框架快速上手
<!-- more -->

## 0X00 

- 这里基于我的情况,需要快速上手


## 0X01 基于Egg框架的项目启动

- 找到目录中的`package.json`,点击右键`show npm scripts` ,然后`dev run`,`dev run`是属于在测试环境中的模式,如果在win环境下跑`run`的话会出现好多窗口,是因为有多少启动了多个线程,就出现了那么多的窗口

## 0X02 Egg基本目录结构

![QQ截图20180608093924.png-3.5kB][1]

- `controller`文件夹和java的一样

- `extent` 属于可选的,可以对框架进行扩展配置,详情见[框架扩展](http://eggjs.org/zh-cn/basics/extend.html)

- `model` 是属于Egg的自定义目录规范,详情[Loader API](https://eggjs.org/zh-cn/advanced/loader.html)

- `public` 属于可选,放置静态文件,详情[egg-static](https://github.com/eggjs/egg-static)

- `service` 和java的service层一样

- `router.js` 这个主要是路由的配置,相当于servlet配置映射的路径,**这个还是比较重要的**

## 0X03 基于RESTful风格的简单的CURD

### 0.Egg中规定的RESTful的请求

- **`ctx`的含义**

**ctx - 当前请求的 Context 实例。**

- Egg中规定的RESTful的请求

![1.png-21.6kB][2]


### 1.简述思路
 - 这里举一个新闻的例子,对新闻进行CURD操作,从`model`层开始编写,在编写`controller`层,最后在配置路由`router.js`配置出RESTful风格的接口(这里不用编写`service`层是因为我`controller`继承了一个公共类,其中有最基础的CURD方法,所以说这次上手还是感觉比较简单)

### 2.Model层的编写,实体的写法

- 下面是一个新闻的实体

```js
'use strict';
module.exports = app => {
    const mongoose = app.mongoose;
    const tempSchema = new mongoose.Schema({
        _id: mongoose.Schema.Types.ObjectId,
        tile: String, //标题
        content: String, //内容
        intro: String, //简介
        creation: Number, //创建时间
        updated: Number, //更新时间
    }, {
        versionKey: false,
    });

    return mongoose.model('news', tempSchema);
};
```
- `use strict` 表示使用严格模式
为什么用严格模式
> * 消除代码运行的一些不安全之处，保证代码运行的安全；
> * 提高编译器效率，增加运行速度；
> * 为未来新版本的Javascript做好铺垫。

- `app.mongoose` 调用mnogoDB插件

- `const tempSchema` 表示mnogoDB的数据库类型?(暂时不确定,没有使用过mnogoDB)

- 5到10行就是字段

- `versionKey: false,` 这个一个是个标准的写法吧

- ` return mongoose.model('news', tempSchema);` 这个可以对应到`service`层

### 3.编写Controller层

- 下面的新闻的controller

```js
'use strict';

const CommonController = require('./commonController');

class NewsController extends CommonController{
    init() {
        this.daoService = this.service.news;
    }

    async create (ctx) {
        await super.create(ctx);
        ctx.logger.debug("增加");
    }

    async show (ctx) {
        await super.show(ctx);
        ctx.logger.debug("根据ID展示");
    }

    async index (ctx) {
        await super.index(ctx);
        ctx.logger.debug("展示所有");
    }

    async update (ctx) {
        await super.update(ctx);
        ctx.logger.debug("更新");
    }

    async destroy () {
        await super.destroy(ctx);
        ctx.logger.debug("删除");
    }



}

module.exports = NewsController;
```

- 这里第一行也是使用严格模式,提高编译和运行速度

- `const CommonController = require('./commonController');` 这是相当于java的导包操作,把`commonController`引用到当前类中(es6有class的写法)

- 第五行和java的class写法一样`extends`也一样表示继承

- `init()` 表示初始化`service`层(感觉在egg框架还是es6的语法规则中,比如你要调用`service`层或者你要调用`model`层的实体的话,都是需要先进行`init()`进行初始化),虽然这里并没有用到新闻的`service`层

- CRUD(那增加进行举例)
```js
 async create (ctx) {
        await super.create(ctx);
        ctx.logger.debug("增加");
    }
```
`async` 表示使用异步的方式,`await`表示必须等到`suoer.create`的返回值在进行下一步
在下面就是日志记录

### 4.简单介绍一下**service**层的写法

- 虽然简单的CURD我是基于现成的类进行编写的(在框架整合了简单的curd),但是在真实的业务情况中这些简单的CURD肯定是无法满足业务需求的,所以这里需要新的`service`层
 下面是例子

```js
'use strict';
const DaoService = require('./daoService');

class NewsService extends DaoService{
    init() {
        this.model = this.ctx.model.News;
    }
}

module.exports = NewsService;
```

- 这里还是要先进行`init()`方法初始化

- 自定义的方法编写到`daoService` 中,这里进行引用应该就可以了


### 5.处理路由**router.js**

- **路由还是比较简单的,但是我觉得还是重要**
    下面是路由配置

```js
'use strict';
const api = '/api/v1';
/**
 * @param {Egg.Application} app - egg application
 */
module.exports = app => {
  const { router, controller } = app;
  router.post(api + '/surveyRecords/search', controller.surveyRecords.index);
  router.get(api + '/surveyRecords/exportRecord', controller.surveyRecords.exportRecord);
  router.resources('surveyRecords', api + '/surveyRecords', controller.surveyRecords);
  // 角色
  router.post(api + '/roles/search', controller.roles.index);
  router.resources('roles', api + '/roles', controller.roles);
  router.resources('news', api + '/news', controller.news);
};

```

- 这里一样是使用严格模式

- `const api = '/api/v1'` 规定请求路径

- `module.exports = app => {}` 标准写法

- `const { router, controller } = app;` 这个应该也是标准写法

- `router.post(api + '/surveyRecords/search', controller.surveyRecords.index);`这个是post请求到`controller`文件夹下的`surveyRecords.js`文件中的`index`方法

- `router.resources('news', api + '/news', controller.news);`这个是整合了post啊get啊那些的RESTful风格,可以根据你的请求进行自动访问规定的方法

## 0X04接口分析(居家养老服务管理系统)

### 0.登录接口

#### A. 路由

>   router.get(api + '/users/login', controller.users.login);

#### B. Controller层代码

```js
    async login(ctx) {
        //从ctx 获取当前GET请求的参数。
        const user = await this.service.users.login(ctx.query.phone, ctx.query.password);
        if (user && user.roles) {
            user.permissions = {};
            const allPromise = Object.keys(user.roles).map(async s => {
                if (Array.isArray(user.roles[s])) {
                    const roles = await this.getRolesBySystem(user.roles, s);
                    user.permissions[s] = roles;
                } else {
                    for (const orgId of Object.keys(user.roles[s])) {
                        const roles = await this.getRolesBySystemOrg(user.roles, s, orgId);
                        user.permissions[s] = {[orgId]: roles};
                    }
                }
            });
            await Promise.all(allPromise);
        }
        ctx.body = user;
    }
```

- `if (user && user.roles)`
user不为空,user.roles不为空

- ` Object.keys()`
返回一个数组

- `if (Array.isArray(user.roles[s]))`
判断是不是数组

- getRolesBySystem()

#### C. Service层代码中的

#####  **login()方法**
```js
  async login(phone, password) {
    const md5Pass = this.ctx.helper.md5(password);
    const queries = {
      $or: [{phone}, {shortName: phone}], password: md5Pass,
    };

    return await this.model.findOne(queries).lean();
  }
```

- 1.通过controller层的` const user = await this.service.users.login(ctx.query.phone, ctx.query.password);`调用到service层来

- 2.`const md5Pass = this.ctx.helper.md5(password);`
调用help层(相当于java的util)进行MD5处理

- 3.**`$or`**是个重点,这里举一个官方例子:

```js
db.inventory.find( { $or: [ { quantity: { $lt: 20 } }, { price: 10 } ] } )
```
意思是取出inventory(存货) 集合,条件在`$or`中分别有两个条件,第一个是quantity字段的值小于20或者price字段等于10,`$lt`的意思就是小于某个值(以后会有更详细的文章来介绍MongoDB的语法)

- 4.` $or: [{phone}, {shortName: phone}], password: md5Pass`
先来大概分析这句的意思,肯定有两个条件`$or`中是一个条件,逗号后面是一个条件。
在来分析`$or`中的条件,`$or`肯定是有两个条件,系统可以用两种账号来进行登录,第一个种是手机也就是**phone**字段,第二种是短命登录也就是**shortName**字段,`{phone}`相当于**`{phone:phone}`**,我猜测用`{phone}`简写是因为在数据库的字段和这里的字段是一样的。
第一个条件就分析完了，在来分析第二个条件也就是`password: md5Pass`这个没有`$or`说明这个值是一个必须满足的条件。
最后在梳理一下，假如传入参数为**phone=110,password=123**，那么这条语句的意思是**在数据库中查找phone字段等于110，或者shortName字段等于110，同时满足password字段=123**

- 5.`return await this.model.findOne(queries).lean();`
**model**层的实体都有一些默认的方法，**findOne()**就是其中一个，这里传入的参数就是上面拼接的MongoDB数据库语句

- 6.`lean()`
这里单独说一下`lean()`，不加这个方法的话他返回的对象中就带有model的一些默认方法,所以要加这个方法

##### **getRolesBySystem()方法**

```js
    async getRolesBySystem(userRoles, system) {
        const searchParams = {system, name: {$in: userRoles[system]}};
        const roles = await this.service.roles.find(searchParams);
        const results = roles.map(a => ({[a.name]: a.permission})).reduce((a, b) => ({...a, ...b}), {});
        return results;
    }
```

- 接收到传入的userRoles数组 和system(相当于Key)



  [1]: http://static.zybuluo.com/pockadmin/l2t0bdz6pgqov3bh0rp7hv6h/QQ%E6%88%AA%E5%9B%BE20180608093924.png
  [2]: http://static.zybuluo.com/pockadmin/ia48d317h4rgshic41rkqgqx/1.png