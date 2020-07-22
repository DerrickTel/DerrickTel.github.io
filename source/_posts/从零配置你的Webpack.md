---
title: 从零配置你的Webpack
date: 2020-06-07 06:11:13
tags: [Webpack]
category: [Webpack]
cover: /image/cover/webpack.png
---

# 从零配置你的Webpack

## 前言

上篇文章简单的介绍了一下，前端进阶之脚手架的搭建。其实我个人认为重要的还是Webpack的配置出来的template，至于脚手架的交互体验可以后期去优化，也可以更加的个性化。但是我们的核心还是放在Webpack等一系列的配置上。

这篇文章的目的

- 可以给自己一个回顾的地方
- 加强对Webpack的理解，每个知识点都会认认真真的彻查！尽量让每一步都是非常清晰明了的！
- 如果可以帮助到大家那是更好不过了

好了，收！话不多说开始！



## 前提

相信来到这里的小伙伴，都是有一些些前端的经验了，至于什么node安装环境变量这里就不赘述了。本人使用的是mac os，如果是Windows的并且碰到问题的话，可以留言，或者直接Google。



## 开始

### 建立一个空的文件夹📁

新建一个文件夹，名为【webpackInit】

并且使用你的编辑器打开他，然后打开命令行执行：

```js
npm init -y
```

这个命令是node帮你初始化一个项目用的，帮你新建一个`package.json`。

至于`-y`是用于默认都以**yes**继续执行。如果想了解一下里面到底有什么的同学可以不用*-y*继续跑一遍。其实里面的东西后期都可以自己修改`package.json`。所以不需要太在意。

### 安装Webpack🔧

现在是北京时间：2020/06/07 06:42:49。

Webpack5有Beta版，这里就不考虑了，后续上正式版的话我应该会出新的文章介绍。

因为我们使用的是 `webpack 4+` 版本，还需要安装 `webpack-cli` ，执行以下命令：

```js
npm install --save-dev webpack webpack-cli
```

因为Webpack主要是编译时使用所以放到“devDependencies”。

确认一下现在的目录结构，以防有同学掉队！

```diff
webpackInit
  |- node_modules
  |- package-lock.json
  |- package.json
```

这里说下题外话。

##### package.json和package-lock.json的区别

package.json 文件只能锁定大版本，也就是版本号的第一位，并不能锁定后面的小版本，`npm install` 都是拉取的该大版本下的最新的版本，为了稳定性考虑我们几乎是不敢随意升级依赖包的，这将导致多出来很多工作量，测试/适配等，所以 package-lock.json 文件出来了，当你每次安装一个依赖的时候就锁定在你安装的这个版本。

那如果我们安装时的包有bug，后面需要更新怎么办？

在以前可能就是直接改 package.json 里面的版本，然后再 `npm install` 了，但是 5 版本后就不支持这样做了，因为版本已经锁定在 package-lock.json 里了，所以我们只能 `npm install xxx@x.x.x` 这样去更新我们的依赖，然后 package-lock.json 也能随之更新。

### 新建配置文件📃

我们在根目录新建文件夹【config】用于存储一些相关的配置文件，然后在【config】里面新建一个文件夹【webpack】表示，专门用于存储webpack的配置文件。然后在【webpack】这个文件夹下面新建文件【webpack.common.config.js】

并敲入以下代码



```javascript
const path = require('path');

module.exports = {
  // 配置入口文件
  entry: {
    app: './src/index.js',
  },
  // 打包📦之后的出口
  output: {
    // 如果不加哈希值，浏览器会有缓存，可能你部署了，但是用户看到的还是老页面
    // 8是hash的长度，如果不设置，webpack会设置默认值为20。
    filename: 'js/[name].[chunkhash:8].bundle.js',
    /**
     * Node.js 中，__dirname 总是指向被执行 js 文件的绝对路径，
     * 所以当你在 /d1/d2/myscript.js 文件中写了 __dirname， 它的值就是 /d1/d2 。
     * path.resolve
     * 1.path.resolve()方法可以将路径或者路径片段解析成绝对路径
     * 2.传入路径从右至左解析，遇到第一个绝对路径是完成解析，例如path.resolve('/foo', '/bar', 'baz') 将返回 /bar/baz
     * 3.如果传入的绝对路径不存在，那么当前目录将被使用
     * 4.当传入的参数没有/时，将被传入解析到当前根目录
     * 5.零长度的路径将被忽略
     * 6.如果没有传入参数，将返回当前根目录
     * 
     * _dirname表示绝对路径
     * 我们碰到的./xx就是相对路径
     * 1.只传入__dirname也可以自动调用path.resolve方法
     * 2.可以拼接路径字符串，但是不调用path.resolve()方法拼接失败
     * 3.__dirname代表的是当前文件（a.js）的绝对路径
     * 4.从右至左解析，遇到了绝对路径/src，因此直接返
     */
    path: path.resolve(__dirname, '../../dist')
  }
}
```

