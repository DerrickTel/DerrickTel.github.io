---
title: Hexo+GitHub构建你的博客
date: 2019-06-17 19:17:25
tags: [Hexo, GitHub]
categories: [博客]
---

## 前言

 - 准备工作
1.[Node.js](http://nodejs.cn/)
2.[GitHub](https://github.com/)账号
3.[Git](https://git-scm.com/)
 - 了解....
 ![在这里插入图片描述](http://i2.tiimg.com/691643/176e52550ac47565.png)
 
 好的。直接开始（对特定知识点不同这里不多做解释咯，自行Google / Baidu）
 ## 开始
 - GitHub新建一个项目

Github账户注册和新建项目，项目必须要遵守格式：账户名.github.io，不然接下来会有很多麻烦。并且需要勾选Initialize this repository with a README
![在这里插入图片描述](http://i2.tiimg.com/691643/275383f15d4e18ee.png)
本人只是因为之前创建过了。

 - 安装Hexo


选一个自己喜欢的文件夹
通过命令行进入这个文件夹（
	1.Git Bash Here
	2.一步步cd 进去
）

输入npm install hexo -g，开始安装Hexo
![在这里插入图片描述](http://i2.tiimg.com/691643/f4d88abbbffb8b26.png)

输入hexo -v，检查hexo是否安装成功
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190618161313335.png)

输入hexo init，初始化该文件夹（有点漫长的等待。。。）
![在这里插入图片描述](http://i2.tiimg.com/691643/bf771334849880b4.png)

输入npm install，安装所需要的组件

![在这里插入图片描述](http://i2.tiimg.com/691643/2b802d0c92647f1b.png)

输入hexo g，首次体验Hexo

![在这里插入图片描述](http://i2.tiimg.com/691643/2ed3473551aab22e.png)

 输入hexo s，开启服务器，访问该网址，正式体验Hexo
 ![在这里插入图片描述](http://i1.fuimg.com/691643/a6f72c64779f466e.png)
问题：假如页面一直无法跳转，那么可能端口被占用了。此时我们ctrl+c停止服务器，接着输入“hexo server -p 端口号”来改变端口号

![在这里插入图片描述](http://i1.fuimg.com/691643/9101d7ccda09e3ec.png)

那么出现如下图就成功了

![在这里插入图片描述](http://i1.fuimg.com/691643/d764bd2849b04c6b.png)

 - Hexo与GitHub联系起来（如果是第一次需要设置user_name和email）

![在这里插入图片描述](http://i1.fuimg.com/691643/b0e1e23b2e878232.png)
![在这里插入图片描述](http://i1.fuimg.com/691643/561dcbf64128cc6f.png)
 
 输入cd ~/.ssh，检查是否由.ssh的文件夹
 
![在这里插入图片描述](http://i1.fuimg.com/691643/59047f43a9649a21.png)

输入ls，列出该文件下的内容。下图说明存在
![在这里插入图片描述](http://i1.fuimg.com/691643/52e0017348db9800.png)
 
 输入ssh-keygen -t rsa -C “547791743@qq.com”，连续三个回车，生成密钥，最后得到了两个文件：id_rsa和id_rsa.pub（默认存储路径是：C:\Users\${用户名不一定是Administrator}\.ssh）。
![在这里插入图片描述](http://i1.fuimg.com/691643/cf81a1ac2297f649.png)

 输入eval "$(ssh-agent -s)"，添加密钥到ssh-agent

 ![在这里插入图片描述](http://i1.fuimg.com/691643/362d1a0fa087f8e1.png)
 
 再输入ssh-add ~/.ssh/id_rsa，添加生成的SSH key到ssh-agent
 
 ![在这里插入图片描述](http://i1.fuimg.com/691643/ee612144d905e92a.png)
 
 登录Github，点击头像下的settings，添加ssh
 
 ![在这里插入图片描述](http://i1.fuimg.com/691643/8080b0e05bd7ae99.png)
 ![在这里插入图片描述](http://i1.fuimg.com/691643/17dafc9683e3c8c5.png)
新建一个new ssh key，将id_rsa.pub文件里的内容复制上去

![在这里插入图片描述](http://i1.fuimg.com/691643/f7fd2ca386ed0114.png)

输入ssh -T git@github.com，测试添加ssh是否成功。如果看到Hi后面是你的用户名，就说明成功了

![在这里插入图片描述](http://i1.fuimg.com/691643/6317ee227df4d8c0.png)

**问题**：假如ssh-key配置失败，那么只要以下步骤就能完全解决

首先，清除所有的key-pair

ssh-add -D

rm -r ~/.ssh

删除你在github中的public-key


重新生成ssh密钥对

ssh-keygen -t rsa -C "xxx@xxx.com"

接下来正常操作
在github上添加公钥public-key:

1、首先在你的终端运行 xclip -sel c ~/.ssh/id_rsa.pub将公钥内容复制到剪切板

2、在github上添加公钥时，直接复制即可

3、保存


测试：
在终端 ssh -T git@github.com

 - 配置Deployment，在其文件夹中，找到_config.yml文件，修改repo值（在末尾）

![在这里插入图片描述](http://i1.fuimg.com/691643/0304ceda26c63426.png)
![在这里插入图片描述](http://i1.fuimg.com/691643/469d154a33bf334d.png)

 - 新建一篇博客，在cmd执行命令：hexo new post “博客名”

![在这里插入图片描述](http://i1.fuimg.com/691643/cbfd492024d09457.png)


![在这里插入图片描述](http://i1.fuimg.com/691643/f58d97a33dbf5d39.png)

在生成以及部署文章之前，需要安装一个扩展：npm install hexo-deployer-git --save

![在这里插入图片描述](http://i1.fuimg.com/691643/df023318dbec70e7.png)


使用编辑器编好文章，那么就可以使用命令：hexo d -g，生成以及部署了


![在这里插入图片描述](http://i1.fuimg.com/691643/8a30a091a1525296.png)


部署成功后访问你的地址：http://用户名.github.io。那么将看到生成的文章


好了，到此为止，最基本的也是最全面的hexo+github搭建博客完结。接下来是进阶的操作


欢迎大家访问我自己搭的博客
https://derricktel.github.io/