---
title: 'chunk 0 [mini-css-extract-plugin] 解决 CSS Modules 警告'
date: 2019-06-29 10:38:06
tags: [CSS-Module]
category: [CSS]
---

## 前言
前两周，用公司的CI部署的时候，发现![](D:\DerrickGit\01Blog\img\chunk 0 [mini-css-extract-plugin] 解决 CSS Modules 警告\aHR0cDovL2kyLnRpaW1nLmNvbS82OTE2NDMvMmVmODNhMzcxNjA4OWEwZS5wbmc.jpeg)
很鲜明的ERR！
其实这个这个只是一个warning。但是由于环境变量（process.env.ci === true）。所以这个warning被转化成了error，导致了编译失败。一般的CI服务器会自动将这个这设置为true。

## 解决方案
因为知道了原因，所以解决方案有以下两个

 - 修复这个warning。
 - 让运维小哥哥帮忙把这个环境变量设置为false。

本着探寻的心。我开始了google，baidu。
我查了整整两天。

其实这些东西可能使用webpack的一些配置就改完了。但是由于我使用的是`react-app-rewired`。
[ant design mobile](https://mobile.ant.design/docs/react/use-with-create-react-app-cn)
具体想了解的可以自己去了解。我们言归正传。

## 探索
我开始找各种“人”的毛病
`less，sass，less-loader，node-sass，postcss-px2rem` 等等。我几乎吧package.json里面的东西都试了一遍，不管是最新的，或者是我查询的过程中有提到了，我升级或者降级为固定版本。
无果！

![一个issue](http://i1.fuimg.com/691643/0c8d8b66e0cfe647.png)

他的大致意思是，CSS更加关注加载顺序，OK
我把我的项目所有的CSS加载顺序改成了一致（其实根本没有加载两个以上CSS的地方，我只是把他们都放到最后一行import）


还是翻车了。但是给了我很大的启发。

我开始关注原理。看报错的内容。

几乎都是和Ant Design Mobil相关的。

我想到了我几乎所有的页面都是几乎一个我自己写的组件`<PageContainer>`

![PageContainer](http://i1.fuimg.com/691643/eb0c0f8564148fc0.png)

在这里我用到了Ant Design Mobile的组件。
我尝试把这里的组件全部删除。然后在本地build。

竟然成功了！

很明显，问题出在了我这个共用组件头上。我开始研究他。

最后我发现，只要在我的这个组件里面的组件直接用到了Ant Design Mobile的组件就会报出CSS Module的警告。

在Ant Design 的issues里面我发现，也有人有这个问题，但是也有人说明了原因。

我个人的理解是这样的：因为Ant Design 内部也做了CSS的按需加载。导致我们引用组件的时候，很容易造成CSS引入顺序的不一致。

然后我想出了解决方案，将我所有的组件，或者说子组件，全部封装到孙子组件中。就可以解决了![子组件](http://i1.fuimg.com/691643/8563fc8bae55827c.png)

最后，预约的看到了

![编译成功](http://i1.fuimg.com/691643/a912b5c8e597a63c.png)