> webpack 配置是标准的 Node.js的CommonJS 模块，它通过require来引入其他模块，通过module.exports导出模块，由 webpack 根据对象定义的属性进行解析。

在根目录新建【src】文件夹📁

在【src】文件夹下新建文件index.js

ok👌，确认一下目录结构

```diff
webpackInit
+ |- config
+ 	|- webpack
+     |- webpack.common.config.js
  |- node_modules
+ |- src
+     |- index.js
  |- package.json
  |- package-lock.json
```

那我们怎么打包呢？在 `package.json` 中配置如下属性：

```diff
"scripts": {
- "test": "echo \"Error: no test specified\" && exit 1",
+ "start": "webpack --config ./config/webpack/webpack.common.config.js"
},

```

好了，我们试试怎么打包吧，虽然你的 `index.js` 中什么代码也没有。
在控制台中输入以下代码：

```
npm run start
```

npm run xxxx 会去执行当前目录下package.json里面的script同名脚本

我们的【npm run start】相当于直接执行了我们写在【start】里面的代码。

执行之后，你会发现根目录多出了一个文件夹： `dist/js` ，其中有一个js文件： `bundle.js` ，那么至此，我们已经成功编译打包了一个js文件，即入口文件： `index.js` 。

### 安装React

在控制台输入以下代码：

```
npm install --save react react-dom
```

--save就是运行时会用到的代码

具体和--dev-save的区别可以自己Google一下

在【src/index.js】里面加入以下代码

```js
import React from 'react';
import ReactDOM from 'react-dom';

const App = () => (
  <>
    Hello World！
  </>
)

ReactDOM.render(<App />, document.getElementById('root'));
```

在根目录加入文件夹【public】，然后在【public】里面加入【index.html】

目录如下：

```diff
webpackInit
+ |- public
+ 	|- index.html
  |- config
  	|- webpack
      |- webpack.common.config.js
  |- node_modules
  |- src
      |- index.js
  |- package.json
  |- package-lock.json
```

然后在【index.html】加入以下代码：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible">
    <meta name="viewport" content="initial-scale=1.0, user-scalable=no, width=device-width">
  <title>从零配置Webpack</title>
</head>
<body>
  <div id="root"></div>
</body>
</html>
```

OK

万事俱备，我们运行：

```
npm run start
```

打包失败了。。。为什么呢？

### 使用babel

为什么我们上面写jsx会打包不了呢，因为webpack根本识别不了jsx语法，那怎么办？使用loader对文件进行预处理。
其中，babel-loader，就是这样一个预处理插件，它加载 ES2015+ 代码，然后使用 Babel 转译为 ES5。那开始配置它吧！

首先安装babel相关的模块：

```
npm install --save-dev babel-loader @babel/preset-react @babel/preset-env @babel/core babel-plugin-import
```

- **babel-loader：**使用Babel和webpack来转译JavaScript文件。
- **@babel/preset-react：**转译react的JSX
- **@babel/preset-env：**转译ES2015+的语法
- **@babel/core：**babel的核心模块
- **babel-plugin-import**：按需加载所需要的babel解析

理论上我们可以直接在 `webpack.common.config.js` 中配置"options"，但最好在当前根目录，注意，一定要是根目录！！！ 新建一个配置文件 `.babelrc` 配置相关的"presets"：

```js
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "targets": {
          // 大于相关浏览器版本无需用到 preset-env
          "edge": 17,
          "firefox": 60,
          "chrome": 67,
          "safari": 11,
          // 兼容到android4 ios6
          "browsers": ["Android >= 4.0", "ios >= 6"]
        }
      }
    ],
    "@babel/preset-react"
  ],
  "plugins": [
    ["import", { "libraryName": "antd-mobile", "style": "css" }] // `style: true` 会加载 less 文件
  ]
}
```

这里有关[bebel的配置](https://www.babeljs.cn/docs/babel-preset-env)可上官网查询文档。

再修改 `webpack.common.config.js` ，添加如下代码：

```js
const path = require('path');

