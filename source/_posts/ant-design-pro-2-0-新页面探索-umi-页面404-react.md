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

![在这里插入图片描述](http://i2.tiimg.com/691643/86ffd9a4cddc8c0a.png)

这个是官方的介绍。根据提示我开始加入我的路由


![在这里插入图片描述](http://i2.tiimg.com/691643/27a1b09e06e2c77b.png)

我在最后面另起一行，加入了我的新的路由。名字叫做Test。

component是你具体路由的实际位置（根路径是pages）


![在这里插入图片描述](http://i2.tiimg.com/691643/25d25bc09010eb44.png)

Ctrl + S

看效果！

![在这里插入图片描述](http://i2.tiimg.com/691643/c56a7c3c9d20f79a.png)

 - 菜单是出来了
 - 为什么404！！
 
 通过4个多小时的google，在这位兄台的指点下。我知道了。


新页面要写在404之前。

![在这里插入图片描述](http://i2.tiimg.com/691643/372f07cd5d3793e8.png)

正常的新手，都是在最后加入一个新的路由。。。不知道这个坑爹的问题是为什么。我马上去质问了一下ant design pro的作者之一。得到了解答！！！非常的激动！！


![在这里插入图片描述](http://i2.tiimg.com/691643/4e4507f49ce412da.png)


所以我们只要在404之前添加新的页面就可以了

![在这里插入图片描述](http://i2.tiimg.com/691643/2daa11d263e0d7df.png)


![在这里插入图片描述](http://i2.tiimg.com/691643/07a82014b4b87ba8.png)

补充：
是因为React-Router规定的。