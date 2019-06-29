---
title: JavaScript 浅拷贝 深拷贝 赋值 引用 JS
date: 2019-06-19 10:10:59
tags: [深拷贝, 浅拷贝]
category: [JavaScript]
---

## 前言
这个问题说严重也不严重，说不小也不小。
如果你也刚刚好碰到了这个问题。就跟着我一起了解一下吧！

## 基本类型和引用类型

 

 - **基本类型**

基本类型也称值类型，数值类型。
包括了

  1. String
 2. Number
 3. Boolean
 4. Null
 5. Undefined
 6. Symbol（ES6新增，表示独一无二的一个值。[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol)）


![在这里插入图片描述](http://i1.fuimg.com/691643/32ba8273642f8421.png)

基本类型的是放在栈区的，访问的时候也是按值访问，就是正常的理解的赋值。

 - **引用类型和浅拷贝**


引用类型，顾名思义就是引用来访问的。

![在这里插入图片描述](http://i1.fuimg.com/691643/c452b97e3865e1d5.png)

在这个例子中，我们操作的是**q**。为什么**w**也改变了呢？

因为**w**不是真的给他赋值了[1, 2]。

而是赋值给了**q**所指向的地址的指针。

有点绕，其实说白了就是 q 是指向地址 A 的。

w = q ，只是把 q 指向 地址 A 的这个指针赋值给了 w 。

w 和 q 都可以操作这个地址的内容。

所以，当我们用 q 来操作这个地址内容的时候，地址内容就变了。当 w 去取值的时候，内容已经变了。


![在这里插入图片描述](http://i1.fuimg.com/691643/4be7463e640451b5.png)


w 和 q 都可以操作这个地址内容。

那么引用类型有那些呢？



![在这里插入图片描述](http://i2.tiimg.com/691643/be17d0efc32741c3.png)

这样简单的引用赋值也叫做浅拷贝。

可能会带来一些问题，比如:

A和B都是数组；

我只是懒得给B赋值和A一模一样的；

所以我直接B = A；

我在操作完B之后我又想去取值A的值的时候，发现A已经变了。

在react中也有对state浅拷贝判断的（PureComponent）


 - **深拷贝**
 
 ![在这里插入图片描述](http://i2.tiimg.com/691643/7eada1d9747ba440.png)

这个就叫做深拷贝。

将他的内容分解出来进行解析。

但是这样会导致一个问题。

麻烦！

比如

let A = [ {a: [1, 2]}, {b: [1, 2]} ]。

这个时候要深拷贝给B的话。

要一直结构到很深处。

有几个个办法可以解决。

1.叫做**序列化**

 - JSON.stringify()

 - JSON.parse()

 但是这样非常浪费资源

2.Object.assign()拷贝

 - 当对象中只有一级属性，没有二级属性的时候，此方法为深拷贝，但是对象中有对象的时候，此方法，在二级属性以后就是浅拷贝。

 3.使用递归的方式实现深拷贝

```
 //使用递归的方式实现数组、对象的深拷贝
function deepClone1(obj) {
  //判断拷贝的要进行深拷贝的是数组还是对象，是数组的话进行数组拷贝，对象的话进行对象拷贝
  var objClone = Array.isArray(obj) ? [] : {};
  //进行深拷贝的不能为空，并且是对象或者是
  if (obj && typeof obj === "object") {
    for (key in obj) {
      if (obj.hasOwnProperty(key)) {
        if (obj[key] && typeof obj[key] === "object") {
          objClone[key] = deepClone1(obj[key]);
        } else {
          objClone[key] = obj[key];
        }
      }
    }
  }
  return objClone;
}
```

5.lodash函数库实现深拷贝

lodash很热门的函数库，提供了 lodash.cloneDeep()实现深拷贝