module.exports = {
  // 配置入口文件
  entry: {
    app: './src/index.js',
  },
  // 打包📦之后的出口
  output: {
    // 如果不加哈希值，浏览器会有缓存，可能你部署了，但是用户看到的还是老页面
    // 8是hash的长度，如果不设置，webpack会设置默认值为20。
    filename: 'js/[name].[chunkhash:8].bundle.js',
    /**
     * Node.js 中，__dirname 总是指向被执行 js 文件的绝对路径，
     * 所以当你在 /d1/d2/myscript.js 文件中写了 __dirname， 它的值就是 /d1/d2 。
     * path.resolve
     * 1.path.resolve()方法可以将路径或者路径片段解析成绝对路径
     * 2.传入路径从右至左解析，遇到第一个绝对路径是完成解析，例如path.resolve('/foo', '/bar', 'baz') 将返回 /bar/baz
     * 3.如果传入的绝对路径不存在，那么当前目录将被使用
     * 4.当传入的参数没有/时，将被传入解析到当前根目录
     * 5.零长度的路径将被忽略
     * 6.如果没有传入参数，将返回当前根目录
     * 
     * _dirname表示绝对路径
     * 我们碰到的./xx就是相对路径
     * 1.只传入__dirname也可以自动调用path.resolve方法
     * 2.可以拼接路径字符串，但是不调用path.resolve()方法拼接失败
     * 3.__dirname代表的是当前文件（a.js）的绝对路径
     * 4.从右至左解析，遇到了绝对路径/src，因此直接返
     */
    path: path.resolve(__dirname, '../../dist')
  },
  module: {
    /**
     * test 规定了作用于以规则中匹配到的后缀结尾的文件， 
     * use 即是使用 babel-loader 必须的属性， 
     * exclude 告诉我们不需要去转译"node_modules"这里面的文件。
     */
    rules:[
      {
        test: /\.(js|jsx)?$/,
        // 开启缓存
        options: { cacheDirectory: true },
        loader: 'babel-loader',
      },
    ]
  }
}
```

接下来激动人心的时刻：

```
npm run start
```

是不是能打包成功了呢？

打开【dist/】你的html页面，看一下是否是“Hello World！”吧！

### 使用webpack-merge🈴️

我们将使用一个名为 [webpack-merge](https://github.com/survivejs/webpack-merge) 的工具。通过“通用”配置，我们不必在环境特定(environment-specific)的配置中重复代码。简单来说就是生产环境不同，我们要给的配置也有所不同，但是可以共用一个共有的配置。

我们先从安装 [webpack-merge](https://github.com/survivejs/webpack-merge) 开始：

```
npm install --save-dev webpack-merge
```

安装结束之后，我们在 `config` 这个文件夹下新建两个文件，分别为 `webpack.prod.config.js` 和 `webpack.dev.config.js` ，这两个文件分别对应生产和开发两个环境的配置。当然你也可以添加test环境。名字也可以自己取，尽量保持一致。



现在的目录结构：

```diff
  webpackInit
	|- config
		|- webpack
	    |- webpack.common.config.js
+     |- webpack.prod.config.js
  省略
```

在【webpack.prod.config.js】加入

```js
const merge = require('webpack-merge'); // 版本为4.x
// webpack-merge 5.x版本应该改为 const { merge } = require('webpack-merge');
const common = require('./webpack.common.config.js');

module.exports = merge(common, {
  mode: 'production',
});
```

然后修改【package.json】

```diff
{
  "name": "webpackInit",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "webpack --config ./config/webpack/webpack.common.config.js",
+   "build": "webpack --config ./config/webpack/webpack.prod.config.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "@babel/core": "^7.10.2",
    "@babel/preset-env": "^7.10.2",
    "@babel/preset-react": "^7.10.1",
    "babel-loader": "^8.1.0",
    "babel-plugin-import": "^1.13.0",
    "webpack": "^4.43.0",
    "webpack-cli": "^3.3.11",
    "webpack-merge": "^4.2.2"
  },
  "dependencies": {
    "react": "^16.13.1",
    "react-dom": "^16.13.1"
  }
}

```

然后

删除【dist文件夹】

之后

```
npm run build
```

是不是也build也打包成功了！

html是不是也可以正常访问！

但是没有显示Hello World，仔细一看还报错了。。。为什么呢。。因为

```
// 如果不加哈希值，浏览器会有缓存，可能你部署了，但是用户看到的还是老页面
    // 8是hash的长度，如果不设置，webpack会设置默认值为20。
    filename: 'js/[name].[chunkhash:8].bundle.js',
```

我们给js文件设置了hash值。不能够直接在html里面加上【<script src="../dist/js/bundle.js"></script>】这么一句，而且每次生成的hash值都会变，那么我们要怎么处理这个问题呢？



### 使用[HtmlWebpackPlugin](https://www.webpackjs.com/plugins/html-webpack-plugin/)



安装[HtmlWebpackPlugin](https://www.webpackjs.com/plugins/html-webpack-plugin/)

在控制台执行以下代码：

```
npm install --save-dev html-webpack-plugin
```

修改【webpack.prod.config.js】

```js
const merge = require('webpack-merge'); // 版本为4.x
// webpack-merge 5.x版本应该改为 const { merge } = require('webpack-merge');
const common = require('./webpack.common.config.js');

