---
title: ant design pro 2.0 新页面探索 umi 页面404 react
date: 2019-06-19 11:16:30
tags: [React-Router, UMI, Ant-Design-Pro]
category: [Ant-Design-Pro]
---

## 前言

		ant design pro 2.0发布了
	
		使用umi作为路由配置，全自动化。

## 开始

![在这里插入图片描述](https://github.com/DerrickTel/DerrickTel.github.io/blob/master/img/ant%20design%20pro%202.0%20%E6%96%B0%E9%A1%B5%E9%9D%A2%E6%8E%A2%E7%B4%A2%20umi%20%E9%A1%B5%E9%9D%A2404%20react/20190312150742764.png?raw=true)

这个是官方的介绍。根据提示我开始加入我的路由


![在这里插入图片描述](https://github.com/DerrickTel/DerrickTel.github.io/blob/master/img/ant%20design%20pro%202.0%20%E6%96%B0%E9%A1%B5%E9%9D%A2%E6%8E%A2%E7%B4%A2%20umi%20%E9%A1%B5%E9%9D%A2404%20react/20190312150929439.png?raw=true)

我在最后面另起一行，加入了我的新的路由。名字叫做Test。

component是你具体路由的实际位置（根路径是pages）


![在这里插入图片描述](http://i2.tiimg.com/691643/25d25bc09010eb44.png)

Ctrl + S

看效果！

![在这里插入图片描述](https://github.com/DerrickTel/DerrickTel.github.io/blob/master/img/ant%20design%20pro%202.0%20%E6%96%B0%E9%A1%B5%E9%9D%A2%E6%8E%A2%E7%B4%A2%20umi%20%E9%A1%B5%E9%9D%A2404%20react/2019031215120430.png?raw=true)

 - 菜单是出来了
 - 为什么404！！

 通过4个多小时的google，在这位兄台的指点下。我知道了。


新页面要写在404之前。

![在这里插入图片描述](https://github.com/DerrickTel/DerrickTel.github.io/blob/master/img/ant%20design%20pro%202.0%20%E6%96%B0%E9%A1%B5%E9%9D%A2%E6%8E%A2%E7%B4%A2%20umi%20%E9%A1%B5%E9%9D%A2404%20react/2019031215140926.png?raw=true)

正常的新手，都是在最后加入一个新的路由。。。不知道这个坑爹的问题是为什么。我马上去质问了一下ant design pro的作者之一。得到了解答！！！非常的激动！！


![在这里插入图片描述](https://github.com/DerrickTel/DerrickTel.github.io/blob/master/img/ant%20design%20pro%202.0%20%E6%96%B0%E9%A1%B5%E9%9D%A2%E6%8E%A2%E7%B4%A2%20umi%20%E9%A1%B5%E9%9D%A2404%20react/2019031215175956.png?raw=true)


所以我们只要在404之前添加新的页面就可以了

![在这里插入图片描述](https://github.com/DerrickTel/DerrickTel.github.io/blob/master/img/ant%20design%20pro%202.0%20%E6%96%B0%E9%A1%B5%E9%9D%A2%E6%8E%A2%E7%B4%A2%20umi%20%E9%A1%B5%E9%9D%A2404%20react/20190312151839845.png?raw=true)


![在这里插入图片描述](http://i2.tiimg.com/691643/07a82014b4b87ba8.png)

补充：
是因为React-Router规定的。