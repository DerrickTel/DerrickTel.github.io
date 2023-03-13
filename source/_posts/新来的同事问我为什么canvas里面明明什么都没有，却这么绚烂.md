---
title: 前端妹子问我为什么canvas里面明明什么都没有，却这么绚烂
date: 2023-03-07 10:22:37
tags: [svg]
category: [重拾前端]
cover: /image/cover/canvas.jpeg
---

前端妹子问我为什么官网上面只有一个canvas标签，里面什么都没有....

我脸色一变”完了！官网挂了！“。走过去一看，原来是canvas的动画啊。

”这个你想学啊~我教你啊，到时候你可别说听不懂“。

## 基础知识

使用 `<canvas>` 元素不是非常难，但你需要一些基本的[HTML](https://developer.mozilla.org/zh-CN/docs/Web/HTML)和[JavaScript](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript)知识。除一些过时的浏览器不支持`<canvas>` 元素外，所有的新版本主流浏览器都支持它。Canvas 的默认大小为 300 像素 ×150 像素（宽 × 高，像素的单位是 px）。但是，可以使用 HTML 的高度和宽度属性来自定义 Canvas 的尺寸。为了在 Canvas 上绘制图形，我们使用一个 JavaScript 上下文对象，它能动态创建图像（creates graphics on the fly）。



## 基本使用

`<canvas>` 标签只有两个属性**——** [`width`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/canvas#attr-width)和[`height`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/canvas#attr-height)。这些都是可选的，并且同样利用 [DOM](https://developer.mozilla.org/zh-CN/docs/Glossary/DOM) [properties](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLCanvasElement) 来设置。当没有设置宽度和高度的时候，canvas 会初始化宽度为 300 像素和高度为 150 像素（这个和`svg`一样）。该元素可以使用[CSS](https://developer.mozilla.org/zh-CN/docs/Glossary/CSS)来定义大小，但在绘制时图像会伸缩以适应它的框架尺寸：如果 CSS 的尺寸与初始画布的比例不一致，它会出现扭曲。

> 如果你绘制出来的图像是扭曲的，尝试用 width 和 height 属性为<canvas>明确规定宽高，而不是使用 CSS。

[`id`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Global_attributes/id)属性并不是<canvas>元素所特有的，而是每一个 HTML 元素都默认具有的属性（比如 class 属性）。给每个标签都加上一个 id 属性是个好主意，因为这样你就能在我们的脚本中很容易的找到它。



### 替代内容

在一些不支持canvas的浏览器或者某些特定版本下，需要加入替代内容。

这非常简单：我们只是在<canvas>标签中提供了替换内容。不支持<canvas>的浏览器将会忽略容器并在其中渲染后备内容。而支持<canvas>的浏览器将会忽略在容器中包含的内容，并且只是正常渲染 canvas。

```html
<canvas id="canvas" width="150" height="150">
  由于当然浏览器不支持canvas，就只显示我啦~
</canvas>
```



### 上下文Context

`canvas`元素创造了一个固定大小的画布，它公开了一个或多个**渲染上下文**，其可以用来绘制和处理要展示的内容。后面会着重介绍2D相关内容。

canvas 起初是空白的。为了展示，首先脚本需要找到渲染上下文，然后在它的上面绘制。canvas元素有一个叫做 `getContext()`的方法，这个方法是用来获得渲染上下文和它的绘画功能。`getContext()`接受一个参数，即上下文的类型。

但是要注意识别某些无法使用`canvas`的情况，这样才更加鲁棒！

```javascript
const canvas = document.getElementById('canvas');
if (canvas.getContext) {
	const ctx = canvas.getContext('2d');
  // 鲁棒的代码……
} else {
  // 无法识别情况……
}
```



## 绘制

不同于 `SVG`，`canvas`只支持两种形式的图形绘制：矩形和路径（由一系列点连成的线段）。所有其他类型的图形都是通过一条或者多条路径组合而成的。不过，我们拥有众多路径生成的方法让复杂图形的绘制成为了可能。

> Tips: SVG的入门指南也有更新，在我的主页里面就可以看见。

### 绘制矩形

canvas 提供了三种方法绘制矩形：

- `fillRect(x, y, width, height)`

  绘制一个填充的矩形

- `strokeRect(x, y, width, height)`

  绘制一个矩形的边框

- `clearRect(x, y, width, height)`

  清除指定矩形区域，让清除部分完全透明。

上面提供的方法之中每一个都包含了相同的参数。x 与 y 指定了在 canvas 画布上所绘制的矩形的左上角（相对于原点）的坐标。width 和 height 设置矩形的尺寸。

纸上得来终觉浅，绝知此事要躬行！撸起袖子，开敲！

```html
<!DOCTYPE html>
<html>
 <head>
  <script type="application/javascript">
    function draw() {
      var canvas = document.getElementById("canvas");
      if (canvas.getContext) {
        var ctx = canvas.getContext("2d");

        ctx.fillStyle = "rgb(200,0,0)";
        ctx.fillRect (10, 10, 55, 50);

        ctx.fillStyle = "rgba(0, 0, 200, 0.5)";
        ctx.fillRect (30, 30, 55, 50);
      }
    }
  </script>
 </head>
 <body onload="draw();">
   <canvas id="canvas" width="150" height="150"></canvas>
 </body>
</html>
```

![1](/image/canvas/1.png)

细心的人已经发现矩形很模糊，而且没发现的还沉浸在自己成功的喜悦之中。

### 高清的Canvas

那为什么会模糊？

主要是现在都是高清屏，就是分辨率上来了，画图的时候不是这样画的，只是等比例放大，所以模糊。

那么怎么办？把画布也同步放大！

加入这样的一个函数，传入canvas的上下文进去，让他来改造。

```javascript
function hdCanvas(canvas, ratio) {
  const rect = canvas.getBoundingClientRect();
  canvas.width = rect.width * ratio;
  canvas.height = rect.height * ratio;
  canvas.style.width = rect.width + "px"
  canvas.style.height = rect.height + "px"
}
```

修改之后就高清多啦~

[矩形源码](https://github.com/DerrickTel/study-demo/blob/main/canvas)

### 绘制路径

绘制路径大致有下面4个步骤

1. 首先，你需要创建路径起始点。
2. 然后你使用画图命令去画出路径。
3. 之后你把路径封闭。
4. 一旦路径生成，你就能通过描边或填充路径区域来渲染图形。

以下是所要用到的函数：

- `beginPath()`

  新建一条路径，生成之后，图形绘制命令被指向到路径上生成路径。

- `closePath()`

  闭合路径之后图形绘制命令又重新指向到上下文中。

- `stroke()`

  通过线条来绘制图形轮廓。

- `fill()`

  通过填充路径的内容区域生成实心的图形。

### 画一个三角形

三角形就是3条简单的直线。

```javascript
function draw() {
      const canvas = document.getElementById("canvas");
      if (canvas.getContext) {
        const ctx = canvas.getContext("2d");
        const ratio = window.devicePixelRatio || 1
        hdCanvas(canvas, ratio)

        // 放大倍数
        ctx.scale(ratio, ratio);

        ctx.lineWidth = 10

        // 空心 - 没有 closePath
        ctx.beginPath();
        ctx.moveTo(75, 50);
        ctx.lineTo(100, 75);
        ctx.lineTo(100, 25);
        ctx.lineTo(75, 50);
        // ctx.closePath();
        ctx.stroke();

        // 空心 - closePath
        ctx.beginPath();
        ctx.moveTo(275, 50);
        ctx.lineTo(300, 75);
        ctx.lineTo(300, 25);
        // ctx.lineTo(275, 50);
        // 有 closePath会自动首尾相连
        ctx.closePath();
        ctx.stroke();

        // 实心
        ctx.beginPath();
        ctx.moveTo(175, 50);
        ctx.lineTo(200, 75);
        ctx.lineTo(200, 25);
        ctx.closePath();
        ctx.fill();
      }
    }
```

![1](/image/canvas/2.png)

可以很明显的看到，没有`closePath`他是没有闭合的。有`closePath`哪怕没有回到起点，他也会自动的首尾相连。

上述的四个函数都可以在这个demo中学习到。

### 画一个圆弧

绘制圆弧或者圆，我们使用`arc()`方法。当然可以使用`arcTo()`，不过这个不太好理解，也不常用就不介绍了，着重介绍`arc()`。

`arc(x, y, radius, startAngle, endAngle, anticlockwise)`

画一个以（x,y）为圆心的以 radius 为半径的圆弧（圆），从 startAngle 开始到 endAngle 结束，按照 anticlockwise 给定的方向（默认为顺时针）来生成。

- x -> 圆弧中心（圆心）的 x 轴坐标。
- y -> 圆弧中心（圆心）的 y 轴坐标。
- radius -> 圆弧的半径。
- startAngle -> 圆弧的起始点，x 轴方向开始计算，单位以弧度表示。
- endAngle -> 圆弧的终点，单位以弧度表示。
- anticlockwise -> 默认为 `false`可选的[`Boolean`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)值，如果为 `true`，逆时针绘制圆弧，反之，顺时针绘制。

> `arc()` 函数中表示角的单位是弧度，不是角度。角度与弧度的 js 表达式：
>
> 弧度=(Math.PI/180)\*角度。

具体的起点，终点坐标可以看下图

![1](/image/canvas/3.gif)

具体看一个demo

```JavaScript
function draw() {
  const canvas = document.getElementById("canvas");
  if (canvas.getContext) {
    const ctx = canvas.getContext("2d");
    const ratio = window.devicePixelRatio || 1
    hdCanvas(canvas, ratio)

    // 放大倍数
    ctx.scale(ratio, ratio);

    ctx.beginPath();
    ctx.arc(100, 50, 20, 0.5 * Math.PI, 2 * Math.PI);
    ctx.stroke();

    /**
         * 不加这句话,上个圆的终点会与当前这个圆的期待你相连, 不信可以试试
         * 也可以用 moveTo 到圆的起点, 这里偷懒了, 不太规范
         */
    ctx.beginPath();
    ctx.arc(200, 50, 20, 0.5 * Math.PI, 4 * Math.PI);
    ctx.stroke();
  }
}
```

![1](/image/canvas/4.png)

我们可以发现，起点与终点之间的值超过了2就会完整的一个圆出现。

为什么是2？

九漏鱼？？？？？？？？？？

圆的周长等于什么？ 2 * π * R



#### 自我学习

马上画一个笑脸。

![1](/image/canvas/5.png)

```javascript
ctx.beginPath();
ctx.arc(75, 75, 50, 0, Math.PI * 2, true); // 绘制
ctx.moveTo(110, 75);
ctx.arc(75, 75, 35, 0, Math.PI, false);   // 口 (顺时针)
ctx.moveTo(65, 65);
ctx.arc(60, 65, 5, 0, Math.PI * 2, true);  // 左眼
ctx.moveTo(95, 65);
ctx.arc(90, 65, 5, 0, Math.PI * 2, true);  // 右眼
ctx.stroke();
```

[源码](https://github.com/DerrickTel/study-demo/blob/main/canvas)

### 二次贝塞尔曲线和三次贝塞尔曲线

- `quadraticCurveTo(cp1x, cp1y, x, y)`

​		绘制二次贝塞尔曲线，`cp1x,cp1y` 为一个控制点，`x,y` 为结束点。

- `bezierCurveTo(cp1x, cp1y, cp2x, cp2y, x, y)`

​		绘制三次贝塞尔曲线，`cp1x,cp1y`为控制点一，`cp2x,cp2y`为控制点二，`x,y`为结束点。

这里就不展开了。这里面很复杂，课后学习！！！



## 色彩与样式

```javascript
// 这些 fillStyle 的值均为 '橙色'
ctx.fillStyle = "orange";
ctx.fillStyle = "#FFA500";
ctx.fillStyle = "rgb(255,165,0)";
ctx.fillStyle = "rgba(255,165,0,1)";
```

### 色彩

- `fillStyle = color` -> 设置图形的填充颜色。
- `strokeStyle = color` -> 设置图形轮廓的颜色。

> color指上述的各种实现方式

### 透明度

```javascript
ctx.globalAlpha = 0.5;
```

这个属性影响到 canvas 里所有图形的透明度，有效的值范围是 0.0（完全透明）到 1.0（完全不透明），默认是 1.0。

因为 `strokeStyle` 和 `fillStyle` 属性接受符合 CSS 3 规范的颜色值，那我们可以用下面的写法来设置具有透明度的颜色。

```javascript
// 指定透明颜色，用于描边和填充样式
ctx.strokeStyle = "rgba(255,0,0,0.5)";
ctx.fillStyle = "rgba(255,0,0,0.5)";
```

### 线型

可以通过一系列属性来设置线的样式。

- [`lineWidth = value`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/lineWidth)

  设置线条宽度。

- [`lineCap = type`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/lineCap)

  设置线条末端样式。

  它可以为下面的三种的其中之一：`butt`，`round` 和 `square`。默认是 `butt`。

- [`lineJoin = type`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/lineJoin)

  设定线条与线条间接合处的样式。

  它可以是这三种之一：`round`, `bevel` 和 `miter`。默认是 `miter`。

- [`miterLimit = value`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/miterLimit)

  限制当两条线相交时交接处最大长度；所谓交接处长度（斜接长度）是指线条交接处内角顶点到外角顶点的长度。

- [`getLineDash()`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/getLineDash)

  返回一个包含当前虚线样式，长度为非负偶数的数组。

- [`setLineDash(segments)`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/setLineDash)

  设置当前虚线样式。

- [`lineDashOffset = value`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/lineDashOffset)

  设置虚线样式的起始偏移量。

通过以下的样例可能会更加容易理解。

把所有的属性都搞一遍。

![1](/image/canvas/6.png)

[源码](https://github.com/DerrickTel/study-demo/tree/main/canvas)

### 渐变

这个和CSS的一样，就不展开了

- [`createLinearGradient(x1, y1, x2, y2)`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/createLinearGradient)

  createLinearGradient 方法接受 4 个参数，表示渐变的起点 (x1,y1) 与终点 (x2,y2)。

- [`createRadialGradient(x1, y1, r1, x2, y2, r2)`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/createRadialGradient)

  createRadialGradient 方法接受 6 个参数，前三个定义一个以 (x1,y1) 为原点，半径为 r1 的圆，后三个参数则定义另一个以 (x2,y2) 为原点，半径为 r2 的圆。

```
var lineargradient = ctx.createLinearGradient(0,0,150,150);
var radialgradient = ctx.createRadialGradient(75,75,0,75,75,100);
```

Copy to Clipboard

创建出 `canvasGradient` 对象后，我们就可以用 `addColorStop` 方法给它上色了。

- [`gradient.addColorStop(position, color)`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasGradient/addColorStop)

  addColorStop 方法接受 2 个参数，`position` 参数必须是一个 0.0 与 1.0 之间的数值，表示渐变中颜色所在的相对位置。例如，0.5 表示颜色会出现在正中间。`color` 参数必须是一个有效的 CSS 颜色值（如 #FFF，rgba(0,0,0,1)，等等）。

### 图案样式

这个也和CSS的一样，也不展开了

[`createPattern(image, type)`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/createPattern)

该方法接受两个参数。Image 可以是一个 `Image` 对象的引用，或者另一个 canvas 对象。`Type` 必须是下面的字符串值之一：`repeat`，`repeat-x`，`repeat-y` 和 `no-repeat`。



### 阴影

- [`shadowOffsetX = float`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/shadowOffsetX)

  `shadowOffsetX` 和 `shadowOffsetY` 用来设定阴影在 X 和 Y 轴的延伸距离，它们是不受变换矩阵所影响的。负值表示阴影会往上或左延伸，正值则表示会往下或右延伸，它们默认都为 `0`。

- [`shadowOffsetY = float`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/shadowOffsetY)

  shadowOffsetX 和 `shadowOffsetY` 用来设定阴影在 X 和 Y 轴的延伸距离，它们是不受变换矩阵所影响的。负值表示阴影会往上或左延伸，正值则表示会往下或右延伸，它们默认都为 `0`。

- [`shadowBlur = float`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/shadowBlur)

  shadowBlur 用于设定阴影的模糊程度，其数值并不跟像素数量挂钩，也不受变换矩阵的影响，默认为 `0`。

- [`shadowColor = color`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/shadowColor)

  shadowColor 是标准的 CSS 颜色值，用于设定阴影颜色效果，默认是全透明的黑色。

### 填充规则

当我们用到 `fill`（或者 [`clip`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/clip)和[`isPointinPath`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/isPointInPath) ）你可以选择一个填充规则，该填充规则根据某处在路径的外面或者里面来决定该处是否被填充，这对于自己与自己路径相交或者路径被嵌套的时候是有用的。

```javascript
function draw() {
  var ctx = document.getElementById('canvas').getContext('2d');
  ctx.beginPath();
  ctx.arc(50, 50, 30, 0, Math.PI*2, true);
  ctx.arc(50, 50, 15, 0, Math.PI*2, true);
  ctx.fill("evenodd");
}
```

这个可以大家自己试试是什么样子的。

## 文本

文本绘制也算是蛮重要的一章，可以绘制各式各样的图案，否则的话，按照`path`去一一绘画，会耗费大量的时间

### 绘制文本

`fillText(text, x, y [, maxWidth\])`

在指定的 (x,y) 位置绘制指定的`实心`文本，绘制的最大宽度是可选的。

`strokeText(text, x, y [, maxWidth\])`

在指定的 (x,y) 位置绘制指定的`空心`文本，绘制的最大宽度是可选的。

### 有样式的文本

- `font = '10px sans-serif'`
  - 当前我们用来绘制文本的样式。这个字符串使用和 `CSS font`属性相同的语法。默认的字体是 `10px sans-serif`。
- `textAlign = 'value'`
  - 文本对齐选项。可选的值包括：`start`, `end`, `left`, `right` or `center`. 默认值是 `start`。
- `textBaseline = 'value'`
  - 基线对齐选项。可选的值包括：`top`, `hanging`, `middle`, `alphabetic`, `ideographic`, `bottom`。默认值是 `alphabetic`。
- `direction = 'value'`
  - 文本方向。可能的值包括：`ltr`, `rtl`, `inherit`。默认值是 `inherit`。

如果你之前使用过 CSS，那么这些选项你会很熟悉。



### 预测文本宽度

当你需要获得更多的文本细节时，下面的方法可以给你测量文本的方法。

- `measureText()`

  将返回一个 `TextMetrics`对象的宽度、所在像素，这些体现文本特性的属性。

下面的代码段将展示如何测量文本来获得它的宽度：

```javascript
function draw() {
  var ctx = document.getElementById('canvas').getContext('2d');
  var text = ctx.measureText("foo"); // TextMetrics object
  text.width; // 16;
}
```
