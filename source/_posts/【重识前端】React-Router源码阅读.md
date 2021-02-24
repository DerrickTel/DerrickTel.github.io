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

# 架构思路

- 监听`URL`的变化
- 改变某些`context`的值
- 获取对应的页面组件
- `render`新的页面

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

源码链接：https://github.com/ReactTraining/react-router/blob/master/packages/react-router/modules/Router.js



```javascript
import HistoryContext from "./HistoryContext.js";
import RouterContext from "./RouterContext.js";
```

这两个东西其实很简单，都是引用了一个叫做`createContext`，目的也很简单，这里其实就是创建的普通context，只不过拥有特定的名称而已。源码如下。就几行。

```javascript
// TODO: Replace with React.createContext once we can assume React 16+
import createContext from "mini-create-react-context";

const createNamedContext = name => {
  const context = createContext();
  context.displayName = name;

  return context;
};

export default createNamedContext;
```

OK，重点是接下来，抛开context之后，我们需要关注的东西。我整理一下。

先看构造函数

### 构造函数

```javascript
constructor(props) {
  super(props);

  this.state = {
    location: props.history.location
  };

  // This is a bit of a hack. We have to start listening for location
  // changes here in the constructor in case there are any <Redirect>s
  // on the initial render. If there are, they will replace/push when
  // they mount and since cDM fires in children before parents, we may
  // get a new location before the <Router> is mounted.
  
  // _isMounted 表示组件是否加载完成
  this._isMounted = false;
  // 组件未加载完毕，但是 location 发生的变化，暂存在 _pendingLocation 字段中
  this._pendingLocation = null;

  // 没有 staticContext 属性，表示是 HashRouter 或是 BrowserRouter
  if (!props.staticContext) {
    this.unlisten = props.history.listen(location => {
      if (this._isMounted) {
        // 组件加载完毕，将变化的 location 方法 state 中
        this.setState({ location });
      } else {
        this._pendingLocation = location;
      }
    });
  }
}
```

有两个值☞`_isMounted`和`_pendingLocation`

分别是  是否挂载   待定

内部维护了一个location，默认值是由外面传入的history，我找到传入的地方。

其实就是之前看到的`BrowserRouter`

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

不难发现，history的默认值就是由history这个库来提供的，这个后面会提到，我们继续看。

先简单看一下注释的翻译(谷歌翻译)

> 这有点hack。 我们必须开始在构造函数中监听位置更改，以防初始渲染中存在任何<Redirect>。 如果有的话，它们将在安装时替换/推动，并且由于cDM在父级之前在子级中触发，因此在安装<Router>之前，我们可能会获得一个新位置。 

什么意思呢，我简单描述一下：

> 因为子组件会比父组件更早渲染完成, 以及`<Redirect>`的存在, 若是在`<Router>`的`componentDidMount`生命周期中对`history.location`进行监听, 则有可能在监听事件注册之前, `history.location`已经由于`<Redirect>`发生了多次改变, 因此我们需要在`<Router>`的`constructor`中就注册监听事件

收回来，进入`if`继续看。

```javascript
this.unlisten = props.history.listen(location => {
  if (this._isMounted) {
  	this.setState({ location });
  } else {
  	this._pendingLocation = location;
  }
});
```

就是调用了`history`的`listen`方法。从代码中可以大致了解到就是对`history`进行监听，然后进行一些操作。

### componentWillUnmount

那么这个`unlisten`什么时候执行呢？

```javascript
componentWillUnmount() {
    if (this.unlisten) {
      this.unlisten();
      this._isMounted = false;
      this._pendingLocation = null;
    }
  }
```

在`Router`这个组件卸载的时候就执行啦。也就是说，取消了对`history`的监听。

### componentDidUnmount

然后看一下`conponentDidMount`

```javascript
componentDidMount() {
    this._isMounted = true;

    if (this._pendingLocation) {
      this.setState({ location: this._pendingLocation });
    }
  }
```

也很简单，就是修改一下是否挂载的值，以及继续之前在构造函数里面的判断。如果暂存数据有的话就把他存下来。

### render

```react
render() {
    return (
      <RouterContext.Provider
        value={{
          // 根据 HashRouter 还是 BrowserRouter，可判断 history 类型
          history: this.props.history,
          // 这个 location 就是监听 history 变化得到的 location
          location: this.state.location,
          // path url params isExact 四个属性
          match: Router.computeRootMatch(this.state.location.pathname),
          // 只有 StaticRouter 会传 staticContext
          // HashRouter 和 BrowserRouter 都是 null
          staticContext: this.props.staticContext
        }}
      >
        <HistoryContext.Provider
          children={this.props.children || null}
          value={this.props.history}
        />
      </RouterContext.Provider>
    );
  }
```

### 总结

Router这个组件主要就是将一些数据进行存储。存到`Context`，之间不乏一些特殊情况的判断，比如子组件渲染比父组件早，以及`Redirect`的情况的处理。在卸载的时候要移除对`history`的监听。

子组件作为消费者，就可以对页面进行修改，跳转，获取这些数值。



## history

之前一直有不断提到的`history`，我们一起来看看它是谁

源码连接：https://github.com/ReactTraining/history/blob/master/packages/history/index.ts#L559

```javascript
let history: BrowserHistory = {
    get action() {
      return action;
    },
    get location() {
      return location;
    },
    createHref,
    push,
    replace,
    go,
    back() {
      go(-1);
    },
    forward() {
      go(1);
    },
    listen(listener) {
      return listeners.push(listener);
    },
    block(blocker) {
      let unblock = blockers.push(blocker);

      if (blockers.length === 1) {
        window.addEventListener(BeforeUnloadEventType, promptBeforeUnload);
      }

      return function() {
        unblock();

        // Remove the beforeunload listener so the document may
        // still be salvageable in the pagehide event.
        // See https://html.spec.whatwg.org/#unloading-documents
        if (!blockers.length) {
          window.removeEventListener(BeforeUnloadEventType, promptBeforeUnload);
        }
      };
    }
  };

  return history;
}
```

大部分精髓就是这里。发现有多熟悉的伙伴！ => `push`、`replace`、`go`等等。这些其实都是`window`提供的。