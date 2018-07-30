---
layout: post
title: CTF 题库记录
categories: CTF
tags:
  - CTF
abbrlink: f45ebcce
date: 2018-07-28 00:21:36
---

**争取每日一更吧,题库来源[实验吧](http://www.shiyanbar.com/ctf/practice)**
<!--more-->


## 0X01 2018-7-28(MD5注入)

- 这是个比较简单的一题

- 界面

![image_1cje8si1s96vck71ah61l57vnu9.png-8.6kB][1]

- 老规矩先看页面源码吧

```html

<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
</head>
<body style="background-color: #999">
	<div style="position:relative;margin:0 auto;width:300px;height:200px;padding-top:100px;font-size:20px;">
	<form action="" method="post">
		<table>
			<tr>
				请用管理员密码进行登录~~
			</tr>
			<tr>
				<td>密码：</td><td><input type="text" name='password'></td>
			</tr>
			<tr>
				<td><input type="submit" name='submit' style="margin-left:30px;"></td>
			</tr>
		</table>
	</form>
	﻿密码错误!	</div>
	<!-- $password=$_POST['password'];
	$sql = "SELECT * FROM admin WHERE username = 'admin' and password = '".md5($password,true)."'";
	$result=mysqli_query($link,$sql);
		if(mysqli_num_rows($result)>0){
			echo 'flag is :'.$flag;
		}
		else{
			echo '密码错误!';
		} -->
</body>
</html>

```

- 页面基于PHP,源码有句sql,如下:

```sql
SELECT * FROM admin WHERE username = 'admin' and password = '".md5($password,true)."'"
```
**存在MD5的注入情况**

- MD5注入

>先来说一下md5()这个函数
md5(string, raw)         raw 可选，默认为false
true:返回16字符2进制格式
false:返回32字符16进制格式

简单来说就是 true将16进制的md5转化为字符了,如果某一字符串的md5恰好能够产生如’or ’之类的注入语句，就可以进行注入了.

我需要把sql查询变成什么样呢?加入我正常情况输入123sql会变成什么样呢?`and password = 'MD5加密的数据"`,我这里进行注入的话最简单的需要查询`admin`表的所有用户,但是怎么查询呢?我需要`or`语句,我这里找了一篇资料来自[More Smoked Leet Chicken](http://mslc.ctf.su/wp/leet-more-2010-oh-those-admins-writeup/),里面提供了一个`string`字符串`ffifdyop`md5后 : 276f722736c95d99e921722cf9ed621c,转成字符串后 ：`'or'6<trash>`

> sql语句拼接为:
password = 'XX''or`'6<trash>`
`6<trash>` 在sql执行中把这节转从string转换为int到bool true

这样就完成了一个MD5的注入


## 2018-7-30 来自 [CG-CTF](https://cgctf.nuptsast.com/challenges#Web) 的签到题

> ![image_1cjla86si6cululki7n6as99.png-6.8kB][2]
![image_1cjla8ir91ev913ad1u671f6e1vr2m.png-9.7kB][3]


## 2018-7-30 来自 [CG-CTF](https://cgctf.nuptsast.com/challenges#Web) 的MD5加密题

> ![image_1cjlacuiopgi1pht1e291ap2jdt23.png-28.8kB][4]

- 看代码应该是用GET请求,输入一个a的值,被MD5加密过后等于QNKCDZO加密后的值,但是a不能为QNKCDZO

- 第一次我想到是hash冲突,但是转过去一想hash冲突的可能太小了,最有可能的是PHP MD5函数的漏洞,一查果不其然,下面总结下:
> **PHP在处理哈希字符串时，会利用”!=”或”==”来对哈希值进行比较，它把每一个以”0E”开头的哈希值都解释为0，所以如果两个不同的密码经过哈希以后，其哈希值都是以”0E”开头的，那么PHP将会认为他们相同，都是0。**
<br>
有下面一些常见的payload:
QNKCDZO
0e830400451993494058024219903391
<br>
s878926199a
0e545993274517709034328855841020
<br>
s155964671a
0e342768416822451524974117254469
<br>
s214587387a
0e848240448830537924465865611904
<br>
s214587387a
0e848240448830537924465865611904
<br>
s878926199a
0e545993274517709034328855841020
<br>
s1091221200a
0e940624217856561557816327384675
<br>
s1885207154a
0e509367213418206700842008763514
<br>
s1502113478a
0e861580163291561247404381396064
<br>
s1885207154a
0e509367213418206700842008763514
<br>
s1836677006a
0e481036490867661113260034900752
<br>
s155964671a
0e342768416822451524974117254469
<br>
s1184209335a
0e072485820392773389523109082030
<br>
s1665632922a
0e731198061491163073197128363787
<br>
s1502113478a
0e861580163291561247404381396064
<br>
s1836677006a
0e481036490867661113260034900752
<br>
s1091221200a
0e940624217856561557816327384675
<br>
s155964671a
0e342768416822451524974117254469
<br>
s1502113478a
0e861580163291561247404381396064
<br>
s155964671a
0e342768416822451524974117254469
<br>
s1665632922a
0e731198061491163073197128363787
<br>
s155964671a
0e342768416822451524974117254469
<br>
s1091221200a
0e940624217856561557816327384675
<br>
s1836677006a
0e481036490867661113260034900752
<br>
s1885207154a
0e509367213418206700842008763514
<br>
s532378020a
0e220463095855511507588041205815
<br>
s878926199a
0e545993274517709034328855841020
<br>
s1091221200a
0e940624217856561557816327384675
<br>
s214587387a
0e848240448830537924465865611904
<br>
s1502113478a
0e861580163291561247404381396064
<br>
s1091221200a
0e940624217856561557816327384675
<br>
s1665632922a
0e731198061491163073197128363787
<br>
s1885207154a
0e509367213418206700842008763514
<br>
s1836677006a
0e481036490867661113260034900752
<br>
s1665632922a
0e731198061491163073197128363787
<br>
s878926199a
0e545993274517709034328855841020

- 所以这道题:
![image_1cjlb9agj1sff1gfb1ls8i1nj7e2g.png-15.2kB][5]


## 2018-7-30 来自 [CG-CTF](https://cgctf.nuptsast.com/challenges#Web) 的签到题2

- 下面是题目:
![image_1cjlbd28d1p8e14g9t71ba912s52t.png-11.3kB][6]

- 尝试复制粘贴
![image_1cjlbebpabm1qq2s1ebjqo13q.png-6.6kB][7]

- 点击开门错了,想一下,密码又长又可以复制?不想让我对比? ok那我先把粘贴的密码展示出来

![image_1cjlbie931tssl1i7crce4it57.png-27.4kB][8]

- 有点小阴险啊 密码长度手动改成11,在输入个n

![image_1cjlbkrdf4hc1lq51a8j4r3158e9.png-10kB][9]

## 2018-7-30 来自 [CG-CTF](https://cgctf.nuptsast.com/challenges#Web) 题目"这不是web"

- 下面是题目:
![image_1cjlc5bbd1p9eov21th85oo1g09m.png-20.1kB][10]
- 下面是进去过后的样子
![image_1cjlc5je915rn1u921i5k2geco13.png-28.5kB][11]

- 一开始我找的源码没发现什么,在去找了找有没有目录遍历,也没有,后来我突发奇想**GIF里面保存了信息?**
我把图片保存下来,百度了一下GIF怎么保存string,没资料,后来我想,直接保存为txt试试?就出现了下面的情况
![image_1cjlcb4f61ghm13901nss1ouq1su1g.png-395.7kB][12]

 - OK了,题目蛮有意思的,哈哈哈


  [1]: http://static.zybuluo.com/pockadmin/amqaztntropom56fhkgpz51p/image_1cje8si1s96vck71ah61l57vnu9.png
  [2]: http://static.zybuluo.com/pockadmin/au1co3a4abbz5ia0bhqwmuti/image_1cjla86si6cululki7n6as99.png
  [3]: http://static.zybuluo.com/pockadmin/ni21zpllgzpfrzfmyav94gd3/image_1cjla8ir91ev913ad1u671f6e1vr2m.png
  [4]: http://static.zybuluo.com/pockadmin/b3udif9mnqduz60dguya4yzg/image_1cjlacuiopgi1pht1e291ap2jdt23.png
  [5]: http://static.zybuluo.com/pockadmin/bayh6jxusdlcnx5p53guqo3l/image_1cjlb9agj1sff1gfb1ls8i1nj7e2g.png
  [6]: http://static.zybuluo.com/pockadmin/a63ni13xv93tx0gdj1b2kora/image_1cjlbd28d1p8e14g9t71ba912s52t.png
  [7]: http://static.zybuluo.com/pockadmin/yaifuatoovq023nmbn3c4m48/image_1cjlbebpabm1qq2s1ebjqo13q.png
  [8]: http://static.zybuluo.com/pockadmin/u5ffd0lb79c6ics24tpksodi/image_1cjlbie931tssl1i7crce4it57.png
  [9]: http://static.zybuluo.com/pockadmin/j02dcrsfqwsslygw7pu95v2q/image_1cjlbkrdf4hc1lq51a8j4r3158e9.png
  [10]: http://static.zybuluo.com/pockadmin/09h1sea2ggbomydxt6ehafcv/image_1cjlc5bbd1p9eov21th85oo1g09m.png
  [11]: http://static.zybuluo.com/pockadmin/p2bilmrt16zmozmd7rai0vd5/image_1cjlc5je915rn1u921i5k2geco13.png
  [12]: http://static.zybuluo.com/pockadmin/1y4es0ojnjugylz7rbkbxhol/image_1cjlcb4f61ghm13901nss1ouq1su1g.png