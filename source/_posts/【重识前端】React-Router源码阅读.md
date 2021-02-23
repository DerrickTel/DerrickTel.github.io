---
title: 【重识前端】React-Router源码阅读
date: 2020-11-14 20:29:17
tags: [源码阅读, React]
category: [重拾前端]
cover: /image/cover/web.jpeg
---

# 前言

此处介绍一下React-Router的核心原理。特别细致的标点符号的不予讨论。

# 学前小知识

`React-Router`其实最核心的东西是`Route`组件和由统一作者开发的`History`库来建立的。接下来跟着镜头一起走进神秘的`ßReact-Router`世界吧。

已经知道怎么使用的直接跳过，到下面的源码分析去┗|｀O′|┛ 嗷~~

# 简单示例

一起建一个简单的示例吧。先用`react`官网的`create-react-app`脚手架弄个`react`出来。

```sh
npx create-react-app my-app
cd my-app
npm start
```

安装[react-router-dom](https://reactrouter.com/web/guides/quick-start)

```sh
npm install react-router-dom
```

在大家安装之余，我简答的介绍一下`react-router`和`react-router-dom`的区别。

## `react-router`和`react-router-dom`的区别。

### 先看提供的API

```javascript
import { Switch, Route, Router } from 'react-router';

import { Swtich, Route, BrowserRouter, HashHistory, Link } from 'react-router-dom';
```

#### React-router：

提供了路由的核心api。如Router、Route、Switch等，但没有提供有关dom操作进行路由跳转的api；

#### React-router-dom：

提供了`BrowserRouter`、`Route`、`Link`等api，可以通过dom操作触发事件控制路由。

`Link`组件，会渲染一个a标签；`BrowserRouter`和`HashRouter`组件，前者使用`pushState`和`popState`事件构建路由，后者使用` hash `和 `hashchange `事件构建路由。

`react-router-dom`在`react-router`的基础上扩展了可操作`dom`的`api`。

`Swtich` 和 `Route` 都是从`react-router`中导入了相应的组件并重新导出，没做什么特殊处理。

`react-router-dom`中`package.json`依赖中存在对`react-router`的依赖，故此，不需要`npm`安装`react-router`。

> - 可直接 npm 安装 react-router-dom，使用其api。

## 简单修改

👌，大家🔥应该已经安装完了吧？接下来简单的修改一下脚手架里面的内容。主要是为了熟悉对手，知己知彼百战百胜。

![demo](/image/router/demo.png)

修改一下`src/app.js`

```javascript
import React from 'react';
import {
  BrowserRouter as Router,
  Switch,
  Route,
  Link,
} from "react-router-dom";

function Home() {
  return (
    <>
      <h1>首页</h1>
      <Link to="/login">登录</Link>
    </>
  )
}

function Login() {
  return (
    <>
      <h1>登录页</h1>
      <Link to="/">回首页</Link>
    </>
  );
}

function App() {
  return (
    <Router>
      <Switch>
        <Route path="/login" component={Login}/>
        <Route path="/" component={Home}/>
      </Switch>
    </Router>
  );
}

export default App;
```

这样就完成了一个最简单的示例了。

# 源码攻读

## BrowserRouter

从前面的简单示例中我们，发现有一个最外层的伙计，叫`BrowserRouter`。我们直接干到他的github源码去瞅瞅。

链接：https://github.com/ReactTraining/react-router/blob/master/packages/react-router-dom/modules/BrowserRouter.js

```javascript
import React from "react";
import { Router } from "react-router";
import { createBrowserHistory as createHistory } from "history";
import PropTypes from "prop-types";
import warning from "tiny-warning";

/**
 * The public API for a <Router> that uses HTML5 history.
 */
class BrowserRouter extends React.Component {
  history = createHistory(this.props);

  render() {
    return <Router history={this.history} children={this.props.children} />;
  }
}

if (__DEV__) {
  BrowserRouter.propTypes = {
    basename: PropTypes.string,
    children: PropTypes.node,
    forceRefresh: PropTypes.bool,
    getUserConfirmation: PropTypes.func,
    keyLength: PropTypes.number
  };

  BrowserRouter.prototype.componentDidMount = function() {
    warning(
      !this.props.history,
      "<BrowserRouter> ignores the history prop. To use a custom history, " +
        "use `import { Router }` instead of `import { BrowserRouter as Router }`."
    );
  };
}

export default BrowserRouter;

```

抛开一些七七八八的判断。重新看。

```javascript
import React from "react";
import { Router } from "react-router";
import { createBrowserHistory as createHistory } from "history";

class BrowserRouter extends React.Component {
  history = createHistory(this.props);

  render() {
    return <Router history={this.history} children={this.props.children} />;
  }
}
```

写着几个大字：《瓜子二手车》他只是一个中间商，他在赚差价。（打钱！

`BrowserRouter`是依赖于两个库：分别为`history`和`react-router`。ok，我们一探究竟。

## react-router



## history

