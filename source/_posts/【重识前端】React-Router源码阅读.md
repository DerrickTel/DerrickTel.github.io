### title: 【重识前端】React-Router源码阅读
date: 2020-11-14 20:29:17
tags: [源码阅读, React]
category: [重拾前端]
cover: /image/cover/web.jpeg

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

## react-router和react-router-dom的区别。

### 先看提供的API

```javascript
import { Switch, Route, Router } from 'react-router';

import { Swtich, Route, BrowserRouter, HashHistory, Link } from 'react-router-dom';
```

#### React-router

提供了路由的核心api。如Router、Route、Switch等，但没有提供有关dom操作进行路由跳转的api；

#### React-router-dom

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

# SPA的核心思想

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

### createBrowserHistory

#### 返回的内容

之前一直有不断提到的`history`，我们一起来看看它是谁

源码连接：https://github.com/ReactTraining/history/blob/master/packages/history/index.ts

我们之前用到的`createBrowserHistory`，他其实是返回的一个对象，这个对象里面有我们常用的一些方法。

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

大部分精髓就是这里。发现有多熟悉的伙伴！ => `push`、`replace`、`go`等等。有些其实都是`window`提供的。

有些直接看代码就可以明白的就不解释了，比如`forward`，`back`。

##### go

比如上面提到的`go`方法，截取部分源码。

```typescript
let globalHistory = window.history;
function go(delta: number) {
    globalHistory.go(delta);
  }
```



##### createHref

```typescript

// 返回一个完整的url
export function createPath({
  pathname = '/',
  search = '',
  hash = ''
}: PartialPath) {
  return pathname + search + hash;
}

// 返回一个url
function createHref(to: To) {
  	// 看上面
    return typeof to === 'string' ? to : createPath(to);
  }
```



##### push

`push`的源码，里面附带了一些会用到的函数。

```typescript
// 就是简单处理一下返回值。
function getNextLocation(to: To, state: State = null): Location {
  return readOnly<Location>({
    ...location,
    ...(typeof to === 'string' ? parsePath(to) : to),
    state,
    key: createKey()
  });
}

// 进行判断
function allowTx(action: Action, location: Location, retry: () => void) {
    return (
      // 长度为0就返回true，长度大于0就调用函数，并传入参数。这个blockers等一下仔细探讨一下
      !blockers.length || (blockers.call({ action, location, retry }), false)
    );
  }


// 顾名思义获取state和url
function getHistoryStateAndUrl(
    nextLocation: Location,
    index: number
  ): [HistoryState, string] {
    return [
      {
        usr: nextLocation.state,
        key: nextLocation.key,
        idx: index
      },
      // 看上面 有专门介绍
      createHref(nextLocation)
    ];
  }

// 返回一些关于location的信息
function getIndexAndLocation(): [number, Location] {
    let { pathname, search, hash } = window.location;
    let state = globalHistory.state || {};
    return [
      state.idx,
      readOnly<Location>({
        pathname,
        search,
        hash,
        state: state.usr || null,
        key: state.key || 'default'
      })
    ];
  }

// 执行listeners内部的一些函数（也就是跳转），后面也会详细解读
function applyTx(nextAction: Action) {
    action = nextAction;
    [index, location] = getIndexAndLocation();
    listeners.call({ action, location });
  }


function push(to: To, state?: State) {
  	// 这里是一个枚举值
    let nextAction = Action.Push;
  	
    let nextLocation = getNextLocation(to, state);
  	// 顾名思义，就是再来一次
    function retry() {
      push(to, state);
    }

    if (allowTx(nextAction, nextLocation, retry)) {
      let [historyState, url] = getHistoryStateAndUrl(nextLocation, index + 1);

      // TODO: Support forced reloading
      // try...catch because iOS limits us to 100 pushState calls :/
      try {
        // MDN的地址: https://developer.mozilla.org/zh-CN/docs/Web/API/History/pushState
        globalHistory.pushState(historyState, '', url);
      } catch (error) {
        // They are going to lose state here, but there is no real
        // way to warn them about it since the page will refresh...
        // MDN的地址: https://developer.mozilla.org/zh-CN/docs/Web/API/Location/assign
        window.location.assign(url);
      }

      applyTx(nextAction);
    }
  }
```

我在注释里面已经进行非常详细的解读了，用到的每个函数都有解释或者官方权威的url。

总结一下：`history.push`的一个完整流程

- 调用`history.pushState`
  - 错误由`window.location.assign`来处理
- 执行一下`listeners`里面的函数

是的，你没有看错，就这么简单，只是里面有很多调用的函数，我都截取出来一一解释，做到每行代码都理解，所以显得比较长，概括来说就是这么简单。