const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = merge(common, {
  mode: 'production',
  plugins: [
    new HtmlWebpackPlugin({
      filename: 'index.html', // 打包之后的html文件名字
      // 这里有小伙伴可能会疑惑为什么不是 '../public/index.html'
      // 我的理解是无论与要用的template是不是在一个目录，都是从根路径开始查找
      template: 'public/index.html', // 以我们自己定义的html为模板生成，不然我们还要到打包之后的html文件中写script
      inject: 'body',// 在body最底部引入js文件，如果是head，就是在head中引入js
      minify: { // 压缩html文件
        removeComments: true, // 去除注释
        collapseWhitespace: true, // 去除空格
      },
    })
  ]
});
```



现在我们再来打包试试

```
npm run build
```

看看dist中是不是多出了html文件，并且自动引入了script，用浏览器打开它试试看是不是能正确输出内容了！

起飞！！🛫️

### 使用clean-webpack-plugin

有些同学已经厌倦了每次都需要删除dist文件夹来验证是否成功。

其实假如我们不删除的话，我们需要修改js文件，这样让webpack知道我们改了东西，他就会重新打包，但是我们每次测试都没有去修改，所以我们需要删除。

但是，如果说我们不去删除dist文件夹的话，我们修改了【src/index.js】。然后再重新build，就会发现【dist/js】下面会又多出一个js文件，这样的话我们就需要观察日志，查看新鲜“出炉”的是哪一个，然后删掉别的。这样非常麻烦。

OK，我们现在就来解决一下这个问题

安装clean-webpack-plugin

```
npm install --save-dev clean-webpack-plugin
```

修改【webpck.prod.config.js】

```diff
const merge = require('webpack-merge'); // 版本为4.x
// webpack-merge 5.x版本应该改为 const { merge } = require('webpack-merge');
const common = require('./webpack.common.config.js');

const HtmlWebpackPlugin = require('html-webpack-plugin');
+ const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = merge(common, {
  mode: 'production',
  plugins: [
    new HtmlWebpackPlugin({
      filename: 'index.html', // 打包之后的html文件名字
      // 这里有小伙伴可能会疑惑为什么不是 '../public/index.html'
      // 我的理解是无论与要用的template是不是在一个目录，都是从根路径开始查找
      template: 'public/index.html', // 以我们自己定义的html为模板生成，不然我们还要到打包之后的html文件中写script
      inject: 'body',// 在body最底部引入js文件，如果是head，就是在head中引入js
      minify: { // 压缩html文件
        removeComments: true, // 去除注释
        collapseWhitespace: true, // 去除空格
      },
    }),
+   new CleanWebpackPlugin()
  ]
});
```

我们先查看现在的js文件前面的hash值。然后我们修改【src/index.js】，随便改成什么，再重新

```
npm run build
```

就会发现文件并没有新增，而且换了新鲜的哈希值。

当然了，我之前说的那些话，同学不相信的话，可以把【new CleanWebpackPlugin()】这一行注释掉，然后再修改【src/index.js】，会发现【dist/js】下面的js文件会增多一条。

### 使用webpack-dev-server

既然刚刚都提到优化了，我们每次都需要build一下，感觉很呆。而webpack官方也提供热部署，那我们现在就使用起来

安装webpack-dev-server

```
npm install webpack-dev-server --save-dev
```

新建文件【webpack.dev.config.js】📃

```diff
  webpackInit
	|- config
		|- webpack
	    |- webpack.common.config.js
      |- webpack.prod.config.js
+     |- webpack.dev.config.js
  省略
```

然后加入如下代码

```js
const path = require('path');
const merge = require('webpack-merge'); // 版本为4.x
// webpack-merge 5.x版本应该改为 const { merge } = require('webpack-merge');
const common = require('./webpack.common.config.js');

const webpack = require('webpack');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = merge(common, {
  mode: 'development',
  output: {
    filename: 'js/[name].[hash:8].bundle.js',
  },
  devServer: {
    contentBase: path.resolve(__dirname, '../dist'),
    open: true,
    port: 9000,
    compress: true,
    hot: true
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: 'public/index.html',
      inject: 'body',
      hash: false
    }),
    new webpack.HotModuleReplacementPlugin()
  ]
});
```

修改文件【package.json】📃

```diff
"scripts": {
-   "start": "webpack --config ./config/webpack/webpack.common.config.js",
+   "start": "webpack-dev-server --inline --config ./config/webpack/webpack.dev.config.js",
    "build": "webpack --config ./config/webpack/webpack.prod.config.js",
  },
