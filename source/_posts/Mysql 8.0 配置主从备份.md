---
layout: post
title: Mysql 8.0 配置主从备份
categories: Mysql
tags:
  - Mysql
abbrlink: 873aa52d
date: 2018-08-30 10:32:42
---

**新的架构需要读写分离,这里配置下主从备份**
<!--more-->

## my.ini文件的位置

- mysql 8.0安装完过后没有my.ini疑惑了我好久,最后发现,配置文件在,C盘的一个隐藏文件夹里面
![image_1cmkd9ofb1qql1pb4s14hkk1qjhp.png-19.4kB][1]

- 具体路径如下图
![image_1cmkddd4ekmr13631p4l75u1knl36.png-21.2kB][2]

## 主库配置

### 修改主库INI文件
- 在`[mysqld]`节点添加如下代码:
```ini
#主节点(Master)配置
# Binary Logging.
#二进制文件存放路径
log-bin=mysql-bin 

# Server Id.
#服务器 id
server-id=1 
```

- `mysql-bin`这个文件夹我是创建在我的mysql安装目录的,暂时不知道有没有用

### 主库创建复制操作用户
- 这个用户主要用于连接主库,进行复制操作

> 创建用户
mysql> CREATE USER '需要添加的用户名'@'从库IP地址' IDENTIFIED WITH mysql_native_password BY '用户密码';
修改权限
mysql> GRANT REPLICATION SLAVE ON *.* TO '用户名'@'从库IP地址';
刷新配置
mysql> flush privileges;

### 获取主节点当前binary log文件名和位置（position）
> mysql> SHOW MASTER STATUS;

- 一般结果如下图:
![image_1cmkea5t91h2mkvsd2clu25hd3j.png-8.7kB][3]

- 需要记录一下 `File` 和 `Position` 的字段信息

## 从库配置

### 配置INI文件
- 在`[mysqld]`节点添加如下代码:

```ini
#从节点(Master)配置
# Server Id.
server-id=2
```

### 从库设置主库参数
> mysql>CHANGE MASTER TO
        MASTER_HOST='主库IP地址',
        MASTER_USER='主库刚刚添加的用户名',
        MASTER_PASSWORD='密码',
        MASTER_LOG_FILE='记录的File值',
        MASTER_LOG_POS=记录的Position值(不加引号直接写数字);

### 开启同步
> mysql>start slave;

### 检查是否连接上主节点

> mysql>show slave status\G;

![image_1cmkfmujg1eei17pfsqh1nko16rn9.png-1.6kB][4]

- 这两个参数正常就OK了,接下来就是测试了

- 检查是否已链接上主节点，根据里面的错误信息修改配置。确保master防火墙关闭，确保`my.ing`里面的server-id不重复,`C:\ProgramData\MySQL\MySQL Server 8.0\Data`里面的`auto.cnf`里面的uuid不重复。


  [1]: http://static.zybuluo.com/pockadmin/09aehy7hs4qguqky6jiogwe2/image_1cmkd9ofb1qql1pb4s14hkk1qjhp.png
  [2]: http://static.zybuluo.com/pockadmin/mxibqo8hu1x57ycl99k5c6l1/image_1cmkddd4ekmr13631p4l75u1knl36.png
  [3]: http://static.zybuluo.com/pockadmin/thzd6ivu2jzxw2ietcxsquys/image_1cmkea5t91h2mkvsd2clu25hd3j.png
  [4]: http://static.zybuluo.com/pockadmin/s1l86sstfp9q48kiy1yb6z9h/image_1cmkfmujg1eei17pfsqh1nko16rn9.png