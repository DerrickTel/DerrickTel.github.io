---
title: 【重识前端】什么是BFC、IFC、GFC 和 FFC
date: 2020-08-12 18:03:34
tags: [CSS]
category: [重拾前端]
cover: /image/cover/web.jpeg
---

## 前言

其实在面试的过程中很有可能会被问到“介绍一下BFC”

那今天就一口气把BFC、IFC、GFC 和 FFC这四兄弟给吃透

吃透透

重点介绍一下BFC，他的三兄弟这里只做介绍，感兴趣的可以上网多查查资料然后回来教教我😂

## 开始

### what

其实在说BFC和他的三兄弟之前，我们先了解一下，什么是FC？

打断一下。说FC之前我希望先说一下BOX

#### BOX

一个页面是由很多元素组成的，而这些元素都是一个接一个盒子。那有人就说了，那`border-radius: 50%`捏？

Ok, 直接看下图吧：

![demo1-1](/image/BFC/1.png)

再来看一个更直观的图。打开Google的首页，然后在控制台输入

```js
[].forEach.call(document.querySelectorAll('*'), function(a){a.style.outline = "1px solid red";})
```

![demo2](/image/BFC/2.png)

所以，其实页面都是由一个又一个的盒子组合而成的。

明确了这个，我们再来说说FC是个什么东西。

#### FC（Formatting Context）

它是W3C CSS2.1规范中的一个概念，定义的是页面中的一块渲染区域，并且有一套渲染规则，它**决定了其子元素将如何定位**，以及**和其他元素的关系和相互作用**。

常见的`Formatting Context` 有：`Block Formatting Context`（BFC | 块级格式化上下文） 和 `Inline Formatting Context`（IFC |行内格式化上下文）。

#### BFC

> MDN: **块格式化上下文（Block Formatting Context，BFC）** 是Web页面的可视CSS渲染的一部分，是块盒子的布局过程发生的区域，也是浮动元素与其他元素交互的区域。



### why

那么FC四兄弟他们的作用是什么？可不可以不要他们？

其实，我们换个角度想想就会舒服很多。比如我们开车上路，如果没有🚦，那么路况可能会变得很糟，不同的路口的🚦的红灯绿灯时间都是经过精心设计的，比如车多的方向，红灯时间就少一些，绿灯多一些。大家都是为了让路况变得更好。

反观FC四兄弟，他们也是为了让不同情况下的CSS可以更有秩序的显示。



### how

那么我要怎么样才可以制造一个BFC？

