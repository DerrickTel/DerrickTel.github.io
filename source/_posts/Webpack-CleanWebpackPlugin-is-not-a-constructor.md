---
title: Webpack CleanWebpackPlugin is not a constructor
date: 2019-06-22 15:55:56
tags: [WebPack, clean-webpack-plugin]
category: [WebPack]
---

## 前言
今天自己跟着webpack官网的demo一步步走下来。发现了这个问题。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190622155255789.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2N1YW5kZXFpbjIwODM=,size_16,color_FFFFFF,t_70)

查了一圈。发现了这个博主。nice！
```
// webpack版本：4.34.0
 
// 抛错原写法
const CleanWebpackPlugin = require("clean-webpack-plugin");
 
...
 
plugins: [
    new CleanWebpackPlugin(['dist'])
]
 
...
 
// 另一种错误写法
 
const CleanWebpackPlugin = require("clean-webpack-plugin");
 
...
 
plugins: [
    new CleanWebpackPlugin(['dist'], {
        root: path.resolve(__dirname, '../'),   //根目录
    })
]
 
...
 
// =============================分割线==============================
 
// 正确写法
 
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
 
...
 
plugins: [
    new CleanWebpackPlugin()
]
 
```

 现在的版本不用指定文件路径了，直接调用new CleanWebpackPlugin()
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190622155123842.png)
索引：https://blog.csdn.net/qq_36242361/article/details/90709258

