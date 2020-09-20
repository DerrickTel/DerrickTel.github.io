---
title: 【重识前端】redux源码阅读
date: 2020-09-11 16:10:35
tags: [源码阅读, React]
category: [重拾前端]
cover: /image/cover/web.jpeg
---

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