```

然后我们

```
npm run start
```

是不是自动开了一个端口为9000的网页，上面是我们写的页面内容，这和我们的配置都是一一对应的。
现在你随意修改index.js中的代码，再回到页面看下是不是也跟着变了，那我们就整合webpack-dev-server成功！

### 使用source-map

source-map可以展示我们代码的错误位置，因为我们的代码都是被webpack打包过的，只有机器看得懂，我们人类无法正常识别。所以需要他。

想试一下未开启是什么状态的同学可以自己故意把代码写错，然后看看控制台的报错。

开启也十分简单。

它的配置非常简单，只需要在 【webpack.dev.config.js】 中增加一个 `devtool` 属性即可！

```diff
module.exports = {
+  devtool: 'cheap-module-eval-source-map',
	//...
}
```

因为我们只有自己写代码的时候才需要查看，到生产环境就要关闭啦，不然我们的智慧结晶就要被【窃取】啦！所以我放在了

## 中场休息

这里回顾一下知识点。

1. 我们先npm init 新建一个空白项目。
2. 然后安装webpack，react。
3. 发现无法编译jsx。
4. 所以我们寻求了babel的帮助，并配置了所需要解析的内容。
5. 觉得在html里面加入js很呆，所以引入了HtmlWebpackPlugin
6. 因为觉得每次删除打包出来的东西很呆，所以引入了clean-webpack-plugin
7. 因为每次都需要重新打包，所以使用webpack-dev
8. 至于webpack-merge是为了打包和编译两个或者说多个状态做预备的，在上述的例子只有本地的dev和build两个环境。

基本上上述的操作过程我都有解释，或者是注释，或者是在文章中说明。大家可以跟着节奏一步一步来，因为我也是一边写blog一边跑代码一边回顾知识点。

至此，webpack算是告一……

诶诶诶诶！js是可以解析了，那css呢！

哦哦，好的，那我们继续启程

## 重新起航

### 使用css-loader和style-loader两兄弟  

假如我们直接引用css的话，会报错了。这里就不演示了，有兴趣的同学可以自己试试。

所以，我们先走命令行敲击

```
npm install --save-dev style-loader css-loader 
```

#### 两兄弟的关系

来说说css-loader和style-loader他们这对鸳鸯的关系。

首先css-loader会把你的CSS文件进行解析，因为webpack是用JS写的，运行在node环境，所以默认webpack打包的时候只会处理JS之间的依赖关系！

所以我们之前的react里面的jsx需要babel的帮助，或者说需要【babel-loader】的帮助，所以我们的css同样需要【css-loader】的帮助，那么又关【style-loader】什么事？可不可以不装他呢？

答案是：可以的，但是你使用起来会非常的麻烦。怎么个麻烦法呢？

如果只用了【css-loader】解析出来的是这样的

```
["./src/index.css", ".test{↵  color: red;↵}", ""]
```

这样咋用嘛，你是解析了，可是你解析的是个XX。

这个时候就需要我们的天降猛男【style-loader】

style-loader 是通过一个JS脚本创建一个style标签，里面包含一些样式。style-loader是不能单独使用的，应为它并不负责解析 css 之前的依赖关系，每个loader的功能都是单一的，各自拆分独立。

#### 上手！

加入index.css

```diff
src
+  |- index.css
   |- indexjs
```

index.css文件的内容如下

```css
.test{
  color: red;
}
```

修改index.js

```js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css'

const App = () => (
  <div className="test">
    Hello World！!
  </div>
)

ReactDOM.render(<App />, document.getElementById('root'));
```

修改【webpack.common.config.js】📃

```diff
//...
rules:[
      {
        test: /\.(js|jsx)?$/,
        // 开启缓存
        options: { cacheDirectory: true },
        loader: 'babel-loader',
      },
+      {
+        test: /\.css$/,
+        use: [ 
+          'css-loader', 
+          'style-loader',
+        ]
+      }
    ]
    //....
