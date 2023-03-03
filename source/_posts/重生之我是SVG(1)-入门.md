---
title: 重生之我是SVG(1)-入门
date: 2023-02-27 10:39:56
tags: [svg]
category: [重拾前端]
cover: /image/cover/svg.png
---



## 概述

引用一句来自[MDN](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element)的一句话：

> SVG 图像是使用各种元素创建的，这些元素分别应用于矢量图像的结构、绘制与布局。在这里，您可以找到每个 SVG 元素的参考文档。



SVG 文件可以直接插入网页，成为 DOM 的一部分，然后用 JavaScript 和 CSS 进行操作。

```html
<!DOCTYPE html>
<html>
<head></head>
<body>
<svg
  id="mysvg"
  xmlns="http://www.w3.org/2000/svg"
  viewBox="0 0 800 600"
  preserveAspectRatio="xMidYMid meet"
>
  <circle id="mycircle" cx="400" cy="300" r="50" />
<svg>
</body>
</html>
```



SVG 代码也可以写在一个独立文件中，然后用`<img>`、`<object>`、`<embed>`、`<iframe>`等标签插入网页。

> ```html
> <img src="circle.svg">
> <object id="object" data="circle.svg" type="image/svg+xml"></object>
> <embed id="embed" src="icon.svg" type="image/svg+xml">
> <iframe id="iframe" src="icon.svg"></iframe>
> ```



CSS 也可以使用 SVG 文件。

> ```css
> .logo {
>   background: url(icon.svg);
> }
> ```



SVG 文件还可以转为 BASE64 编码，然后作为 Data URI 写入网页。

> ```html
> <img src="data:image/svg+xml;base64,[data]">
> ```



## 基本使用

更全面的标签集在[MDN](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element)中会有展示。这里展示一个完整图片会用的基本组成标签。

### `<svg>`标签

SVG 代码都放在顶层标签`<svg>`之中。下面是一个例子。

> ```html
> <svg width="100%" height="100%">
>   <circle id="mycircle" cx="50" cy="50" r="50" />
> </svg>
> ```



`<svg>`的`width`属性和`height`属性，指定了 SVG 图像在 HTML 元素中所占据的宽度和高度。除了相对单位，也可以采用绝对单位（单位：像素）。如果不指定这两个属性，SVG 图像默认大小是300像素（宽） x 150像素（高）。



如果只想展示 SVG 图像的一部分，就要指定`viewBox`属性。

> ```html
> <svg width="100" height="100" viewBox="50 50 50 50">
>   <circle id="mycircle" cx="50" cy="50" r="50" />
> </svg>
> ```

`<viewBox>`属性的值有四个数字，分别是左上角的横坐标和纵坐标、视口的宽度和高度。上面代码中，SVG 图像是100像素宽 x 100像素高，`viewBox`属性指定视口从`(50, 50)`这个点开始。所以，实际看到的是右下角的四分之一圆。

注意，视口必须适配所在的空间。上面代码中，视口的大小是 50 x 50，由于 SVG 图像的大小是 100 x 100，所以视口会放大去适配 SVG 图像的大小，即放大了四倍。

如果不指定`width`属性和`height`属性，只指定`viewBox`属性，则相当于只给定 SVG 图像的长宽比。这时，SVG 图像的默认大小将等于所在的 HTML 元素的大小。



### 图形标签

```html
<svg width="400" height="250" version="1.1" xmlns="http://www.w3.org/2000/svg">

  <rect x="10" y="10" width="30" height="30" stroke="black" fill="transparent" stroke-width="5"/>
  <rect x="60" y="10" rx="10" ry="10" width="30" height="30" stroke="black" fill="transparent" stroke-width="5"/>

  <circle cx="25" cy="75" r="20" stroke="red" fill="transparent" stroke-width="5"/>
  <ellipse cx="75" cy="75" rx="20" ry="5" stroke="red" fill="transparent" stroke-width="5"/>

  <line x1="10" x2="50" y1="110" y2="150" stroke="orange" fill="transparent" stroke-width="5"/>
  <polyline points="60 110 65 120 70 115 75 130 80 125 85 140 90 135 95 150 100 145"
      stroke="orange" fill="transparent" stroke-width="5"/>

  <polygon points="50 160 55 180 70 180 60 190 65 205 50 195 35 205 40 190 30 180 45 180"
      stroke="green" fill="transparent" stroke-width="5"/>

  <path d="M20,230 Q40,205 50,230 T90,230" fill="none" stroke="blue" stroke-width="5"/>
</svg>
```

![1](/image/svg1/basic.png)

