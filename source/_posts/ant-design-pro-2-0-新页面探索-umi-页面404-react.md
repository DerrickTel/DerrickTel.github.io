---
title: ant design pro 2.0 新页面探索 umi 页面404 react
date: 2019-06-19 11:16:30
tags: [React-Router, UMI, Ant-Design-Pro]
category: [Ant-Design-Pro]
cover: /image/cover/antdP.png
---

## 前言

		ant design pro 2.0发布了
	
		使用umi作为路由配置，全自动化。

## 开始

![在这里插入图片描述](/image/AntDesignPro探索/20190312150742764.png)

这个是官方的介绍。根据提示我开始加入我的路由


![在这里插入图片描述](/image/AntDesignPro探索/20190312151053807.png)

我在最后面另起一行，加入了我的新的路由。名字叫做Test。

component是你具体路由的实际位置（根路径是pages）


![在这里插入图片描述](/image/AntDesignPro探索/20190312150929439.png)

Ctrl + S

看效果！

![在这里插入图片描述](/image/AntDesignPro探索/2019031215120430.png)

 - 菜单是出来了
 - 为什么404！！

 通过4个多小时的google，在这位兄台的指点下。我知道了。


新页面要写在404之前。

![在这里插入图片描述](/image/AntDesignPro探索/2019031215140926.png)

正常的新手，都是在最后加入一个新的路由。。。不知道这个坑爹的问题是为什么。我马上去质问了一下ant design pro的作者之一。得到了解答！！！非常的激动！！


![在这里插入图片描述](/image/AntDesignPro探索/2019031215175956.png)


所以我们只要在404之前添加新的页面就可以了

![在这里插入图片描述](/image/AntDesignPro探索/20190312151839845.png)


补充：
是因为React-Router规定的。