```

记得重新运行，因为webpack的配置他读取一次，所以如果你修改了配置，需要【ctrl+c】关闭重新运行。

嘿嘿……是不是运行不了啊。

其实我是故意的，我想告诉大家一个知识点。

> loader的加载顺序是从右往左。这里的编译顺序是先用css-loader将css代码编译，再交给style-loader插入到网页里面去。所以css-loader在右，style-loader在左。

虽然我们的数组换行了，但是仔细看不难看出顺序。

所以我们只需要将他们换个位置就可以了。代码我就不贴了。

现在大家应该记忆很深刻了吧！

大家重新【ctrl+c】关闭重新运行就行了。我们的hello world是不是变红啦～

### 安装less-loader

说到css，说句实在话，没几人真的是在写纯css的吧？现在谁不是less，sass或者其他css预处理呢？而且这些预处理的好处我就不细说了，感兴趣的自己Google吧，本文用的是less（因为ant design用的也是less，哈哈，假装是蚂蚁的一员）

在命令行输入

```
npm install --save-dev less less-loader
```

less没什么好说的，用他肯定要装，less-loader，顾名思义，就是less的解析者。

在【webpack.common.config.js】增加

```diff
{
        test: /\.css$/,
        use: [ 
          'style-loader',
          'css-loader', 
        ]
      },
+      {
+        test: /\.less$/,
+        use: [ 
+          'style-loader',
+          'css-loader', 
+          'less-loader',
+        ]
+      }
```

依旧是顺序问题，先解析less，把less解析成常规的css，然后再解析css，最后插入到网页中去。

修改【index.js】引入自己写的【.less】文件

大家重新【ctrl+c】关闭重新运行就行了。至于你写了什么less特性，只要有效果就行了。

### 安装url-loader和file-loader

说完CSS，美丽的网页当然离不开我们动人的图片啦。

【file-loader】的作用是，把你的文件打包起来，和js文件放在一起，这样用户访问我们的网页的时候，其实也需要访问我们的url，既增加了服务器的压力，也增加了用户升级流量的压力，需要去下载这个文件。

【url-loader】

如果页面图片较多，发很多http请求，会降低页面性能。这个问题可以通过url-loader解决。url-loader会将引入的图片编码，生成dataURl并将其打包到文件中，最终只需要引入这个dataURL就能访问图片了。

url-loader和file-loader两兄弟的搭配可以有效的减少不必要的url请求，因为有的图片你要去请求url获取，如果小的话完全可以用base64替代。如果图片很大的话就用file-loader，这样可以减少编码的压力。

在命令行输入

```
npm install file-loader url-loader --save-dev
```

修改【webpack.common.config.js】📃

```diff
module: {
  rules: [
    //...
+    {
+      test: /\.(jpg|png|gif)$/,
+      use: {
+        loader: 'url-loader',
+        options: {
+          name: '[name].[ext]', //输出的文件名
+          outputPath: 'images/', // 输出到dist目录下的路径（dist/images/）
						/**
             * 如果你这个图片文件大于8192b，即8kb，那我url-loader就不用，转而去使用file-loader，
             * 把图片正常打包成一个单独的图片文件到设置的目录下，若是小于了8kb，
             * 那好，我就将图片打包成base64的图片格式插入到bundle.js文件中，
             * 这样做的好处是，减少了http请求，但是如果文件过大，js文件也会过大，
             * 得不偿失，这是为什么有limit的原因！
             */
+          limit: 8192,
+        },
+      }
+    }
  ]
}
```



## 进阶

基本上我们的webpack可以正常运行了，css，js，jsx都可以解析了。但是我们需要考虑一些进阶的东西，优化。

### 使用uglifyjs-webpack-plugin

在控制台执行以下代码：

```
npm install uglifyjs-webpack-plugin --save-dev
```

在【webpack.prod.config.js】添加代码

```js
const UglifyJsPlugin = require('uglifyjs-webpack-plugin');

module.exports = {
//...
  optimization: {
    minimizer: [
      new UglifyJsPlugin({
        test: /\.js(\?.*)?$/i,  //测试匹配文件,
        include: /\/includes/, //包含哪些文件
        excluce: /\/excludes/, //不包含哪些文件

        cache: false,   //是否启用文件缓存，默认缓存在node_modules/.cache/uglifyjs-webpack-plugin.目录
        parallel: true,  //使用多进程并行运行来提高构建速度

        //允许过滤哪些块应该被uglified（默认情况下，所有块都是uglified）。 
        //返回true以uglify块，否则返回false。
        chunkFilter: (chunk) => {
            // `vendor` 模块不压缩
            if (chunk.name === 'vendor') {
              return false;
            }
            return true;
          }
        }),
  
    ],
  },
  //..
};
```

### 使用splitChunks

其实我们写的代码，有些库的代码是不需要每次都编译的，最简单的例子就是React，这个我们几乎每个js文件都会用到。所以我们可以将它们单独打包，这样只需要打包一次。

修改【webpack.common.config.js】

```diff
entry: {
     index: './src/index.js',
+    common: ['react', 'react-dom']
  },
```

修改【webpack.prod.config.js】

```js
const merge = require('webpack-merge'); // 版本为4.x
// webpack-merge 5.x版本应该改为 const { merge } = require('webpack-merge');
const common = require('./webpack.common.config.js');

