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

[`id`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Global_attributes/id)属性并不是`<canvas>`元素所特有的，而是每一个 HTML 元素都默认具有的属性（比如 class 属性）。给每个标签都加上一个 id 属性是个好主意，因为这样你就能在我们的脚本中很容易的找到它。



### 替代内容

在一些不支持canvas的浏览器或者某些特定版本下，需要加入替代内容。

这非常简单：我们只是在`<canvas>`标签中提供了替换内容。不支持`<canvas>`的浏览器将会忽略容器并在其中渲染后备内容。而支持`<canvas>`的浏览器将会忽略在容器中包含的内容，并且只是正常渲染` canvas`。

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

![1](/image/canvas/7.png)

上图[源码](https://github.com/DerrickTel/study-demo/tree/main/canvas)



## 图像

canvas中还有一项强大的功能就是他的图像处理能力，并不局限于静态图片，可以做到动态图片，甚至是游戏画面等等。这个图片的来源也不至于常见的JEPG、PNG、GIF等，甚至可以是video的某一帧，甚至是其他的canvas作为图片源。

使用图片大致可以分为两步

- 获得一个HTML对象（可以是URL，图片，视频，其他canvas）
- 使用`drawImage()`函数将获取的内容绘制在canvas上



### 图片源


1. HTMLImageElement -> `<img>`
2. HTMLVideoElement -> `<video>`
3. HTMLCanvasElement -> `<canvas>`
5. ImageBitmap -> 位图资源，这里就不展开介绍了。因为浏览器的支持不是很好。

#### HTMLImageElement

第一种就是直接拿到`<img src="" id="img" />`

第二种是自己创建一个元素，使用 `new Image()` 或 `document.createElement("img")` 方法

```javascript
const c = document.getElementById("canvas-1");
const ctx = c.getContext("2d");
const img = document.getElementById("img1");
const img2 = new Image();
img2.onload = function() {
    ctx.drawImage(this, 10, 10);
}
img2.src = "https://www.twle.cn/static/i/meimei_160x160.png";
ctx.drawImage(img, 10, 10);
```

#### HTMLVideoElement

在`drawImage`里的`video`并不是拿到什么数据流，而且拿的那一瞬间的一帧图案。

这里给一个简单的小demo后面也有源码。大家可以运行一下自己试试看。

![1](/image/canvas/8.gif)

>  在线代码: https://codesandbox.io/s/heuristic-feather-qnwdcs?file=/index.html

#### HTMLCanvasElement

这里很简单。就是传入别的`canvas`的`context`就好。直接看个例子就可以悟道了。

```javascript
const canvas1  = document.createElement("canvas1");
const ctx1 = canvas1.getContext("2d");
ctx1.fillText("canvas"）

const canvas2  = document.createElement("canvas1");
const ctx2 = canvas1.getContext("2d");
canvas2.drawImage(canvas1, 10, 10);
```

就这么简单，直接将`canvas1`作为`drawImage`的第一个参数就行



### drwaImage的变化

`drawImage` 方法的又一变种是增加了两个用于控制图像在 canvas 中缩放的参数。

- [`drawImage(image, x, y, width, height)`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/drawImage)

  这个方法多了 2 个参数：`width` 和 `height，`这两个参数用来控制 当向 canvas 画入时应该缩放的大小

`drawImage` 方法的第三个也是最后一个变种有 8 个新参数，用于控制做切片显示的。

- [`drawImage(image, sx, sy, sWidth, sHeight, dx, dy, dWidth, dHeight)`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/drawImage)

  第一个参数和其他的是相同的，都是一个图像或者另一个 canvas 的引用。其他 8 个参数最好是参照右边的图解，前 4 个是定义图像源的切片位置和大小，后 4 个则是定义切片的目标显示位置和大小。



## 状态恢复与保存

在了解变形之前，我先介绍两个在你开始绘制复杂图形时必不可少的方法。

- [`save()`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/save)

  保存画布 (canvas) 的所有状态

- [`restore()`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/restore)

  save 和 restore 方法是用来保存和恢复 canvas 状态的，都没有参数。Canvas 的状态就是当前画面应用的所有样式和变形的一个快照。

Canvas 状态存储在栈中，每当`save()`方法被调用后，当前的状态就被推送到栈中保存。一个绘画状态包括：

- 当前应用的变形（即移动，旋转和缩放，见下）

- 以及下面这些属性：[`strokeStyle`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/strokeStyle), [`fillStyle`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/fillStyle), [`globalAlpha`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/globalAlpha), [`lineWidth`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/lineWidth), [`lineCap`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/lineCap), [`lineJoin`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/lineJoin), [`miterLimit`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/miterLimit), [`lineDashOffset`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/lineDashOffset), [`shadowOffsetX`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/shadowOffsetX), [`shadowOffsetY`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/shadowOffsetY), [`shadowBlur`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/shadowBlur), [`shadowColor`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/shadowColor), [`globalCompositeOperation`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/globalCompositeOperation), [`font`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/font), [`textAlign`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/textAlign), [`textBaseline`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/textBaseline), [`direction`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/direction), [`imageSmoothingEnabled`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/imageSmoothingEnabled)

  

你可以调用任意多次 `save`方法。每一次调用 `restore` 方法，上一个保存的状态就从栈中弹出，所有设定都恢复。

我们尝试用这个连续矩形的例子来描述 canvas 的状态栈是如何工作的。

第一步是用默认设置画一个大四方形，然后保存一下状态。改变填充颜色画第二个小一点的蓝色四方形，然后再保存一下状态。再次改变填充颜色绘制更小一点的半透明的白色四方形。

到目前为止所做的动作和前面章节的都很类似。不过一旦我们调用 `restore`，状态栈中最后的状态会弹出，并恢复所有设置。如果不是之前用 `save` 保存了状态，那么我们就需要手动改变设置来回到前一个状态，这个对于两三个属性的时候还是适用的，一旦多了，我们的代码将会猛涨。

当第二次调用 `restore` 时，已经恢复到最初的状态，因此最后是再一次绘制出一个黑色的四方形。

```javascript
ctx.fillRect(0, 0, 150, 150);   // 使用默认设置绘制一个矩形
ctx.save();                  // 保存默认状态

ctx.fillStyle = '#09F'       // 在原有配置基础上对颜色做改变
ctx.fillRect(15, 15, 120, 120); // 使用新的设置绘制一个矩形

ctx.save();                  // 保存当前状态
ctx.fillStyle = '#FFF'       // 再次改变颜色配置
ctx.globalAlpha = 0.5;
ctx.fillRect(30, 30, 90, 90);   // 使用新的配置绘制一个矩形

ctx.restore();               // 重新加载之前的颜色状态
ctx.fillRect(45, 45, 60, 60);   // 使用上一次的配置绘制一个矩形

ctx.restore();               // 加载默认颜色配置
ctx.fillRect(60, 60, 30, 30);   // 使用加载的配置绘制一个矩形
```

![1](/image/canvas/9.png)

[源码](https://github.com/DerrickTel/study-demo/tree/main/canvas)

## 变形

这里面的都和css的一样，就不多介绍了。

- `translate(x, y)`

  x *是左右偏移量，*y 是上下偏移量，

- `rotate(angle)`

  这个方法只接受一个参数：旋转的角度 (angle)，它是顺时针方向的，以弧度为单位的值。

​		旋转的中心点始终是 canvas 的原点，如果要改变它，我们需要用到 `translate`方法。

- **`scale(x, y)`**

  `scale`方法可以缩放画布的水平和垂直的单位。两个参数都是实数，可以为负数，x 为水平缩放因子，y 为垂直缩放因子，如果比 1 小，会缩小图形，如果比 1 大会放大图形。默认值为 1，为实际大小。

  

  

- [`transform(a, b, c, d, e, f)`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/transform)

​		这个方法是将当前的变形矩阵乘上一个基于自身参数的矩阵

​		这个函数的参数各自代表如下：

1. `a (m11) `水平方向的缩放
2. `b(m12)`竖直方向的倾斜偏移
3. `c(m21)`水平方向的倾斜偏移
4. `d(m22)`竖直方向的缩放
5. `e(dx)`水平方向的移动
6. `f(dy)`竖直方向的移动

- [`setTransform(a, b, c, d, e, f)`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/setTransform)

  这个方法会将当前的变形矩阵重置为单位矩阵，然后用相同的参数调用 `transform`方法。如果任意一个参数是无限大，那么变形矩阵也必须被标记为无限大，否则会抛出异常。从根本上来说，该方法是取消了当前变形，然后设置为指定的变形，一步完成。

- [`resetTransform()`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/resetTransform)

  重置当前变形为单位矩阵，它和调用以下语句是一样的：`ctx.setTransform(1, 0, 0, 1, 0, 0);`





![1](/image/canvas/10.png)

[源码](https://github.com/DerrickTel/study-demo/tree/main/canvas)



## 图像混合

`globalCompositeOperation = type`

type有26种。

```JavaScript
[ 'source-over','source-in','source-out','source-atop',
 'destination-over','destination-in','destination-out','destination-atop',
 'lighter', 'copy','xor', 'multiply', 'screen', 'overlay', 'darken',
 'lighten', 'color-dodge', 'color-burn', 'hard-light', 'soft-light',
 'difference', 'exclusion', 'hue', 'saturation', 'color', 'luminosity'
]
```

![1](/image/canvas/11.png)

[源码](https://github.com/DerrickTel/study-demo/tree/main/canvas)

## 基础动画

我们知道canvas其实就是一个画布，我们在上面作画。等我们看到的时候已经作画完毕。那要怎么制作动画呢？如果需要移动它，我们不得不对所有东西（包括之前的）进行重绘。重绘是相当费时的，而且性能很依赖于电脑的速度。

我们可以制作很多帧，每一帧都是定格。已快速的速度播放，比如小时候看的定格动画。看的影视剧其实也可以拆分为60个定格图片。打的游戏FPS多少就是拆成多少帧"canvas"

那这样的话是不是就有思路啦？

有两种方法可以实现这样的动画操控。首先可以通过 `setInterval` 和 `setTimeout` 方法来控制在设定的时间点上执行重绘。



```javascript
setInterval(() => {
  ctx.arc(x, y, 50, 0, Math.PI * 2, true);
  ctx.stroke();

  if(x >= 300) {
  	x = 70
  } else {
  	x += 1
  }
}, 1000 / 60)
```

![1](/image/canvas/12.gif)

好像差点意思，不是我们想要的那个圆圈的活动啊。虽然确实动起来了！是不是我们在画新的圆之前先清除一下？

代码稍作修改

```javascript
const timer = setInterval(() => {
          ctx.clearRect(0,0,450,150)
          ctx.translate(x, y)
          ctx.arc(70, 80, 50, 0, Math.PI * 2, true);
          ctx.stroke();
          if(x >= 300) {
            x = 1
          } else {
            x += 1
          }
        }, 1000 / 10)
```

![1](/image/canvas/13.gif)

之前还没发现，怎么中间还有一条线，而且这个间距明明都是 `1`啊，为什么越来越大，而且这个并没有清除成功啊！

1. 中间的线应该是因为我们是连续作画，没有`closepath`导致的拖拽。
2. 间距应该是因为这个translate每次都在之前的基础继续叠加，变成指数相关了。需要`save`一下状态
3. 清除失败应该也是因为没有save清除之后的状态。

再改一下！！！！

```javascript
const timer = setInterval(() => {
          ctx.save()
          ctx.beginPath();
          ctx.clearRect(0,0,450,150)
          ctx.translate(x, y)
          ctx.arc(70, 80, 50, 0, Math.PI * 2, true);
          ctx.stroke();
          ctx.closePath();
          ctx.restore();
          if(x >= 300) {
            x = 1
          } else {
            x += 1
          }
        }, 1000 / 60)
```

![1](/image/canvas/14.gif)

ohhhhhhhhhhh！成型！！！

### 总结

1. 重绘之前要清空、或者保留状态。
2. 画好一个东西之后要`closePath`

### 强化训练

做一个时钟~

![1](/image/canvas/15.gif)

[我做的源码](https://github.com/DerrickTel/study-demo/blob/main/canvas/bell.html)~

里面几乎用到了之前学到的所有知识。还是很值得练习的，最好是自己手写一份，我的只能算参考不算标准答案。

## 拖影动画

拖影效果的原理就是将之前的清除函数`clearRect`换成带透明度的一个蒙版覆盖上去

```javascript
ctx.fillStyle = 'rgba(255,255,255,0.3)';
ctx.fillRect(0,0, 450, 150);
// ctx.clearRect(0, 0, 450, 150)
```

直接替换一下之前做的时钟。

![1](/image/canvas/16.gif)

[源码](https://github.com/DerrickTel/study-demo/blob/main/canvas/bell.html)

## 事件

### 键盘事件

| 属性       | 描述                       |
| ---------- | -------------------------- |
| onkeydown  | 当按下按键时运行脚本       |
| onkeypress | 当按下并松开按键时运行脚本 |
| onkeyup    | 当松开按键时运行脚本       |

### 鼠标事件

通过鼠标触发事件, 类似用户的行为

| 属性         | 描述                                     |
| ------------ | ---------------------------------------- |
| onclick      | 当单击鼠标时运行脚本                     |
| ondblclick   | 当双击鼠标时运行脚本                     |
| ondrag       | 当拖动元素时运行脚本                     |
| ondragend    | 当拖动操作结束时运行脚本                 |
| ondragenter  | 当元素被拖动至有效的拖放目标时运行脚本   |
| ondragleave  | 当元素离开有效拖放目标时运行脚本         |
| ondragover   | 当元素被拖动至有效拖放目标上方时运行脚本 |
| ondragstart  | 当拖动操作开始时运行脚本                 |
| ondrop       | 当被拖动元素正在被拖放时运行脚本         |
| onmousedown  | 当按下鼠标按钮时运行脚本                 |
| onmousemove  | 当鼠标指针移动时运行脚本                 |
| onmouseout   | 当鼠标指针移出元素时运行脚本             |
| onmouseover  | 当鼠标指针移至元素之上时运行脚本         |
| onmouseup    | 当松开鼠标按钮时运行脚本                 |
| onmousewheel | 当转动鼠标滚轮时运行脚本                 |
| onscroll     | 当滚动元素的滚动条时运行脚本             |

### 实践

```javascript
c.addEventListener('mouseover', function(e){
  timer = setInterval(draw, 1000 / 60);
});

c.addEventListener('mouseout', function(e){
  clearInterval(timer);
});
```



## 像素操作

之前学习过对于图像的获取。现在要学习的是对于图像具体细节的获取，某个像素点的色彩。对于图像的操作，比如图像平滑操作以及如何从canvas画布中保存图像。

### ImageData 对象

`ImageData` 对象中存储着 `canvas` 对象真实的像素数据

它提供了以下属性

| 属性             | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| ImageData.data   | 只读，`Uint8ClampedArray` 类型的一维数组，包含着 `RGBA` 格式的整型数据，范围在 `0` 至 `255` 之间（ 包括 255 ）顺序的数据， |
| ImageData.height | 只读，无符号长整型（unsigned long），图片高度，单位是像素    |
| ImageData.width  | 只读，无符号长整型（unsigned long），图片宽度，单位是像素    |

### Uint8ClampedArray

`Uint8ClampedArray` 是一个 `高度 × 宽度 × 4 bytes` 的一维数组

这是什么意思呢 ？

直接上图

![1](/image/canvas/17.png)

可以看到他每四个单位里面就会放好一个点位的`RGBA`对应的数值。那我们可以做些什么呢？

很简单！取色器！

### 取色器

结合之前学到的事件，配置鼠标事件，将鼠标当前的颜色输出展示出来。

```javascript
const canvas = document.getElementById("canvas");

const div = document.getElementById("div");
const span = document.getElementById("span");

if (canvas.getContext) {
  const ctx = canvas.getContext("2d");
  const img = new Image()
  img.src = './img/colorPicker.jpeg'
  img.onload = function() {
    ctx.drawImage(img, 0, 0);
    img.style.display = 'none';
  };

  function pick(event) {
    const x = event.offsetX;
    const y = event.offsetY;
    const pixel = ctx.getImageData(x, y, 1, 1);
    const data = pixel.data;

    const rgba = `rgba(${data[0]}, ${data[1]}, ${data[2]}, ${data[3] / 255})`;
    div.style.background = rgba;
    span.textContent = rgba
  }

  canvas.addEventListener('mousemove', pick)
}
```

![1](/image/canvas/18.gif)

既然可以取到颜色那么，对于颜色的修改就可以直接赋值操作了。比如加滤镜之类的，这个需要上网找特定的颜色修改公式了。不展开啦。



## 保存

[`HTMLCanvasElement`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLCanvasElement) 提供一个 `toDataURL()` 方法，此方法在保存图片的时候非常有用。它返回一个包含被类型参数规定的图像表现格式的[数据链接](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/Data_URLs)。返回的图片分辨率是 96 dpi。

- [`canvas.toDataURL('image/png')`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLCanvasElement/toDataURL)

  默认设定。创建一个 PNG 图片。

- [`canvas.toDataURL('image/jpeg', quality)`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLCanvasElement/toDataURL)

  创建一个 JPG 图片。你可以有选择地提供从 0 到 1 的品质量，1 表示最好品质，0 基本不被辨析但有比较小的文件大小。

当你从画布中生成了一个数据链接，例如，你可以将它用于任何[``](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/image)元素，或者将它放在一个有 download 属性的超链接里用于保存到本地。

你也可以从画布中创建一个[`Blob`](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob)对像。

- [`canvas.toBlob(callback, type, encoderOptions)`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLCanvasElement/toBlob)

  这个创建了一个在画布中的代表图片的 `Blob` 对像。

```javascript
const canvas = document.getElementById("canvas");
// 创建一个 a 标签，并设置 href 和 download 属性
const el = document.createElement('a');
// 设置 href 为图片经过 base64 编码后的字符串，默认为 png 格式
el.href = canvas.toDataURL();
el.download = '文件名称';

// 创建一个点击事件并对 a 标签进行触发
const event = new MouseEvent('click');
el.dispatchEvent(event);
```



## 结语

至此`canvas`基本上算是小成了

有问题欢迎留言交流

[所有的源码点我](https://github.com/DerrickTel/study-demo/tree/main/canvas)
