---
title: 【重识前端】redux源码阅读
date: 2020-09-11 16:10:35
tags: [源码阅读, React]
category: [重拾前端]
cover: /image/cover/web.jpeg
---

最近在写【重拾前端】系列，下面有几个快速通道，大家自取

[【重识前端】原型/原型链和继承](https://juejin.im/post/6850418113944420359)

[【重识前端】闭包与模块](https://juejin.im/post/6850418117508759566)

[【重识前端】全面攻破this](https://juejin.im/post/6854573215658803208)

[【重识前端】一次搞定JavaScript的执行机制](https://juejin.im/post/6859911609633767438)

[【重识前端】什么是BFC、IFC、GFC 和 FFC](https://juejin.im/post/6860375101322461198)

[【重识前端】深入内存世界](https://juejin.im/post/6865204092877144077)

[【重识前端】暴走的异步编程](https://juejin.im/post/6867814019055484942)

[【重识前端】redux源码阅读](https://juejin.im/post/6877910314110140423/)

# 前言

之前一直在使用redux，后来出了hooks，用的比较少了，但是还是会使用，他们不冲突，各自对应的场景不同。听说redux源码比较少，也比较好懂（不是贬低，越强的代码其实越简洁明了）

所以准备阅读一下，也算是重识前端系列的一员吧。

注意一下，其实我们写react的时候用到的其实是redux-react。后面也会一起介绍。

更好的阅读体验其实是在项目里面打断点，这里是我的[GitHub地址](https://github.com/DerrickTel/redux-source-analyse)，喜欢的同学可以fork或者star一下哦。里面我把源码都拉出来放到文件夹里面然后由create-react-app来调用，而不是直接用npm装。这有助于理解与调试，里面也有我的注释，其实可以直接阅读源码的注释会更快。

redux和rdux-react的代码也是目前最新（2020-09-13）的master分支上面的。

# 开始

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

## createStore

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

### 参数

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

这个重点其实在于`applyMiddleware`，后续我们会介绍到的。

### 内置变量

这个比较简单，我就都写在注释里面了

```js
let currentReducer = reducer // 临时存放 reducer 的地方
let currentState = preloadedState as S // 临时存放 state 的地方
let currentListeners: (() => void)[] | null = [] // 监听队列
let nextListeners = currentListeners // 引用赋值, 和正式的队列进行区分, 别有他用
let isDispatching = false // 是不是正在dispatch
```



### 内置方法

#### ensureCanMutateNextListeners

这里其实是为了弄两个数组，一个操作一个原数组而抽象的一个方法。

有一个小的知识点，`===`对于数组的判断只能用于判断是否是同一个内存地址，不信的话大家可以试试。

```js
const a = [1, 2];
const b = [1, 2];
const c = a;
console.log(a === b);
console.log(a === c);
console.log(b === c);
```



所以下面的操作的就变得很好理解了。在我们操作`listener`之前会先存储一下快照。大家要多理解一下，因为后面很多地方会用到这个函数。

```js
// Google翻译: 确保可以使下一个侦听器突变
  // 我的理解是存储一下快照, 以为接下来可能会进行操作.
  function ensureCanMutateNextListeners() {
    // 是否相同, 是不是简单的引用赋值, 是的话就浅拷贝一份
    if (nextListeners === currentListeners) {
      nextListeners = currentListeners.slice()
    }
  }
```



#### getState

这个函数很简单，首先判断一下是否是在dispatch，然后返回当前的state。

```js
// 获取当前的state
  function getState(): S {
    // 如果正在dispatch 就报错,因为要获取最新的state, dispatch很有可能会改变state
    if (isDispatching) {
      throw new Error(
        'You may not call store.getState() while the reducer is executing. ' +
          'The reducer has already received the state as an argument. ' +
          'Pass it down from the top reducer instead of reading it from the store.'
      )
    }

    return currentState as S
  }
```

有没有感觉redux好像也不过如此~~接着看哦。

#### subscribe

中文翻译：订阅者。

其实也很好理解。可能有用过`redux`的同学都有用过这个。

我举一个怎么用的🌰：

```js
const unSubscribe = store.subscribe(() => {
  console.log('store.subscribe', store.getState())
})
// 取消订阅就直接调用这个函数就好了
// unSubscribe();
```

一旦触发了`dispatch`就会触发这个函数，也就是我们写上去的`console.log()`

然后我们再来看这个函数。

```js
function subscribe(listener: () => void) {
    // listener必须为函数，因为要以回调函数的方式来触发。
    if (typeof listener !== 'function') {
      throw new Error('Expected the listener to be a function.')
    }

    // 如果正在dispatch中则抛错，和getState同理
    if (isDispatching) {
      throw new Error(
        'You may not call store.subscribe() while the reducer is executing. ' +
          'If you would like to be notified after the store has been updated, subscribe from a ' +
          'component and invoke store.getState() in the callback to access the latest state. ' +
          'See https://redux.js.org/api/store#subscribelistener for more details.'
      )
    }

    // 是否有监听者,或者是否被订阅
    let isSubscribed = true

    // 保存快照, 拷贝一份监听队列到nextListeners(如果是引用的话)
    ensureCanMutateNextListeners()
    // 往监听队列里面推入一个监听者
    nextListeners.push(listener)

    // 返回一个取消订阅的方法
    return function unsubscribe() {
      // 如果没有被订阅, 直接shut down
      if (!isSubscribed) {
        return
      }

      // 如果正在dispatch 报错，和上面👆同理
      if (isDispatching) {
        throw new Error(
          'You may not unsubscribe from a store listener while the reducer is executing. ' +
            'See https://redux.js.org/api/store#subscribelistener for more details.'
        )
      }

      // 取消订阅
      isSubscribed = false

      // 依旧是拿下当前的监听队列，不然等一下删除的时候，会删掉所有的队列。其实我们只需要删掉监听者就好了
      ensureCanMutateNextListeners()
      // 找到监听者
      const index = nextListeners.indexOf(listener)
      // 删除这个监听者
      nextListeners.splice(index, 1)
      // 清空当前的监听队列
      currentListeners = null
    }
  }
```



#### dispatch

这个函数可以说是史上最TM常用的函数的，最TM重要之重中之重，而且还是更新store的唯一方法，注意这个唯一。但是又非常好理解。

我们还是先看一个🌰吧：

```js
props.dispatch({
  type: 'TEST',
  test1: 1,
  test2: 2,
  // ...
})
```

然后，我们的store就改变了，就会更新props，来触发render，视图可能就会更新之类的。

👌，现在我们有一个一点点了解，来看一下源码。

```js
// 修改store的唯一方法
  function dispatch(action: A) {
    // action必须是一个对象
    if (!isPlainObject(action)) {
      throw new Error(
        'Actions must be plain objects. ' +
          'Use custom middleware for async actions.'
      )
    }

    // action必须拥有一个type
    if (typeof action.type === 'undefined') {
      throw new Error(
        'Actions may not have an undefined "type" property. ' +
          'Have you misspelled a constant?'
      )
    }

    // 如果正在dispatching，那么不执行dispatch操作。
    if (isDispatching) {
      throw new Error('Reducers may not dispatch actions.')
    }

    // 设置dispatching状态为true，并使用reducer生成新的状态树。
    try {
      isDispatching = true
      currentState = currentReducer(currentState, action)
    } finally {
      // 当获取新的状态树完成后，设置状态为false.
      isDispatching = false
    }

    // 将目前最新的监听方法放置到即将执行的队列中遍历并且执行
    const listeners = (currentListeners = nextListeners)
    for (let i = 0; i < listeners.length; i++) {
      const listener = listeners[i]
      listener()
    }

    // 将触发的action返回
    return action
  }
```

其实看下来，我们发现他就是简单的将我们的参数转发给了`reducer`，然后执行了我们之前建立的监听队列。

有没有同学会和我想的一样，那为什么不直接执行`reducer`？因为我们里面有订阅函数需要执行，对吧，而且还有`dispatch`的状态需要通知`store`，不然`getState`之类的，我们之前有看到判断`dispatch`状态的很可能出现偏差。

我们只需要`connect`一下就可以直接在`props`里面拿到`dispatch`，不需要`import`之类的，很简洁。

#### replaceReducer

这个函数顾名思义，也很好理解，我就不多做解释了，直接看注释吧。

```js
// 这个其实很少用到, 官方的介绍是主要是用于动态替换reducer的时候会用到
  function replaceReducer<NewState, NewActions extends A>(
    nextReducer: Reducer<NewState, NewActions>
  ): Store<ExtendState<NewState, StateExt>, NewActions, StateExt, Ext> & Ext {
    if (typeof nextReducer !== 'function') {
      throw new Error('Expected the nextReducer to be a function.')
    }

    // 修改reducer
    // 当前的currentReducer更新为参数nextReducer
    // TODO: do this more elegantly
    ;((currentReducer as unknown) as Reducer<
      NewState,
      NewActions
    >) = nextReducer

    // This action has a similar effect to ActionTypes.INIT.
    // Any reducers that existed in both the new and old rootReducer
    // will receive the previous state. This effectively populates
    // the new state tree with any relevant data from the old one.
    // 和INIT的dispatch相同，发送一个dispatch初始化state，表明一下是REPLACE
    // 自己👀看一下utils方法的ActionTypes， 随性的随机数
    dispatch({ type: ActionTypes.REPLACE } as A)
    // change the type of the store by casting it to the new store
    return (store as unknown) as Store<
      ExtendState<NewState, StateExt>,
      NewActions,
      StateExt,
      Ext
    > &
      Ext
  }
```

发现两个很有趣的点：

1. 大佬们也会写`todo`，关键是！他们的`TODO`年头有点久了，哈哈哈哈，5年了，看样子不是只有我不会继续优化的，哈哈哈哈。也有可能是没有想到更好的方法。。。
2. 里面他自己`dispatch`了一个，`ActionTypes.REPLACE`。

ok，我们一探究竟，这个`ActionTypes.REPLACE`

扒开衣服！直接开看👀！

```js
/**
 * These are private action types reserved by Redux.
 * For any unknown actions, you must return the current state.
 * If the current state is undefined, you must return the initial state.
 * Do not reference these action types directly in your code.
 */

const randomString = () =>
  Math.random().toString(36).substring(7).split('').join('.')

const ActionTypes = {
  INIT: `@@redux/INIT${/* #__PURE__ */ randomString()}`,
  REPLACE: `@@redux/REPLACE${/* #__PURE__ */ randomString()}`,
  PROBE_UNKNOWN_ACTION: () => `@@redux/PROBE_UNKNOWN_ACTION${randomString()}`
}

export default ActionTypes

```

其实就是一个不想和我们的`type`冲突的随机数，告诉`redux`，这个是他们内部的一个`dispatch`而已。

#### observable

这个玩意，其实我是不太懂他是干什么的。。。全局搜也没有搜出来，而且我也没用过。。。

大家看注释吧，不过多解释，如果有懂哥，欢迎一起交流讨论呀。

```js
// 查询之后,也没发现有什么特别的用处...暂时跳过,如果有帅哥看到的话可以不吝赐教
  function observable() {
    const outerSubscribe = subscribe
    return {
      /**
       The minimal observable subscription method.
       @param observer Any object that can be used as an observer.
       The observer object should have a `next` method.
       @returns An object with an `unsubscribe` method that can
       be used to unsubscribe the observable from the store, and prevent further
       emission of values from the observable.
       */
      subscribe(observer: unknown) {
        if (typeof observer !== 'object' || observer === null) {
          throw new TypeError('Expected the observer to be an object.')
        }

        //获取观察着的状态
        function observeState() {
          const observerAsObserver = observer as Observer<S>
          if (observerAsObserver.next) {
            observerAsObserver.next(getState())
          }
        }

        observeState()
        //返回取消订阅的方法
        const unsubscribe = outerSubscribe(observeState)
        return { unsubscribe }
      },

      [$$observable]() {
        return this
      }
    }
  }
```



其实他内部还自己执行了一下`dispatch`，在`createStore`的最后。

```js
dispatch({ type: ActionTypes.INIT } as A)
```

其实也很好理解，我们`createStore`的时候，初始的`state`就是这样来的~

里面的`INIT`，在我们之前`REPLACE`里面也有介绍，也是一个随机数而已。

## applyMiddleware

这个坑，是之前在`createStore`的`ensureCanMutateNextListeners`埋下的，现在我们一起来看看这个。

这个redux拓展dispatch的唯一的标准的方法。

首先，我们得知道怎么用。我贴下我自己demo。

```js
// middleware
const logger = (store:any) => (next:any) => (action:any) => {
  // debugger
  console.info('dispatching', action)
  const result = next(action);
  console.log('next state', store.getState())
  return result
}
```

这个是官方的demo，不了解的，[点我快速了解](https://www.redux.org.cn/docs/advanced/Middleware.html)

ok，我们直接贴源码，很短，但是有很多奥妙（奥妙打钱！）。

```js
export default function applyMiddleware(
  ...middlewares: Middleware[]
): StoreEnhancer<any> {
  return (createStore: StoreEnhancerStoreCreator) => <S, A extends AnyAction>(
    reducer: Reducer<S, A>,
    preloadedState?: PreloadedState<S>
  ) => {
    const store = createStore(reducer, preloadedState)
    let dispatch: Dispatch = () => {
      throw new Error(
        'Dispatching while constructing your middleware is not allowed. ' +
          'Other middleware would not be applied to this dispatch.'
      )
    }

    const middlewareAPI: MiddlewareAPI = {
      getState: store.getState,
      dispatch: (action, ...args) => dispatch(action, ...args)
    }
    const chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose<typeof dispatch>(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}
```

看起来很短，但是却有很多深奥的地方。一个个来。

在开始之前，我们需要先了解一下函数柯里化。直接上demo帮助理解

```js
function test(a) {
  return function test2(b){
    console.log(a + b)
  }
}

const t = test(1);
t(2); // 3
```

这里就是一个简单的函数柯里化，我们可以先存储一下第一次传入的参数，后面如果这个`1`会复用的话，那么我们定义的变量`t`就可以在很多地方使用，里面用到了闭包的知识，而且是非常典型的闭包，存储形参`a`，实参`1`。

还记得我们的使用用例吗？

```js
// middleware
const logger = (store:any) => (next:any) => (action:any) => {
  // debugger
  console.info('dispatching', action)
  const result = next(action);
  console.log('next state', store.getState())
  return result
}
```

这里面第一个参数`store`就是对应到我们`createStore`源码里面我们挖的坑。

```js
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

    return enhancer(createStore)(
      reducer,
      preloadedState as PreloadedState<S>
    ) as Store<ExtendState<S, StateExt>, A, StateExt, Ext> & Ext
  }
```

第二个`next`，其实就是`middlewareAPI`里面的`dispatch`

```js
const middlewareAPI: MiddlewareAPI = {
      getState: store.getState,
      dispatch: (action, ...args) => dispatch(action, ...args),
      test:2,
    }
```

增加一个`log`在`middleware`里面就可以知道了

```js
const logger = (store:any) => (next:any) => (action:any) => {
  // debugger
  
  console.log(store)
  console.log('next===',next)
  console.log(action)
  console.info('dispatching', action)
  // const result = next(action);
  console.log('next state', store.getState())
  // return result
}
```

这个时候`log`的是我们在`createStore`里面的`dispatch`，还出现了我们的注释

说明`next`这个参数就是对应的`middlewareAPI`里面的`dispatch`也就是说是`createStore`里面的`dispatch`。



第三个参数`action`

很简单，其实就是普普通通的一个`action`。不过是从我们`middleaware`里面得到的。这个没有疑惑。



> 我们会发现一个很有趣的事情就是，如果我们注释掉调用`next(action)`，并且不将结果`return`的话，这个`action`就会被卡住，没有发出去，被我们打断了，所以这就是我们为什么需要做这个的原因。那么就可以引申出我们可以自定义打断某些情况下的action，然后需要在那些情况下给action里面加些什么，都可以办到。



ok，我们看了之后好像知道`applyMiddleware`里面的参数都对应这哪些东西了，但是似乎不知道为什么是一种柯里化的形式来展示的。我们看下面的代码就会知道了。

```js
// 调用每一个这样形式的middleware = store => next => action =>{}, 
// 组成一个这样[f(next)=>acticon=>next(action)...]的array，赋值给chain
const chain = middlewares.map(middleware => middleware(middlewareAPI))
// debugger
// compose看 -> compose.js文件
// compose(...chain)会形成一个调用链, next指代下一个函数的注册, 这就是中间件的返回值要是next(action)的原因
// 如果执行到了最后next就是原生的store.dispatch方法
dispatch = compose(...chain)(store.dispatch) 
```

里面提到了`compose`这个函数，其实这个函数就是整个中间件的精髓所在。柯里化的奥义！

```js
export default function compose(...funcs: Function[]) {
  if (funcs.length === 0) {
    // infer the argument type so it is usable in inference down the line
    return <T>(arg: T) => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce((a, b) => (...args: any) => a(b(...args)))
}
```

不了解reduce的同学可以[点这里](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)。

这个compose其实是一个调用链，我们的所有的中间件都会按顺序加载，然后是👇这种形式

```js
middleware = middleware1 => middleware2 => middleware3 => middleware4 //...
```

我们通过一下代码知道，确实是往下传递的。

```js
// middleware
const logger = (store:any) => (next:any) => (action:any) => {
  // debugger
  
  console.log(store)
  console.log('next===',next)
  console.log(action)
  console.info('dispatching', action)
  action.aa = 2
  const result = next(action);
  console.log('next state', store.getState())
  return result
}

const logger2 = (store:any) => (next:any) => (action:any) => {
  // debugger
  
  console.log(store)
  console.log('next2===',next)
  console.log('logger2==-=-=-=-=-=-=',action)
  console.info('dispatching', action)
  const result = next(action);
  console.log('next state', store.getState())
  return result
}

const store = createStore(rootReducer, {todos:[]}, applyMiddleware(logger, logger2));
```

在`logger2`里面确实打印出来`aa`这个属性的值，而`logger`却没有



## compose

这个在上面介绍applyMiddleware的时候说过了。就不多说了。



## combineReducers

这个中文翻译就是《组合reducer》。

其实就是 字面意思，我们看一下怎么用就了解了。

```js
export default combineReducers({
  firstReducer: todos,
  secondReducer: visibilityFilter
})
```

然后我们看一下`log`出来的`props`是什么样子的。

```js
{
	firstReducer: {todos: Array(0)}
	secondReducer: "SHOW_ALL"
}
```

也就是我们一开始设置进去的`key`值，返回的时候也是按这个格式返回的。

里面有许多函数，我们先看他到处的函数`combineReducers`

### combineReducers

```js
// 用于合并reducer 一般是这样combineReducers({a,b,c})
export default function combineReducers(reducers: ReducersMapObject) {
  // reducers中key的数组
  const reducerKeys = Object.keys(reducers)
  // 最终的reducer
  const finalReducers: ReducersMapObject = {}

  // 循环， 目的为了给finalReducers赋值， 过虑了不符合规范的reducer
  for (let i = 0; i < reducerKeys.length; i++) {
    // 接受当前的key
    const key = reducerKeys[i]

    // 如果不是生产环境， 当前的reducer是undefined会给出warning
    if (process.env.NODE_ENV !== 'production') {
      if (typeof reducers[key] === 'undefined') {
        warning(`No reducer provided for key "${key}"`)
      }
    }

    // reducer要是一个function
    if (typeof reducers[key] === 'function') {
      // 赋值给finalReducers
      finalReducers[key] = reducers[key]
    }
  }
  // 符合规范的reducer的key数组
  const finalReducerKeys = Object.keys(finalReducers)

  // This is used to make sure we don't warn about the same
  // keys multiple times.
  // 意想不到的key， 先往下看看
  let unexpectedKeyCache: { [key: string]: true }
  // production环境为{}
  if (process.env.NODE_ENV !== 'production') {
    unexpectedKeyCache = {}
  }

  let shapeAssertionError: Error
  try {
    // 看这个function
    assertReducerShape(finalReducers)
  } catch (e) {
    shapeAssertionError = e
  }

  // 返回function， 即为createStore中的reducer参数既currentReducer
  // 自然有state和action两个参数， 可以回createStore文件看看currentReducer(currentState, action)
  return function combination(
    state: StateFromReducersMapObject<typeof reducers> = {},
    action: AnyAction
  ) {
    // reducer不规范报错
    if (shapeAssertionError) {
      throw shapeAssertionError
    }

    // 比较细致的❌信息，顺便看了一下getUndefinedStateErrorMessage，都是用于提示warning和error的， 不过多解释了
    if (process.env.NODE_ENV !== 'production') {
      const warningMessage = getUnexpectedStateShapeWarningMessage(
        state,
        finalReducers,
        action,
        unexpectedKeyCache
      )
      if (warningMessage) {
        warning(warningMessage)
      }
    }

    let hasChanged = false
    const nextState: StateFromReducersMapObject<typeof reducers> = {}
    for (let i = 0; i < finalReducerKeys.length; i++) {

      // 获取finalReducerKeys的key和value（function）
      const key = finalReducerKeys[i]
      const reducer = finalReducers[key]
      // 当前key的state值
      const previousStateForKey = state[key]
      // 执行reducer， 返回当前state
      const nextStateForKey = reducer(previousStateForKey, action)
      // 不存在返回值报错
      if (typeof nextStateForKey === 'undefined') {
        const errorMessage = getUndefinedStateErrorMessage(key, action)
        throw new Error(errorMessage)
      }
      // 新的state放在nextState对应的key里
      nextState[key] = nextStateForKey
      // 判断新的state是不是同一引用， 以检验reducer是不是纯函数
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }
    hasChanged =
      hasChanged || finalReducerKeys.length !== Object.keys(state).length
      // 改变了返回nextState
    return hasChanged ? nextState : state
  }
  /*
  *  新版本的redux这部分改变了实现方法
  *  老版本的redux使用的reduce函数实现的
  *  简单例子如下
  * function combineReducers(reducers) {
  *    return (state = {}, action) => {
  *        return Object.keys(reducers).reduce((currentState, key) => {
  *            currentState[key] = reducers[key](state[key], action);
  *             return currentState;
  *         }, {})
  *      };
  *    }
  * 
  * */
}
```

上面是主要源码，一个个剖析进去。

先是对数据进行清洗，第一次筛选出符合要求的`reducer`们

```js
// reducers中key的数组
  const reducerKeys = Object.keys(reducers)
  // 最终的reducer
  const finalReducers: ReducersMapObject = {}

  // 循环， 目的为了给finalReducers赋值， 过虑了不符合规范的reducer
  for (let i = 0; i < reducerKeys.length; i++) {
    // 接受当前的key
    const key = reducerKeys[i]

    // 如果不是生产环境， 当前的reducer是undefined会给出warning
    if (process.env.NODE_ENV !== 'production') {
      if (typeof reducers[key] === 'undefined') {
        warning(`No reducer provided for key "${key}"`)
      }
    }

    // reducer要是一个function
    if (typeof reducers[key] === 'function') {
      // 赋值给finalReducers
      finalReducers[key] = reducers[key]
    }
  }
```



### assertReducerShape

初筛之后的finalReducers会交由assertReducerShape再一次进行检验。里面就是一个判断，然后给出警告⚠️之类的。就不多说了。

```js
function assertReducerShape(reducers) {
  Object.keys(reducers).forEach(key => {
    const reducer = reducers[key]
   // reducer返回值
    const initialState = reducer(undefined, { type: ActionTypes.INIT })
    // undefined throw Error
    if (typeof initialState === 'undefined') {
      throw new Error(
        `Reducer "${key}" returned undefined during initialization. ` +
          `If the state passed to the reducer is undefined, you must ` +
          `explicitly return the initial state. The initial state may ` +
          `not be undefined. If you don't want to set a value for this reducer, ` +
          `you can use null instead of undefined.`
      )
    }

    // 很明显assertReducerShape是用于reducer的规范
    // 回到combineReducers
    if (
      typeof reducer(undefined, {
        type: ActionTypes.PROBE_UNKNOWN_ACTION()
      }) === 'undefined'
    ) {
      throw new Error(
        `Reducer "${key}" returned undefined when probed with a random type. ` +
          `Don't try to handle ${
            ActionTypes.INIT
          } or other actions in "redux/*" ` +
          `namespace. They are considered private. Instead, you must return the ` +
          `current state for any unknown actions, unless it is undefined, ` +
          `in which case you must return the initial state, regardless of the ` +
          `action type. The initial state may not be undefined, but can be null.`
      )
    }
  })
}
```



### combination

然后就是返回combination这个函数。参数的意义就是他的名字。就不多说了。

```js
return function combination(state = {}, action) {
    // reducer不规范报错
    if (shapeAssertionError) {
      throw shapeAssertionError
    }

    // 比较细致的❌信息，顺便看了一下getUndefinedStateErrorMessage，都是用于提示warning和error的， 不过多解释了
    if (process.env.NODE_ENV !== 'production') {
      const warningMessage = getUnexpectedStateShapeWarningMessage(
        state,
        finalReducers,
        action,
        unexpectedKeyCache
      )
      if (warningMessage) {
        warning(warningMessage)
      }
    }

    let hasChanged = false
    const nextState = {}
    for (let i = 0; i < finalReducerKeys.length; i++) {
      // 获取finalReducerKeys的key和value（function）
      const key = finalReducerKeys[i]
      const reducer = finalReducers[key]
      // 当前key的state值
      const previousStateForKey = state[key]
      // 执行reducer， 返回当前state
      const nextStateForKey = reducer(previousStateForKey, action)
      // 不存在返回值报错
      if (typeof nextStateForKey === 'undefined') {
        const errorMessage = getUndefinedStateErrorMessage(key, action)
        throw new Error(errorMessage)
      }
      // 新的state放在nextState对应的key里
      nextState[key] = nextStateForKey
      // 判断新的state是不是同一引用， 以检验reducer是不是纯函数
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }
    // 改变了返回nextState
    return hasChanged ? nextState : state
  }
  /*
  *  新版本的redux这部分改变了实现方法
  *  老版本的redux使用的reduce函数实现的
  *  简单例子如下
  * function combineReducers(reducers) {
  *    return (state = {}, action) => {
  *        return Object.keys(reducers).reduce((currentState, key) => {
  *            currentState[key] = reducers[key](state[key], action);
  *             return currentState;
  *         }, {})
  *      };
  *    }
  * 
  * */
```



其实就是执行reducer，然后把值拿出来比对，如果是引用的话就返回之前的state，如果不是就使用新的。他的比对和之前说的数组比对一样，都是比对内存地址。

为什么要这么做呢？

好问题！

这样做的好处其实就是为了纯函数，为了一切可预测。而且简单的比较内存地址比递归的性能好太多了。

那么我们怎么样才可以契合这样的理念呢？

就是我们在写reducer返回的时候，需要返回一个全新的对象，通常是

```js
return{
	...state,
	xx: xx
}
```

类似这样。这样的话js就会新开辟一个内存地址出来。在比对的时候内存地址就不会相同了。



## bindActionCreators

这个函数其实非常少用到，而且也没有什么特别晦涩的东西，只是一个封装。但是是一种更为优雅的方式。

直接看代码，很少。

```js
export default function bindActionCreators(actionCreators, dispatch) {
  // actionCreators为function
  if (typeof actionCreators === 'function') {
    return bindActionCreator(actionCreators, dispatch)
  }

  // 不是object throw Error
  if (typeof actionCreators !== 'object' || actionCreators === null) {
    throw new Error(
      `bindActionCreators expected an object or a function, instead received ${
        actionCreators === null ? 'null' : typeof actionCreators
      }. ` +
        `Did you write "import ActionCreators from" instead of "import * as ActionCreators from"?`
    )
  }

  // object 转为数组
  const keys = Object.keys(actionCreators)
  // 定义return 的props
  const boundActionCreators = {}
  for (let i = 0; i < keys.length; i++) {
    // actionCreators的key 通常为actionCreators function的name（方法名）
    const key = keys[i]
    // function => actionCreators工厂方法本身
    const actionCreator = actionCreators[key]
    if (typeof actionCreator === 'function') {
      // 参数为{actions：function xxx}是返回相同的类型
      boundActionCreators[key] = bindActionCreator(actionCreator, dispatch)
    }
  }
  // return 的props
  return boundActionCreators
}
```

代码很简单也很清晰，就是弄一个新的对象，然后把`actionCreators`里面的`value`变成`bindActionCreator(actionCreator, dispatch)`的值，那么一起看一下`bindActionCreator`

```js
function bindActionCreator<A extends AnyAction = AnyAction>(
  actionCreator: ActionCreator<A>,
  dispatch: Dispatch
) {
  return function (this: any, ...args: any[]) {
    return dispatch(actionCreator.apply(this, args))
  }
}
```

其实这里看起来有点迷糊，看也是能看懂，其实就是返回一个方法，这个方法是干什么的呢？返回执行dispatch之后的返回值。这么搞这么一圈究竟有什么秘密呢？

我们写一个demo就会知道了。

假如我们要做一个动态的增加的action，分别为点击一次+1，点击一次+2，点击一次+3。

我们需要

```js
const counterIncActionCreator = function(step) {
  return {
    type: 'INCREMENT',
    step: step || 1
  }
}

// 为了简化代码我把dispatch函数定义为只有打印功能的函数
const dispatch = function(action) {
  console.log(action)
}

const action1 = counterIncActionCreator()
dispatch(action1) // { type: 'INCREMENT', step: 1 }

const action2 = counterIncActionCreator(2)
dispatch(action2) // { type: 'INCREMENT', step: 2 }

const action3 = counterIncActionCreator(3)
dispatch(action3) // { type: 'INCREMENT', step: 3 }

```

有点难受其实，但是我们用了`bindActionCreator`之后

```js
const increment = bindActionCreator(counterIncActionCreator, dispatch)

increment() // { type: 'INCREMENT', step: 1 }

increment(2) // { type: 'INCREMENT', step: 2 }

increment(3) // { type: 'INCREMENT', step: 3 }

```

就整个帅起来，有没有？！更加优雅了。

而且我们还可以制作增和减的两个工厂🏭

```js
const MyActionCreators = {
  increment: function(step) {
    return {
      type: 'INCREMENT',
      step: step || 1
    }
  },

  decrement: function(step) {
    return {
      type: 'DECREMENT',
      step: - (step || 1)
    }
  }
}

```



老方法

```js
// 原始的调度方式
dispatch(MyActionCreators.increment()) // { type: 'INCREMENT', step: 1 }
dispatch(MyActionCreators.increment(2)) // { type: 'INCREMENT', step: 2 }
dispatch(MyActionCreators.increment(3)) // { type: 'INCREMENT', step: 3 }
dispatch(MyActionCreators.decrement()) // { type: 'DECREMENT', step: -1 }
dispatch(MyActionCreators.decrement(2)) // { type: 'DECREMENT', step: -2 }
dispatch(MyActionCreators.decrement(3)) // { type: 'DECREMENT', step: -3 }

```

进化之后

```js
MyNewActionCreators.increment() // { type: 'INCREMENT', step: 1 }
MyNewActionCreators.increment(2) // { type: 'INCREMENT', step: 2 }
MyNewActionCreators.increment(3) // { type: 'INCREMENT', step: 3 }
MyNewActionCreators.decrement() // { type: 'DECREMENT', step: -1 }
MyNewActionCreators.decrement(2) // { type: 'DECREMENT', step: -2 }
MyNewActionCreators.decrement(3) // { type: 'DECREMENT', step: -3 }

```



## Redux总结

至此我们的redux源码阅读就到此为止。

接下来我们一起学习一下和redux配合使用的react-redux

## react-redux

React 原本和redux没有关系，redux只是负责数据处理，理论上无论是vue或者angular都可以使用。

但是react-redux将react和redux紧密的联系在了一起。用更符合react的方式来调用redux。

后续会出一篇介绍react-redux的文章。

# 总结

自此redux源码阅读算是告一段落了。其实源码很简单，但是思想却很高级。

如果有不懂的，或者哪里说错了，小伙伴欢迎在评论区讨论。

# 索引

>https://juejin.im/post/6844903638377185288
>
>https://www.redux.org.cn/





> 实在是没有地方吐槽，我在这里说一下最近的事情吧。先是部门黄了，然后大领导换人了，大领导不知道是不是新官上任三把火，非要搞什么提效，其实就是把人弄走，然后加大工作量，刚刚开始换组我就去写很老的vue项目，没有鄙视vue的意思，是因为我之前都在写react，然后一下让我写，也不太适应而已，还是很少的版本。那就写就写吧，反正做前端的都得学不是？可是，大领导为了给做给老板看，然后让我们下班不要走，就是没有事情做就干坐着的那种。本着90后的性子我就走了，然后我走了之后，有些小伙伴可能也开始走了，然后我就被抓去劝退了....我也没有死皮赖脸的赖着。借口也非常的奇葩"你活在自己的世界里面，没有关心周围同事忙不忙，改bug不积极，代码质量低"之类的。。。可是我只是工作的时候戴着而已，因为经常有产品上来就在旁边开始讨论了，然后我自己的工作都做不完，怎么关心同事啊。。强行加了工作量之后我都很晚走或者回家工作了。因为在公司赖着真的很难受，为什么不在自己舒适的家中工作呢？还可以马上睡觉那种，音乐还可以外放。。。再说了我之前的代码，因为项目黄了，应该没人看了吧....唉，其实就是 劝退。我都懂。
>
> 但是我的简历其实变得极其丑陋了，因为我年初的才跳槽过来的。现在这个时间非常尴尬....而且我来的第一天听说要996，马上提了离职申请，我的领导同意了，可以之前的大领导和我说以后不会996了就是正常上下班。。然后他就跳槽了。。。
>
> 算是自己不认真审查公司，不过他们也太骗人了吧...唉。
>
> 现在在家里也很焦虑，不知道找不找得到工作，杭州的公司看了一圈都是996，学阿里的加班不学阿里的工资。。。
>
> 国庆以后再说吧，希望早点把重识前端系列更完。

