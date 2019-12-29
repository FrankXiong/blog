---
title: 部署 Node 项目到 CentOS 服务器
date: 2016-04-05 14:15:56
tags:
- 运维
- Node.js
- CentOS
categories:
- 后端
---
最近开始折腾Node，跟着慕课网的教程写了个电影网站，于是想把网站部署到服务器上，本文记录了我整个环境搭建的流程。

通常Node和mongoDB一起搭配使用，再加上Node的一个热门的开发框架Express，以及angular.js，共同构成了整个web开发的技术架构（这次的开发中没有用到angular）。取其首字母，也就是所谓的"MEAN"。不废话了，下面是正文。

-----------

# 服务器配置
- 阿里云ECS 单核1G内存（这里要安利一下阿里云的学生优惠活动，一个月只要￥10，学生党的福利~）
- 操作系统：CentOS 7.0 64位

首先SSH连接服务器管理终端：
![](https://mares.oss-cn-qingdao.aliyuncs.com/blog/node-deploy/1.png)

# 安装Node
Node.JS的安装方法很多，这里贴上一种方法以供参考。
http://yijiebuyi.com/blog/4fcce2f8b1aed8389f34c27f22864a04.html

# 安装MongoDB
在MongoDB官网上看了下，没找到在centOS直接用apt-get安装mongo的方法，那就手动来下载安装吧。
1.输入以下命令：curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-3.2.4.tgz
mongoDB就开始下载了，也可以用wget来下载。(下载过程比较缓慢，不知道是我的网速还是curl的问题...)

![](https://mares.oss-cn-qingdao.aliyuncs.com/blog/node-deploy/2.png)
2.下载结束后解压文件：tar xf mongodb-linux-x86_64-rhel70-3.2.4.tgz
文件名太长了，重命名一下：mv mongodb-linux-x86_64-rhel70-3.2.4  mongodb
3.进入mongodb文件夹，新建logs文件夹，并在其下创建一个mongodb.log文件用于保存日志。创建data文件夹，在data文件夹下再新建db文件夹，用于存储mongoDB的数据。

![](https://mares.oss-cn-qingdao.aliyuncs.com/blog/node-deploy/3.png)
4.添加环境变量

![](https://mares.oss-cn-qingdao.aliyuncs.com/blog/node-deploy/4.png)
5.重新加载环境变量,验证结果。
用mongod -verison或者-v看到下面的结果，就证明mongoDB安装成功了

![](https://mares.oss-cn-qingdao.aliyuncs.com/blog/node-deploy/5.png)
# 上传项目文件到服务器
Mac上可以用scp上传，windows上用FTP。FTP上传工具很多，随意选一种即可。
# 启动MongoDB
进入mongo目录的bin文件夹，输入如下命令，dbpath后指定的是Node项目的路径，这样就可以直接通过该项目启动数据库
```mongod --dbpath "/developer/mongodb/imooc"```
# 连接MongoDB
在Node项目根目录下输入mongo命令就可以建立与数据库的连接。另外，如果你前面没有指定在启动mongoDB的时候指定项目路径的话，你就还需要使用use命令建立两者的关联。当时我忘了这一点，于是注册后的帐号等数据都没有被保存到数据库中。

-----------
下面就能看到网站欢快地跑起来了~~~因为没做域名解析，暂时只能通过IP地址来访问
附一张这个网站的截图

![电影详情页](https://mares.oss-cn-qingdao.aliyuncs.com/blog/node-deploy/6.png)

另：网站的Github地址 https://github.com/FrankXiong/imooc