一些名牛的官网动画基本上是由这些东西组成的。更多的是创意以及一些更高级的变形。后面会逐一介绍，主要我觉得是要扎好马步！

#### 矩形 `<rect>`

矩形基本上由6个元素确定整体样子。分别是

- x -> 矩形左上角`x`位置, 默认值为 0
- y -> 矩形左上角`y`位置, 默认值为 0
- width -> 矩形的宽度, 不能为负值否则报错, 0 值不绘制
- height -> 矩形的高度,  不能为负值否则报错, 0 值不绘制
- rx -> 圆角`x`方向半径, 不能为负值否则报错
- ry -> 圆角`y`方向半径, 不能为负值否则报错



举个小的例子来解释一下`rx`与`ry`。

```html
<rect x="10" y="10" width="30" height="30" stroke="black" fill="transparent" stroke-width="5"/>
<rect x="60" y="10" rx="10" ry="10" width="30" height="30" stroke="black" fill="transparent" stroke-width="5"/>
```



![1](/image/svg1/rxry.png)



#### 圆形`<circle>`

正如你猜到的，`circle`元素会在屏幕上绘制一个圆形。它只有 3 个属性用来设置圆形。

```html
<circle cx="25" cy="75" r="20" stroke="red" fill="transparent" stroke-width="5"/>
```

- r -> 圆的半径
- cx -> 圆心的 `x ` 位置
- cy -> 圆心的 `y` 位置



#### 椭圆`<ellipse>`

`ellipse`是`circle`元素更通用的形式，你可以分别缩放圆的 x 半径和 y 半径（通常数学家称之为长轴半径和短轴半径）。

```html
<ellipse cx="75" cy="75" rx="20" ry="5" stroke="red" fill="transparent" stroke-width="5"/>
```

- ry -> 椭圆的 `y` 半径
- rx ->椭圆的 `x` 半径
- cx -> 圆心的 `x ` 位置
- cy -> 圆心的 `y` 位置



#### 线条`<line>`

绘制直线。它取两个点的位置作为属性，指定这条线的起点和终点位置。

```html
<line x1="10" x2="50" y1="110" y2="150" stroke="orange" fill="transparent" stroke-width="5"/>
```

- x1 -> 起点的 `x` 位置
- y1 -> 起点的 `y` 位置
- x2 -> 终点的 `x` 位置
- y2 -> 终点的 `y` 位置



#### 折线`<polyline>`

是一组连接在一起的直线。因为它可以有很多的点，折线的的所有点位置都放在一个 points 属性中：

```html
<polyline points="60 110 65 120 70 115 75 130 80 125 85 140 90 135 95 150 100 145"
        stroke="orange" fill="transparent" stroke-width="5"/>
```

- points -> 点集数列。每个数字用空白、逗号、终止命令符或者换行符分隔开。每个点必须包含 2 个数字，一个是 x 坐标，一个是 y 坐标。所以点列表 (0,0), (1,1) 和 (2,2) 可以写成这样：“0 0, 1 1, 2 2”。



#### 多边形`<polygon>`

`polygon`和折线很像，它们都是由连接一组点集的直线构成。不同的是，`polygon`的路径在最后一个点处自动回到第一个点。需要注意的是，矩形也是一种多边形，如果需要更多灵活性的话，你也可以用多边形创建一个矩形。

```html
<polygon points="50 160 55 180 70 180 60 190 65 205 50 195 35 205 40 190 30 180 45 180"
        stroke="green" fill="transparent" stroke-width="5"/>
```

- points -> 点集数列。每个数字用空白符、逗号、终止命令或者换行符分隔开。每个点必须包含 2 个数字，一个是 x 坐标，一个是 y 坐标。所以点列表 (0,0), (1,1) 和 (2,2) 可以写成这样：“0 0, 1 1, 2 2”。路径绘制完后闭合图形，所以最终的直线将从位置 (2,2) 连接到位置 (0,0)。



#### 路径`<path>`

`path`可能是 SVG 中最常见的形状。你可以用 path 元素绘制矩形（直角矩形或者圆角矩形）、圆形、椭圆、折线形、多边形，以及一些其他的形状，例如贝塞尔曲线、2 次曲线等曲线。后面会有一章专门介绍这个。

```html
<path d="M20,230 Q40,205 50,230 T90,230" fill="none" stroke="blue" stroke-width="5"/>
```

- d -> 一个点集数列以及其它关于如何绘制路径的信息。具体字母的含义在后面会介绍，因为太多啦~

上面的demo全都在一个[demo的github](https://github.com/DerrickTel/study-demo)里面。
