---
title: 【重识前端】redux源码阅读
date: 2020-09-11 16:10:35
tags: [源码阅读, React]
category: [重拾前端]
cover: /image/cover/web.jpeg
---

## 前言

之前一直在使用redux，后来出了hooks，用的比较少了，但是还是会使用，他们不冲突，各自对应的场景不同。听说redux源码比较少，也比较好懂（不是贬低，越强的代码其实越简洁明了）

所以准备阅读一下，也算是重识前端系列的一员吧。

注意一下，其实我们写react的时候用到的其实是redux-react。后面也会一起介绍。

更好的阅读体验其实是在项目里面打断点，这里是我的[GitHub地址](https://github.com/DerrickTel/redux-source-analyse)，喜欢的同学可以fork或者star一下哦。里面我把源码都拉出来放到文件夹里面然后由create-react-app来调用，而不是直接用npm装。这有助于理解与调试，里面也有我的注释，其实可以直接阅读源码的注释会更快。

redux和rdux-react的代码也是目前最新（2020-09-13）的master分支上面的。

## 开始

redux是用的rollup打包的。

打开根目录下面的`rollup.config.js`可以看到入口是src目录下的index文件。

可以看到他导出的一些东西

```js
export {
  createStore,
  combineReducers,
  bindActionCreators,
  applyMiddleware,
  compose,
  __DO_NOT_USE__ActionTypes
}
```

我们一个个开始。首先映入眼帘的是createStore

### createStore

开始之前，我们先看看是怎么用。以下采用[我的源码](https://github.com/DerrickTel/redux-source-analyse)阅读里面的代码：

```js
// middleware
const logger = (store:any) => (next:any) => (action:any) => {
  // debugger
  console.info('dispatching', action)
  let result = next(action);
  console.log('next state', store.getState())
  return result
}

let store = createStore(rootReducer, applyMiddleware(logger));
```

再看看源码的createStore。

```js
export default function createStore<
  S,
  A extends Action,
  Ext = {},
  StateExt = never
>(
  reducer: Reducer<S, A>,
  preloadedState?: PreloadedState<S> | StoreEnhancer<Ext, StateExt>,
  enhancer?: StoreEnhancer<Ext, StateExt>
): Store<ExtendState<S, StateExt>, A, StateExt, Ext> & Ext {
  //.. 一些校验的源码就不介绍了，想了解的可以看我的GitHub，上面有注释
  let currentReducer = reducer // 临时存放 reducer 的地方
  let currentState = preloadedState as S // 临时存放 state 的地方
  let currentListeners: (() => void)[] | null = [] // 监听队列
  let nextListeners = currentListeners // 引用赋值, 和正式的队列进行区分, 别有他用
  let isDispatching = false // 是不是正在dispatch
  // ... 
}
```

createStore接收3个参数`reducer`，`preloadedState`，`enhancer`。

我们一一剖析他们分别是做什么的。

#### reducer

这个大家太熟悉了。就是一个函数，然后我们在里面写`switch`，`switch` `dispatch`过来 `action`的`type`。然后做对应的操作。最后返回一个新的对象，也就是全新的`store`的数据。为什么是返回一个全新的`store`，是为了防止js的引用。

忘记的，或者不太懂的同学可以看一下下面的demo：

```js
const initState = {
  todos: []
}

// 这里面我们一般会赋值上默认的state，因为后续可能会用到默认的情况，像数组之类的就非常常见。
const todos = (state = initState, action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        ...state
      }
    case 'TOGGLE_TODO':
      return state.map(
        todo =>
          todo.id === action.id ? { ...todo, completed: !todo.completed } : todo
      )
    default: {
      return state
    }
      
  }
}

export default todos
```



#### preloadedState

这个其实我们用的不是很多，因为我们一般预存的都直接卸载`reducer`里面了。像👆🌰里面的`state = initState`就省去了我们的传递这个参数的过程。而且数据放在`reducer`里面也更加清晰。

哪怕是在[阮一峰老师的日志](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_one_basic_usages.html)里面也是这样教的。

```js
const defaultState = 0;
const reducer = (state = defaultState, action) => {
  switch (action.type) {
    case 'ADD':
      return state + action.payload;
    default: 
      return state;
  }
};

const state = reducer(1, {
  type: 'ADD',
  payload: 2
});
```

中文翻译就是：预存的`state`。也可以理解为初始的，默认的。因为我们一般在也页面里面直接拿`store`里面的值，或者`map`或者什么。很有可能就会报错，因为如果没有默认值我们直接用`map`就会报错....相信刚刚开始用的小伙伴应该有这样的困惑。

但是，我们通常都是传两个参数，那咋办？我看他好像是顺序读取的诶...

redux兼容我们啦，我们可以看到前面有一段代码是为了做这个的

```js
// preloadedState为function enhancer为undefined的时候说明initState没有初始化, 但是有middleware
if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
  enhancer = preloadedState // 把 preloadedState 赋值给 enhancer
  preloadedState = undefined // preloadedState赋值undeifined
}
```



#### enhancer

中文翻译是：增强。其实就是我们常用的中间件，用于增强我们的`redux`。常见有的有`log`，`redux-thunk`，`redux-saga`等等。

我们发现代码里面有一段是这样的：

```js
// debugger
// 如果参数enhancer存在
  if (typeof enhancer !== 'undefined') {
    // 如果enhancer存在，那他必须是个function, 否则throw Error哈
    if (typeof enhancer !== 'function') {
      throw new Error('Expected the enhancer to be a function.')
    }
    /**
     * 传入符合参数类型的参数，就可以执行 enhancer,
     * 但是这个return深深的吸引了我, 因为说明有applyMiddleware的时候后面的都不用看了 ??? 当然不可能
     * 可是applyMiddleware其实是必用项，所以猜想一下applyMiddleware强化store之后会enhancer赋值undefined，再次调用createStore
     * 上下打个debugger看一下执行顺序(debugger位置以注释)，果然不出所料
     * 好了， 假设我们还不知道applyMiddleware()这个funcrion具体干了什么，
     * 只知道他做了一些处理然后重新调用了createStore并且enhancer参数为undefined
     * 先记下，后续在看applyMiddleware， 因为我们现在要看的是createStore
     * * */
    // debugger
    return enhancer(createStore)(reducer, preloadedState)
  }
```

这个重点其实在于`applyMiddleware`