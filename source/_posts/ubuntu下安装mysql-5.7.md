
---
layout: post
title: "ubuntu下安装mysql-5.7"
date: 2018-7-19 10:19:55
categories: Ubuntu
tags: 
		- Ubuntu
---

**今天看我盟哥在搭建到耍我也耍一哈,顺便记录下**
<!-- more -->

## 0X00 第一步

> `sudo apt-get install mysql-server`
会安装以下依赖:
apparmor 
mysql-client-5.7 
mysql-common 
mysql-server 
mysql-server-5.7 
mysql-server-core-5.7 

无需再安装mysql-client等,安装过程会提示输入用户名,输入完毕回车,如下
![image_1cio5ibn3kvp14q5f5v1go31t3d9.png-35.9kB][1]

输入完毕会提示输入用户密码,输入完毕回车,如下
![image_1cio5jkulv7g1fbs1f0k1gp71e3vm.png-24.8kB][2]

## 0X01 安装完成

- 查看是mysql是否占用端口,处于 LISTEN 说明就OK了(类似于看是否启动)
> 'sudo netstat -tap | grep mysql'
![image_1cio5p6vn9bomf1ma185jelf13.png-14.7kB][3]

## 0X02 localhost登录mysql服务

- 本机登录测试 (这里root是你的用户名)
> 'mysql -u root -p' 
![image_1cio5tnoj1o3cs9j6glm4p1rna1g.png-43.1kB][4]

## 0X03 常见命令

> 开启mysql `service mysql start`
关闭mysql `service mysql stop`
重启mysql `service mysql restart`

##  0X04 常用配置

### 1. 允许远程访问
- 首先修改文件 
> `sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf` 需要修改的地方在第43行左右
![image_1cio6cih111hdrc414ggg80n4s2a.png-68kB][5]
**点击`i`进入编辑模式,编辑完按`esc`退出编辑模式,在按`shift加冒号`(shift + :),输入`wq`保存退出**
![image_1cio6didq1upl1li8jjqjqvs6m2n.png-3.2kB][6]

- 保存退出,执行授权命令：
进入mysql服务`mysql -u root -p`, 执行`
grant all on *.* to root@'%' identified by '你的密码' with grant option;`**(记住复制分号,意思是赋予任何主机访问数据库权限)**
刷新`flush privileges;`**(记住复制分号)**
![image_1cio6qvqja2s1aump8drqaacp34.png-18.7kB][7]

- 然后执行quit命令退出mysql服务，执行如下命令重启mysql：

> `service mysql restart` #重启mysql
可能会叫你输入用户密码(不是mysql的密码,你是登录服务器的密码)
![image_1cio70csd1i8312a31ik512791gek3h.png-24kB][8]

- 测试连接
![QQ图片20180719104959.png-24.7kB][9]

### 2. 添加远程用户

- 输入以下命令
> `mysql -u root -p` #进入mysql
`grant all on *.* to pock@'%' identified by '123456';` #添加一个用户名为pock 密码为123456的用户
`flush privileges;` #刷新配置
**输入quit退出mysql**
`service mysql restart` #重启mysql
![222.png-23.3kB][10]

- **如果遇到新添加的远程用户无法登录的情况进入mysql输入以下命令**
> grant all privileges  on *.* to root@'%' identified by "root用户的密码";

### 修改远程用户密码
> `mysql -u root -p` #进入mysql
`use mysql;` 使用用户表
`update user set password=password('新密码') where user='需要修改的用户';`
`flush privileges;`刷新配置

## 0X05 安全配置

### 防火墙不多说

### 配置账号权限

- 禁止root账号远程登录
> `update user set host = "localhost" where user = "root" and host= "%";
flush privileges;`

- 禁止项目用户的账号远程登录（项目配置文件）
> `update user set host = "localhost" where user = "web_user" and host= "%";`
`flush privileges;`

同时绑定该账号只允许内网访问（如内网IP：172.19.230.1）
> `update user set host = "172.19.230.1" where user = "web_user" and host= "localhost";`
`flush privileges;　`

- 通过内网连接数据
> `mysql -h 172.19.230.1 -u web_user -p`

- 针对远程需要登录的可以重新新建一个用户用于远程连接数据库
> `grant all on *.* to yc_user@'%' identified by 'yc123456' with grant option;`
`flush privileges;`

- 说一下**where user = "web_user"** web_user的权限分类
> 1.root 禁止远程访问
2.web_user 只允许内网访问，该用户暴露在项目代码中
3.yc_user 可以远程访问，该用户不应该暴露在项目代码中

## 0X06 服务管理

- 不建议用`chkconfig`,这里使用`sysv-rc-conf 
- 安装sysv-rc-conf
> `sudo apt-get install sysv-rc-conf`

- 直接加入启动程序，例如把 /etc/init.d/mysql 加入到系统自动 启动列表中：
> `sudo sysv-rc-conf mysql on`   #开启
`sudo sysv-rc-conf mysql off`  #关闭

- 直接使用 `sysv-rc-conf` 来管理安装命令如下:
> `sudo sysv-rc-conf`

- 界面如下:
![image_1cio993uptjv1uv2rh3aim1kq066.png-57.4kB][11]

- 介绍下快捷键
>使用空格键可以在on和off之间切换
+号可以启动服务
-号可以停止服务
ctrl + n 翻到下一页
ctrl + p 翻到上一页
h可以查看帮助
q退出

- 介绍下级别(运行级别的详情不太懂)
>
0 停机
>
1 单用户，Does not configure network interfaces, start daemons, or allow non-root logins
>
2 多用户，无网络连接 Does not configure network interfaces or start daemons
>
3 多用户，启动网络连接 Starts the system normally.
>
4 用户自定义
>
5 多用户带图形界面
>
6 重启


  [1]: http://static.zybuluo.com/pockadmin/qi03khdqkm0lqi82febr1qfm/image_1cio5ibn3kvp14q5f5v1go31t3d9.png
  [2]: http://static.zybuluo.com/pockadmin/szcjbiwl6m9rj7otsaxhyx53/image_1cio5jkulv7g1fbs1f0k1gp71e3vm.png
  [3]: http://static.zybuluo.com/pockadmin/fatl6dhzojiov3v8zxrbrq0r/image_1cio5p6vn9bomf1ma185jelf13.png
  [4]: http://static.zybuluo.com/pockadmin/4m2gggwwpz68aiufhrue6az7/image_1cio5tnoj1o3cs9j6glm4p1rna1g.png
  [5]: http://static.zybuluo.com/pockadmin/p6f1vklospsc6ias36ouehis/image_1cio6cih111hdrc414ggg80n4s2a.png
  [6]: http://static.zybuluo.com/pockadmin/d2s1z5wmfpwm63sptgqfhwat/image_1cio6didq1upl1li8jjqjqvs6m2n.png
  [7]: http://static.zybuluo.com/pockadmin/ojrntgdeazgeuhvpiuhvklw2/image_1cio6qvqja2s1aump8drqaacp34.png
  [8]: http://static.zybuluo.com/pockadmin/lqxbfgp1gsqf21zr5o980ig8/image_1cio70csd1i8312a31ik512791gek3h.png
  [9]: http://static.zybuluo.com/pockadmin/y0zr6exy5v4ft7vqfprwce2i/QQ%E5%9B%BE%E7%89%8720180719104959.png
  [10]: http://static.zybuluo.com/pockadmin/rnb79hdsscbmx9i787ncwqyu/222.png
  [11]: http://static.zybuluo.com/pockadmin/fbhpkemt9bb6t9k133o4ltwk/image_1cio993uptjv1uv2rh3aim1kq066.png