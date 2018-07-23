---
title: Git整理
categories: Git
tags:
  - Git
abbrlink: 19a5d6d0
date: 2018-06-13 11:16:19
---

**记录一下容易忘记的GIT命令**
<!-- more -->

## 创建SSH Keys

- 输入指令，进入.ssh文件夹

> cd ~/.ssh/ 
如果提示 “ No such file or directory”，你可以手动的创建一个 .ssh文件夹即可

- 配置全局的name和email，这里是的你github或者bitbucket的name和email

> git config --global user.name "pockadmin"  
  
> git config --global user.email "pockadmin@163.com"  

- 生成key

> ssh-keygen -t rsa -C “pockadmin@163.com”

- 连续按三次回车，这里设置的密码就为空了，并且创建了key。

- 打开Admin目录进入.ssh文件夹，用记事本打开id_rsa.pub，复制里面的内容添加到你github或者bitbucket ssh设置里即可
