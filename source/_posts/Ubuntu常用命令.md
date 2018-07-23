---
title: Ubuntu常用命令
categories: Ubuntu常用命令
tags:
  - Ubuntu
abbrlink: fa6c14b0
date: 2018-06-19 14:03:49
---

**Ubuntu常用命令**
<!-- more -->

## 查看磁盘空间大小命令
 > df -hl 查看磁盘剩余空间  
df -h 查看每个根路径的分区大小  
du -sh [目录名] 返回该目录的大小  
 du -sh * 查看目录下每个文件夹的大小
du -sm [文件夹] 返回该文件夹总M数  
df --help 查看更多功能 

## 查看文件修改时间（精确到秒）
> ls --full-time

## 删除文件

> rm [参数] [文件名]
-r 就是向下递归，不管有多少级目录，一并删除
-f 就是直接强行删除，不作任何提示的意思

## 修改权限

> sudo chown www:www ./ -R