const HtmlWebpackPlugin = require('html-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

const UglifyJsPlugin = require('uglifyjs-webpack-plugin');

module.exports = merge(common, {
  mode: 'production',
  plugins: [
    new HtmlWebpackPlugin({
      filename: 'index.html', // 打包之后的html文件名字
      // 这里有小伙伴可能会疑惑为什么不是 '../public/index.html'
      // 我的理解是无论与要用的template是不是在一个目录，都是从根路径开始查找
      template: 'public/index.html', // 以我们自己定义的html为模板生成，不然我们还要到打包之后的html文件中写script
      inject: 'body',// 在body最底部引入js文件，如果是head，就是在head中引入js
      minify: { // 压缩html文件
        removeComments: true, // 去除注释
        collapseWhitespace: true, // 去除空格
      },
    }),
    new CleanWebpackPlugin()
  ],
  optimization: {
    minimizer: [
      new UglifyJsPlugin({
        test: /\.js(\?.*)?$/i,  //测试匹配文件,
        // include: /\/includes/, //包含哪些文件
        // excluce: /\/excludes/, //不包含哪些文件

        //允许过滤哪些块应该被uglified（默认情况下，所有块都是uglified）。 
        //返回true以uglify块，否则返回false。
        chunkFilter: (chunk) => {
            // `vendor` 模块不压缩
            if (chunk.name === 'vendor') {
              return false;
            }
            return true;
          }
        }),
  
        cache: false,   //是否启用文件缓存，默认缓存在node_modules/.cache/uglifyjs-webpack-plugin.目录
        parallel: true,  //使用多进程并行运行来提高构建速度
    ],
    splitChunks: {
      /**
       * 默认值是async
       * 拆分模块的范围，它有三个值async、initial和all。
       * async表示只从异步加载得模块（动态加载import()）里面进行拆分
       * initial表示只从入口模块进行拆分
       * all表示以上两者都包括
       */
      chunks: 'all',
      // minSize: 30000, // 生成chunk的最小大小（以字节为单位）。只有大于这个数字才可以成一个chunk 
      // minRemainingSize: 0, // 只有剩下一个chunk的时候才会生效，默认是和minSize一样的，开发的时候默认是0
      // maxSize: 0, // 告诉webpack尝试将大于maxSize字节的块拆分为较小的部分。
      // minChunks: 1, // 拆分前必须共享模块的最小块数。
      // maxAsyncRequests: 6, // 按需加载时最大并行请求数。
      // maxInitialRequests: 4, // 入口点的最大并行请求数。入口文件
      // automaticNameDelimiter: '~', // 默认情况下，webpack将使用块的来源和名称生成名称（例如vendors~main.js）。此选项使您可以指定用于生成名称的定界符。
      cacheGroups: {
        /**
         * 当webpack处理文件路径时，它们始终包含/在Unix系统和\Windows上。
         * 这就是为什么[\\/]在{cacheGroup}.test字段中使用in 来表示路径分隔符的原因。
         * /或\in {cacheGroup}.test会在跨平台使用时引起问题。
         */
        // defaultVendors: {
        //   test: /[\\/]node_modules[\\/]/, // 分块目标
        //   priority: -10 // 权重
        // },
        // default: {
        //   minChunks: 2, // 最小引用
        //   priority: -20, // 权重
        //   // 如果当前块包含已从主捆绑包中拆分出的模块，则将重用该模块，而不是生成新的模块。这可能会影响块的结果文件名。
        //   reuseExistingChunk: true
        // },
        // 上述的splitChunks全是webpack4未设置情况下的默认值，除了chunks从【async】->【all】其他都没有改
        // ok，我们现在加入我们自己想要的代码分割
        // 因为我准备加入react等业务千变万化而不会变的库
        common: {
          test: "common", // webpack扫面的关键字
          name: "common", // 生成的名字
          enforce: true // 是否缓存
        },
      }
    }
  },
});
```

不想写diff啦，直接CV啦。。。

为什么我需要把react提取出来，因为哪里都需要用，而且他几乎不可能会变，所以我特别提出来做了缓存，其余的还是使用的默认配置（除了chunk改为了‘all’）。

> Tobias Koppers@Wsokra：optimization. splitchunks. chunks: althe only option you need for vendor and commons splitting in webpackBest combine it with html-webpack-plugin or equivalent html generation18120

作者都发推特说了，那我们也就接受吧～就改个all，然后补一下react～

再重新打包，你会发现index.bundle.js（不被缓存）的hash值变了，但是common.bundle.js（能被缓存）的hash值没变。

### 使用mini-css-extract-plugin

js都独立📦，那我css也要！

其实如果把CSS打包成一个文件然后让html引用的话可以减小html文件的大小，暗合了HTTP2的多路复用，多文件小数量。包括之前的splitChunks里面我们配置的react也是为了HTTP2。

在命令行输入：

```
npm install --save-dev mini-css-extract-plugin
```

修改【webpack.common.config.js】

```diff
+ const MiniCssExtractPlugin = require('mini-css-extract-plugin');

//...
module:{
    rules:[
      {
        test: /\.css$/,
        use: [ 
+          MiniCssExtractPlugin.loader,
-          'style-loader',
           'css-loader', 
        ]
      },
    ],
  },
+plugins: [
+    new MiniCssExtractPlugin({
+      filename: 'css/[name].[hash].css',
+      chunkFilename: 'css/[id].[hash].css',
+    }),
  ]
```



### 使用CSS Module

其实很简单，只需要修改一下配置【wbpack.common.config.js】

```diff
{
        test: /\.css$/,
        use: [ 
          MiniCssExtractPlugin.loader,
-          'css-loader'
+          {
+            loader: 'css-loader',
+            options: {
+              // importLoaders: 1,
+              modules: true,
+            },
+          },
        ]
      },
```

### 使用PostCSS

> postcss 一种对css编译的工具，类似babel对js的处理，常见的功能如： 1 . 使用下一代css语法 2 . 自动补全浏览器前缀 3 . 自动把px代为转换成rem 4 . css 代码压缩等等 postcss 只是一个工具，本身不会对css一顿操作，它通过插件实现功能，autoprefixer 就是其一。

安装postcss

```
npm install postcss postcss-loader --save-dev
```

安装postcss某个插件，以Autoprefixer举例

```
npm install postcss-aspect-ratio-mini postcss-write-svg postcss-px-to-viewport postcss-viewport-units postcss-flexbugs-fixes postcss-preset-env cssnano --save-dev
```

在根目标新建文件【postcss.config.js】

```js
/* eslint-disable import/no-extraneous-dependencies */
/* eslint-disable @typescript-eslint/no-var-requires */
const postcssAspectRatioMini = require('postcss-aspect-ratio-mini');
const postcssPxToViewport = require('postcss-px-to-viewport');
const postcssWriteSvg = require('postcss-write-svg');
const postcssViewportUnits = require('postcss-viewport-units');
const cssnano = require('cssnano');
const postcssPresetEnv = require('postcss-preset-env')

const postcssFlexbugsFixes = require('postcss-flexbugs-fixes')

module.exports = {
  plugins: [
    postcssFlexbugsFixes,
    // 在这个位置加入我们需要配置的代码
    // 在这个位置加入我们需要配置的代码
    // 在这个位置加入我们需要配置的代码
    postcssAspectRatioMini({}),
    postcssPxToViewport({
      viewportWidth: 750, // 基准宽度（一般的设计都是这个基准
      viewportHeight: 1334, // 基准高度（一般的设计都是这个基准
      unitPrecision: 3, // (Number) The decimal numbers to allow the REM units to grow to.
      viewportUnit: 'vw', // (String) 单位
      selectorBlackList: ['.list-ignore', /notTransform/], // 带上这个单词的就不会fix为vw单位
      minPixelValue: 1, // (Number) 最小像素
      mediaQuery: false, // (Boolean) 允许在媒体查询中转换px。
      exclude: /(\/|\\)(node_modules)(\/|\\)/,
    }),
    postcssWriteSvg({
      utf8: false
    }),
    postcssPresetEnv({}),
    postcssViewportUnits({
      filterRule: rule => rule.selector.includes('::after')
        && rule.selector.includes('::before')
        && rule.selector.includes(':after')
        && rule.selector.includes(':before')
    }),
    cssnano({
      "cssnano-preset-advanced": {
        zindex: false,
        autoprefixer: false
      },
    })
  ]
};
```

修改【webpack.common.config.js】

```diff
{
        test: /\.css$/,
        use: [ 
          'style-loader',
          'css-loader', 
+          'postcss-loader'
        ]
      },
      {
        test: /\.less$/,
        use: [ 
          'style-loader',
          'css-loader', 
          'less-loader',
+          'postcss-loader'
        ]
      }
```

然后，修改我们的css文件，看看我们写的单位为px的有没有被改为vw的自适应单位。

顺便试一下带有【notTransform】的是不是还是px作为单位。

## 结束

到此，我们的webpack配置，就算是入门了，对于webpack的配置我们还有很长的路要走。大家加油！

如果有哪里写的不好或者写错了，欢迎大家在评论区讨论。