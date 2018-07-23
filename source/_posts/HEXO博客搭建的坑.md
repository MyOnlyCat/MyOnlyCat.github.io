---
layout: post
title: HEXO博客搭建的坑
categories: HEXO
tags:
  - HEXO
abbrlink: 254c3d71
date: 2018-05-18 10:37:28
---


**记录下第二次搭建博客的流程**
<!-- more -->

### 前景

- 原来的博客换了电脑就没法更新了,很麻烦,这里我使用分支的方式进行部署,顺便记录一下自己遇到问题

### 0X00 环境配置

- 配置nodejs
- 安装git命令客户端
- 配置git,命令为 `git config --global user.name "你的github名字"` , `git config --global user.email "你的github邮箱"`

### 0X01 GitHub仓库创建

- **第一步**

 ![image_1ci6q97fu1b1312cm161sfgfd619.png-26.2kB][1]

- **第二步**

**注意这里名字要用自己的GitHub用户名加.github.io,才能让GitHub自动给你分配githubpage**

![image_1ci6qb9b95st9ml1t04gmj1shp1m.png-22.7kB][2]

- **第三步**

在仓库里面随便创建一个REMDME文件,方便修改分支

- **第四步**

新建一个分支

![image_1ci6qh0v81rql124jqbg4ti11fc23.png-27.4kB][3]

设置为默认分支

![image_1ci6qiorh1gaotacoupnc36u22g.png-54.9kB][4]

### 0X02 配置HEXO

- **第一步**

使用git把你的默认分支clone下来(直接复制地址clone肯定是你的默认分支)

- **第二步**

    - 进入项目目录在git用使用`hexo init blog`
    - 把blog文件中的东西复制到你的项目目录去(如果blog文件中有`.git`的话就删除在复制就完了)

- **第三步**
    - 1. 使用github的windows客户端把修改提交到分支

### 0X03 部署

- 'hexo g'
- `hexo s` //启动本地服务,方便看效果
- `hexo d` //提交到分支了,然后master分支就合并了
- 上一步可能存在一些问题,我平常更新是结合github的Windows客户端进行提交,也就是我`hexo s`启动过后看效果,ok的话我就直接用客户端进行上传,上传完成过后访问线上网站并不会更新,我需要点击

![1.png-11.2kB][5] 这个按钮才会更新线上的(原因我没研究,怪我github用的太少)

### 0X04 遇到的问题

> - `ERROR Plugin load failed: hexo-server`
解决:  **`sudo npm install hexo-server`**

> - `hexo-renderer`相关的错误
解决: **`npm install hexo-renderer-ejs --save`**
    **`npm install hexo-renderer-stylus --save`**
    **`npm install hexo-renderer-marked --save`**
    这个时候再重新生成静态文件，命令：
   ** `hexo generate`** （或**`hexo g`**）
    启动本地服务器：
    **`hexo server`** （或**`hexo s`**）

    


  [1]: http://static.zybuluo.com/pockadmin/5bov22g8g7u90ldk9bd4uzl7/image_1ci6q97fu1b1312cm161sfgfd619.png
  [2]: http://static.zybuluo.com/pockadmin/qc8xnzya3s2zltvkq84ydv8e/image_1ci6qb9b95st9ml1t04gmj1shp1m.png
  [3]: http://static.zybuluo.com/pockadmin/t8j4pl4h907ykwztmc478ocs/image_1ci6qh0v81rql124jqbg4ti11fc23.png
  [4]: http://static.zybuluo.com/pockadmin/v6ij1rfprxct4ermwqrdvd8u/image_1ci6qiorh1gaotacoupnc36u22g.png
  [5]: http://static.zybuluo.com/pockadmin/sn6n90yr96y9vlrbwt80j1lh/1.png