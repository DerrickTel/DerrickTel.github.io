---
title: window.history.back(); 缓存 返回上级页面不刷新数据
date: 2019-06-19 11:12:34
tags: [HTML, JavaScript]
category: [Web前端]
---

## 问题

 - 我们经常会做完一个操作之后返回上一个页面（比如新增完一条记录）

 - 然后我们希望返回上一个页面的时候就自动刷新他。
 - 但是由于JS的缓存机制
 - 导致我们的数据还是从前一次里面取的（他的初衷是希望你更快，更省资源）
 - 但是和我们的需求不同
 - 如图：
 
![在这里插入图片描述](http://i2.tiimg.com/691643/c1123102f826e726.png)

## 探索
从网上找的一些资料：

 - 在`window.history.back();` 后面加`location.reload();`
 - `window.history.go(-1);window.location.reload()`

均无终而返

## 解决

 - `window.location.replace(document.referrer)`
 - `window.location.href=“上一个页面URL”`（下下策）