- 根元素（`<html>）`
- 浮动元素（元素的 [`float`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/float) 不是 `none`）
- 绝对定位元素（元素的 [`position`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/position) 为 `absolute` 或 `fixed`）
- 行内块元素（元素的 [`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 为 `inline-block`）
- 表格单元格（元素的 [`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 为 `table-cell`，HTML表格单元格默认为该值）
- 表格标题（元素的 [`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 为 `table-caption`，HTML表格标题默认为该值）
- 匿名表格单元格元素（元素的 [`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 为 `table、``table-row`、 `table-row-group、``table-header-group、``table-footer-group`（分别是HTML table、row、tbody、thead、tfoot 的默认属性）或 `inline-table`）
- [`overflow`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/overflow) 值不为 `visible` 的块元素
- [`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 值为 `flow-root` 的元素
- [`contain`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/contain) 值为 `layout`、`content `或 paint 的元素
- 弹性元素（[`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 为 `flex` 或 `inline-flex `元素的直接子元素）
- 网格元素（[`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 为 `grid` 或 `inline-grid` 元素的直接子元素）
- 多列容器（元素的 [`column-count`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/column-count) 或 [`column-width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/column-width) 不为 `auto，包括 ``column-count` 为 `1`）
- `column-span` 为 `all` 的元素始终会创建一个新的BFC，即使该元素没有包裹在一个多列容器中（[标准变更](https://github.com/w3c/csswg-drafts/commit/a8634b96900279916bd6c505fda88dda71d8ec51)，[Chrome bug](https://bugs.chromium.org/p/chromium/issues/detail?id=709362)）。



### BFC可以做些什么？

#### 让浮动内容和周围的内容等高



🌰：

```html
<!DOCTYPE html>
<html>
<head>
<style>
.box {
    background-color: rgb(224, 206, 247);
    border: 5px solid rebeccapurple;
}

.float {
    float: left;
    width: 200px;
    height: 150px;
    background-color: white;
    border:1px solid black;
    padding: 10px;
}      
</style>
</head>
<body>

<div class="box">
    <div class="float">I am a floated box!</div>
    <p>I am content inside the container.</p>
</div>

<p>This is some text outside the div element.</p>

</body>
</html>

```

可以看到，那个float把底下的div盖住了。

我们随便弄一个BFC的特性中的一条：[`overflow`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/overflow) 值不为 `visible` 的块元素。

```html
<!DOCTYPE html>
<html>
<head>
<style>
.box {
    background-color: rgb(224, 206, 247);
    border: 5px solid rebeccapurple;
    overflow: auto;
}

.float {
    float: left;
    width: 200px;
    height: 150px;
    background-color: white;
    border:1px solid black;
    padding: 10px;
}          
</style>
</head>
<body>

<div class="box">
    <div class="float">I am a floated box!</div>
    <p>I am content inside the container.</p>
</div>

<p>This is some text outside the div element.</p>

</body>
</html>

```

那就设置成`auto`吧~

有没有很神奇的发现！！！

再告诉大家一个黑科技：表格标题（元素的 [`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 为 `table-caption`，HTML表格标题默认为该值）。这个是没有副作用的哦。

```html
<!DOCTYPE html>
<html>
<head>
<style>
.box {
    background-color: rgb(224, 206, 247);
    border: 5px solid rebeccapurple;
    display: table-caption;
}

.float {
    float: left;
    width: 200px;
    height: 150px;
    background-color: white;
    border:1px solid black;
    padding: 10px;
}          
</style>
</head>
<body>

<div class="box">
    <div class="float">I am a floated box!</div>
    <p>I am content inside the container.</p>
</div>

<p>This is some text outside the div element.</p>

</body>
</html>

```

给 `<div>` `display: flow-root` 属性后，`<div>` 中的所有内容都会参与 BFC，浮动的内容不会从底部溢出。

关于值 `flow-root `的这个名字，当你明白你实际上是在创建一个行为类似于根元素 （浏览器中的`<html>`元素） 的东西时，就能发现这个名字的意义了——即创建一个上下文，里面将进行 flow layout。

#### 外边距塌陷

```html
<!DOCTYPE html>
<html>
<head>
<style>
.blue{
  height: 50px;
  margin: 50px 0;
} 
.red-inner {
  height: 50px;
  margin: 50px 0;
}

.blue {
  background: blue;
}

.red-outer {
  overflow: hidden;
  background: red;
}        
</style>
</head>
<body>
<div class="blue"></div>
<div class="red-outer">
  <div class="red-inner">red inner</div>
</div>
</body>
</html>

```

意外的发现，我们两个margin都是50px，理论上应该是100px啊，可是看起来怎么只有50px的样子。。。

首先这不是 CSS 的 bug，我们可以理解为一种规范，**如果想要避免外边距的重叠，可以将其放在不同的 BFC 容器中。**

```html
<!DOCTYPE html>
<html>
<head>
<style>
.container {
    overflow: hidden;
}
p {
    width: 100px;
    height: 100px;
    background: lightblue;
    margin: 100px;
}       
</style>
</head>
<body>
<div class="container">
    <p></p>
</div>
<div class="container">
    <p></p>
</div>

</body>
</html>


```

这时候，两个盒子边距就变成了 200px

### BFC的约束条件

- 内部的Box会在垂直方向上一个接一个的放置
- 垂直方向上的距离由margin决定。（完整的说法是：属于同一个BFC的两个相邻Box的margin会发生重叠（塌陷），与方向无关。）
- 每个元素的左外边距与包含块的左边界相接触（从左向右），即使浮动元素也是如此。（这说明BFC中子元素不会超出他的包含块，而position为absolute的元素可以超出他的包含块边界）
- BFC的区域不会与float的元素区域重叠
- 计算BFC的高度时，浮动子元素也参与计算
- BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面元素，反之亦然



## 补充

### **IFC**

> IFC(Inline Formatting Contexts)直译为"内联格式化上下文"，IFC的line box（线框）高度由其包含行内元素中最高的实际高度计算而来（不受到竖直方向的padding/margin影响)
> IFC中的line box一般左右都贴紧整个IFC，但是会因为float元素而扰乱。float元素会位于IFC与与line box之间，使得line box宽度缩短。 同个ifc下的多个line box高度会不同。 IFC中时不可能有块级元素的，当插入块级元素时（如p中插入div）会产生两个匿名块与div分隔开，即产生两个IFC，每个IFC对外表现为块级元素，与div垂直排列。
> 那么IFC一般有什么用呢？
> 水平居中：当一个块要在环境中水平居中时，设置其为inline-block则会在外层产生IFC，通过text-align则可以使其水平居中。
> 垂直居中：创建一个IFC，用其中一个元素撑开父元素的高度，然后设置其vertical-align:middle，其他行内元素则可以在此父元素下垂直居中。

###  GFC

> **GFC**
> GFC(GridLayout Formatting Contexts)直译为"网格布局格式化上下文"，当为一个元素设置display值为grid的时候，此元素将会获得一个独立的渲染区域，我们可以通过在网格容器（grid container）上定义网格定义行（grid definition rows）和网格定义列（grid definition columns）属性各在网格项目（grid item）上定义网格行（grid row）和网格列（grid columns）为每一个网格项目（grid item）定义位置和空间。 
> 那么GFC有什么用呢，和table又有什么区别呢？首先同样是一个二维的表格，但GridLayout会有更加丰富的属性来控制行列，控制对齐以及更为精细的渲染语义和控制。

### FFC

> **FFC**
> FFC(Flex Formatting Contexts)直译为"自适应格式化上下文"，display值为flex或者inline-flex的元素将会生成自适应容器（flex container），可惜这个牛逼的属性只有谷歌和火狐支持，不过在移动端也足够了，至少safari和chrome还是OK的，毕竟这俩在移动端才是王道。
> Flex Box 由伸缩容器和伸缩项目组成。通过设置元素的 display 属性为 flex 或 inline-flex 可以得到一个伸缩容器。设置为 flex 的容器被渲染为一个块级元素，而设置为 inline-flex 的容器则渲染为一个行内元素。
> 伸缩容器中的每一个子元素都是一个伸缩项目。伸缩项目可以是任意数量的。伸缩容器外和伸缩项目内的一切元素都不受影响。简单地说，Flexbox 定义了伸缩容器内伸缩项目该如何布局。