重点来看一下`listen`和被调用的`createBrowserHistory`

##### replace

这里面用的函数，在前面的push都有解析，可以往上面去找找，就不多赘述了。

```typescript
function replace(to: To, state?: State) {
    let nextAction = Action.Replace;
    let nextLocation = getNextLocation(to, state);
    function retry() {
      replace(to, state);
    }

    if (allowTx(nextAction, nextLocation, retry)) {
      let [historyState, url] = getHistoryStateAndUrl(nextLocation, index);

      // TODO: Support forced reloading
      globalHistory.replaceState(historyState, '', url);

      applyTx(nextAction);
    }
  }
```



##### listen

在`history`返回的`listen`是一个函数。这个函数我们之前在react-router的源码中发现，他是在构造函数和卸载的时候会用到。

```javascript
listen(listener) {
      return listeners.push(listener);
    },
```

仔细看看，这个`listeners`是在做些什么。

##### createEvents

这个函数后面的blockers也会用到

```typescript
let listeners = createEvents<Listener>();

function createEvents<F extends Function>(): Events<F> {
  let handlers: F[] = [];

  return {
    get length() {
      return handlers.length;
    },
    push(fn: F) {
      handlers.push(fn);
      return function() {
        handlers = handlers.filter(handler => handler !== fn);
      };
    },
    call(arg) {
      handlers.forEach(fn => fn && fn(arg));
    }
  };
}
```

这个方法，顾名思义，就是创建事件。定义了一个变量 `handlers` 数组，用于存放要处理的回调函数事件。

然后返回了一个对象。

`push` 方法就是往 `handlers` 中添加要执行的函数。

这块主要在 `history.listen()` 中使用，可以翻到开头看下 `history` 中返回了 `listen()` 方法，就是调用了`listeners.push(listener)` 。

最后 `call()` 方法就比较容易理解，就是取出 `handlers` 里面的回调函数并逐个执行。

总结一下，就是存储一下`push`进来的函数，并进行过滤。之后调用的时候会依次执行。`length`就是当前拥有的函数数量。

再切回去，就会发现，每次调用这个`listen`就相当于`push`一个函数到内部的一个变量`handlers`中。

##### block

和`listeners`一样，也是用`createEvents`创建的，就不多说啦。说一下哪里会用到这个吧。

```typescript
let blockers = createEvents<Blocker>();
```

- `block(prompt)` - (function) Prevents navigation (see [the history docs](https://github.com/ReactTraining/history/blob/master/docs/blocking-transitions.md))

这个是`react-router`官网的解释。

我这里简单概括一下，就是用于关闭或者回退浏览器的误操作会用到的。详细的可以看点击进去查看。

##### 总结

至此，我们调用的`createBrowserHistory`所返回的一些属性的源码都已经了如指掌了。但是具体是怎么工作还是一知半解。

#### 具体核心原理

先秀一下源码。history的核心原理就是这个。先别被这么多行代码唬到了，很多都是我们在之前的push里面有解释的

```typescript
const PopStateEventType = 'popstate';
let blockedPopTx: Transition | null = null;
function handlePop() {
  // 如果有
  if (blockedPopTx) {
    blockers.call(blockedPopTx);
    blockedPopTx = null;
  } else {
    let nextAction = Action.Pop;
    let [nextIndex, nextLocation] = getIndexAndLocation();

    if (blockers.length) {
      if (nextIndex != null) {
        let delta = index - nextIndex;
        if (delta) {
          // Revert the POP
          blockedPopTx = {
            action: nextAction,
            location: nextLocation,
            retry() {
              go(delta * -1);
            }
          };

          go(delta);
        }
      } else {
        // Trying to POP to a location with no index. We did not create
        // this location, so we can't effectively block the navigation.
        warning(
          false,
          // TODO: Write up a doc that explains our blocking strategy in
          // detail and link to it here so people can understand better what
          // is going on and how to avoid it.
          `You are trying to block a POP navigation to a location that was not ` +
          `created by the history library. The block will fail silently in ` +
          `production, but in general you should do all navigation with the ` +
          `history library (instead of using window.history.pushState directly) ` +
          `to avoid this situation.`
        );
      }
    } else {
      applyTx(nextAction);
    }
  }
}

window.addEventListener(PopStateEventType, handlePop);
```

重点说一下`window.addEventListener(PopStateEventType, handlePop)`

MDN地址：https://developer.mozilla.org/zh-CN/docs/Web/API/Window/popstate_event

其实就是监听路由的变化然后，执行回调函数

# 核心API

