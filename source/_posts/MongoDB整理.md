
---
title: "MongoDB整理"
date: 2018-6-13 10:16:28
categories: MongoDB整理
tags: 
		- MongoDB
---


**初学MongoDB记录下遇见的语句**
<!-- more -->

## 数据库导出
- 举例
> `mongodump -h IP地址:MongoDB端口 -u MongoDB账户 -p MongoDB密码  -d 数据库名 -c 表名 -o /data/programs/bk/tasks.json(这里是导出的路径)`

- `-h` 为需要导出的数据库地址注意加端口号
- `-u` 为MongoDB的账号
- `-p` 为MongoDB密码
- `-d` 需要导出的数据在哪一个库
- `-c` 需要导出的表名
- `-o` 导出到哪里(可以控制导出的类型修改后缀名)


## 常用语句

### **`db.businesorders.find().pretty()`**

- 查看某个数据库的数据

### **`$gte`, `$gt`, `$lte`, `$lt`**

- `$gte` 大于等于,
- `$gt` 大于,
- `$lte`小于等于,
- `$lt`小于

### **`$match`**

- 比如说我查询之前需要根据时间筛选一次就需要这个,下面是例子

```js
{$match : {'created' : {$gte : 1527782400000,$lte : 1530374400000}}}
```

### **`$group`**

- 分组

> { $group: { _id: <expression>, <field1>: { <accumulator1> : <expression1> }, ... } }

第一个字段必须是`_id`,对应的就是分类的字段,field随便取

### 展示所有table

- `show tables`

### **`$skip`,`$limit`**

- 分页语句,下面是例子

```js
//$skip 跳过前面的数据  from就是前面几页的数据
//$limit 和mysql一样
const page = Number(pagin.page || 1);
const pageSize = Number(pagin.pageSize || 10);
const from = (page - 1) * pageSize;
{ $skip: from },
{ $limit: pageSize },
```

### **`$unwind`**

- 用来处理数组,下面举例 service

![image_1cgik7p5tg4l12n21kadqdof8j9.png-6.5kB][1]

这里为数组结构,我分组的时候需要用某个service中的字段进行`$$group`,但是数组结构没办法直接用`.`点出来,要把数组进行处理,这里就用`$unwind`处理,下面是代码

```js
{$unwind : '$service'},
{$group : {
    _id : {isHCS : '$target.isHCS' , serviceType : "$service.category"}, 
    price : {$sum : '$price.value'}
        }
    }
]) 
```

### **`$push`**

- 在`$group`的时候在集合里面放一个数组,只能push数组

```js
db.projectsms.aggregate([
{$unwind: '$phones'},
	{
                $group: {
                    '_id': '$_id',
                    'name': {$push : '$role'},
					'state': {$push : '$state'},
					'projects': {$push : '$projects._id'},
					'phones': {$push: '$phones'}
                }
            }
])
```

### **`find`语句**

- 使用mongoose工具的时候可以使用一下方式进行过滤,不需要使用aggregate(聚合进行字段筛选):

```js
//params是筛选条件,{name: 1, sex:1, identityNumber: 1}意思是只要这些字段
//默认_id字段也是返回的。如果需要去掉加上{name: 1, sex:1, identityNumber: 1, _id: 0}
let content = await this.model.find(params,{name: 1, sex:1, identityNumber: 1});
```

### **`updata`语句**

```js
 await this.model.update(
                {这里可以写条件和find的一样}
                {$set: {'uuid': mongoose.Types.ObjectId().toString()}}
            );
```

### **`count`语句**

```js
await this.model.count(query); # query里面是查询条件,返回数量
```

### **`$set` 语句**

- 假如有一下数据:
- 
```js
//第一条
{"_id": 1, "imgs": [7, 9, 2]},
//第二条
{"_id": 2, "imgs": [17, 56, 8]},
//第三条
{"_id": 3, "imgs": [93, 11, 534]}
```

- 这里有个需求,我想要更新**_id**为2,把**imgs**节点中元素为56的跟换为99

```js
> db.students.update({_id:2, imgs:99},{$set: {'imgs.$':56} })
```

**请记住，位置$操作符充当更新文档查询中第一个匹配的占位符。**

- 假如数据变化稍微复杂,结构如下:

```js
{
  _id: 1,
  imgs: [
     { "header": 12, "body": 32, "food": 67 },
     { "header": 545, "body": 77, "food": 31 },
     { "header": 54, "body": 22, "food": 65 }
  ]
}
```

- 有个需求,我需要更新嵌套文档中**_id**为1中的**imgs**节点中的的**body**字段为77的那一条数据中的**food**改为1

```js

db.xxxx.update(
   { _id: 1, "imgs.body": 77 }, //这一个管道为条件
   { $set: { "imgs.$.body" : 77 } } //这一个管道为修改内容,$只会修改满足条件的第一个

```

- 还是上面的数据结构,这里又有一个需求,我需要更新嵌套文档中**_id**为1中的**imgs**节点中的的**body**字段为**大于等于**77,**header**小于等于500的那一条数据中的**food**改为1

```js

db.xxxx.update(
   { _id: 1, "imgs": { $elemMatch : {body : { $gte : 77 }}, {header : { $lte : 500 }} } }, //这一个管道为条件
   { $set: { "imgs.$.food" : 77 } } //这一个管道为修改内容,$只会修改满足条件的第一个

```

**$elemMatch()操作符匹配多个内嵌文档的查询条件**

  [1]: http://static.zybuluo.com/pockadmin/6kku392uv4ws1e4yvawi5knr/image_1cgik7p5tg4l12n21kadqdof8j9.png