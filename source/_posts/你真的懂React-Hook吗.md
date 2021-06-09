---
title: 你真的懂React Hook吗？
date: 2020-05-20 00:19:44
tags: [React, ReactHook]
category: [React]
cover: /image/cover/React.jpeg
---
## 前言

1. 读这篇文章的前提是你已经对React Hook有所了解的情况下，如果你还没有了解，请先移步官网学习一下。
   1. 最好不要去网上看别人的总结之类的，无非就是超的官网的，而且这样会让你的认知从一开始就走偏。
2. 这篇文章主要是探究Hook的动机，使用中的一些疑问；
   1. 使用的话React官网已经讲得很详细了，这里就不多赘述了。
3. 有需要看接下来的疑难点的伙伴欢迎直接跳过探究直接看具体的疑问；

## 探究

主要从3个方面研究React Hook

根据黄金思维圈（What、How、Why）

### What

什么是Hook？

打开Google翻译，得到的解释：*钩、钩子*

> 再看看React官网的解释：They let you use state and other React features without writing a class.（它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性。）

所以，结合一下。我个人的理解是这样的：对于函数式的组件，可以用钩子（Hook）将想要的外部功能给“钩”进来。

在React Hook出来之前，函数式组件都是无状态的组件，最多就是根据`props`来加一些判断的逻辑；而在React Hook出来之后就可以在函数式组件里面加入状态（useState），类生命周期（useEffect），甚至是一些自己的复用逻辑（自定义Hook）等等这些外部的功能。



### How

怎么使用Hook？

大家一起看一下官网的一个例子。

题目：显示一个计数器。当你点击按钮，计数器的值就会增加。

#### Class组件



```javascript
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```



#### React Hook



```javascript
import React, { useState } from 'react';

function Example() {
  // 声明一个叫 "count" 的 state 变量  const [count, setCount] = useState(0);
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```



这样就算是完成了一个最简单的React Hook 实践，关于一些官方提供的Hook晚点会介绍。



### Why

- 为什么会有Hooks？
- Hook能解决什么问题？

做任何一件事情我觉得都应该理清这个两个问题，这样的话就会事半功倍。

我们先看看React官方是怎么解释“Why”的

> 1. 在组件之间复用状态逻辑很难
> 2. 复杂组件变得难以理解
> 3. 难以理解的 class

本人个人认为第三点是来凑数的....

为什么这么说？

因为React用了这么久了基本都是在使用Class组件，这个是在之前，哪怕是现在学习React的必经之路吧！所以，这点我接下来就会跳过了😂

### 在组件之间服用状态逻辑很难

其实高阶组件或者说是props都是很好的解决了复杂的聚合业务逻辑，那为什么说在**组件之间服用状态逻辑很难**呢？

其实道理非常简单。

举个简单的例子，方便大家理解。

场景：有 请求A，请求B，请求C，请求D。他们的请求都有相互依赖关系比如，发请求B的时候必须拿到请求A的结果中的某个值，而请求C也必须拿到请求B的结果中的某个值。以此类推请求D。

Promise出来之前是怎么做的呢？

```javascript
$.ajax({
    type:"post",
    success: function(){//成功回调
        //再次异步请求
        $.ajax({
            type:"post",
            url:"...",
            success:function(){//成功回调
              //再次异步请求
              $.ajax({
                  type:"post",
                  url:"...",
                  success:function(){
                      .......//如此循环
                  }
              })
          }
        })
    }
})
```

这还只是3层，如果是100层呢？那看起来就非常的难受了！

Promise较好的解决了这个问题

```javascript
new Promise(f1)
 .then(f2)
 .then(f3)
 .then(f4)
 .then(f5)
 .then(f5)
…………
```

然后是async/await。这里就不展开了，有兴趣的可以自己去了解一下。

结论

之所以这么大费周章的讲是为了解释，React中的高阶组件（HOC）。他的逻辑其实和回调地狱类似，一个两个其实都还算优雅或者说舒服，一旦多了的话。。。

```javascript
export default withHover(
  withTheme(
    withAuth(
      withRepos(Profile)
    )
  )
)

// 就会变成这样，不够优雅
<WithHover>
  <WithTheme hovering={false}>
    <WithAuth hovering={false} theme='dark'>
      <WithRepos hovering={false} theme='dark' authed={true}>
        <Profile 
          id='JavaScript'
          loading={true} 
          repos={[]}
          authed={true}
          theme='dark'
          hovering={false}
        />
      </WithRepos>
    </WithAuth>
  <WithTheme>
</WithHover>
```

而且每个高阶组件的逻辑复用我们可能还要一个个去研读。



### 复杂组件变得难以理解

其实，这点非常好理解。举一个非常简单常见的例子大家就会明白了。

场景：假如我有一个子组件Child，他的功能是这样的：父组建会给一个id，在组件创建的时候获取一下有关信息，在id改变的时候再重新获取。

#### Class组件

```javascript
componentDidMount () {
    this.fetch(this.props.id)
 }
componentDidUpdate (prevProps) {
  if (prevProps.id !== this.props.id) {
    this.fetch(this.props.id)
  }
}
fetch = id => {
  this.setState({ loading: true })
  fetchInfo(id)
    .then(info => this.setState({
    info,
    loading: false
  }))
}
```

#### React Hook：

```javascript
const fetch = id => {
  this.setState({ loading: true })
  fetchInfo(id)
    .then(info => this.setState({
    info,
    loading: false
  }))
}
useEffect(() => {
  fetch(this.props.id)
}, [this.props.id])
```



### 结论

简单的说一下他的优点吧。

1. 复用代码更加简单（需要什么就“钩”进来）
2. 清爽的代码风格，一目了然。（useState支持数组和对象，可以清晰的定义特殊的字段等等）
3. 代码量更少（可以看一下我之前的子父组建的例子）
4. 更愿意去写一些小组件复用（我个人喜欢React就是因为他的组件写起来非常的顺手！ps：没有贬低其他框架的意思。。）
5. 其实我个人认为React Hook在宣扬一个观念“按需加载”。

## useState

### 使用

简单的使用在上面的探究-How里面有介绍，更多的在React官网也有介绍。

#### 请回答以下代码的运行结果

![](https://overreacted.io/46c55d5f1f749462b7a173f1e748e41e/counter.gif)

```javascript
function Counter() {
  const [count, setCount] = useState(0);

  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
      <button onClick={handleAlertClick}>
        Show alert
      </button>
    </div>
  );
}
```

你猜alert会弹出什么呢？会是5吗？— 这个值是alert的时候counter的实时状态。或者会是3吗？— 这个值是我点击时候的状态。

> 分割线

来自己 [试试吧！](https://codesandbox.io/s/w2wxl3yo0l)

#### 答案是

3

这是为什么呢？function组建究竟是如果工作的呢？

我们发现`count`在每一次函数调用中都是一个常量值。值得强调的是 — **我们的组件函数每次渲染都会被调用，但是每一次调用中`count`值都是常量，并且它被赋予了当前渲染中的状态值。**

这并不是React特有的，普通的函数也有类似的行为：

```jsx
function sayHi(person) {
  const name = person.name;  setTimeout(() => {
    alert('Hello, ' + name);
  }, 3000);
}

let someone = {name: 'Dan'};
sayHi(someone);

someone = {name: 'Yuzhi'};
sayHi(someone);

someone = {name: 'Dominic'};
sayHi(someone);
```

在 [这个例子](https://codesandbox.io/s/mm6ww11lk8)中, 外层的`someone`会被赋值很多次（就像在React中，*当前*的组件状态会改变一样）。**然后，在`sayHi`函数中，局部常量`name`会和某次调用中的`person`关联。**因为这个常量是局部的，所以每一次调用都是相互独立的。结果就是，当定时器回调触发的时候，每一个alert都会弹出它拥有的`name`。

这就解释了我们的事件处理函数如何捕获了点击时候的`count`值。如果我们应用相同的替换原理，每一次渲染“看到”的是它自己的`count`：

```jsx
// During first render
function Counter() {
  const count = 0; // Returned by useState()  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }
  // ...
}

// After a click, our function is called again
function Counter() {
  const count = 1; // Returned by useState()  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }
  // ...
}

// After another click, our function is called again
function Counter() {
  const count = 2; // Returned by useState()  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }
  // ...
}
```

所以实际上，每一次渲染都有一个“新版本”的`handleAlertClick`。每一个版本的`handleAlertClick`“记住” 了它自己的 `count`：

```jsx
// During first render
function Counter() {
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + 0);    }, 3000);
  }
  // ...
  <button onClick={handleAlertClick} /> // The one with 0 inside  // ...
}

// After a click, our function is called again
function Counter() {
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + 1);    }, 3000);
  }
  // ...
  <button onClick={handleAlertClick} /> // The one with 1 inside  // ...
}

// After another click, our function is called again
function Counter() {
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + 2);    }, 3000);
  }
  // ...
  <button onClick={handleAlertClick} /> // The one with 2 inside  // ...
}
```

这就是为什么[在这个demo中](https://codesandbox.io/s/w2wxl3yo0l)中，事件处理函数“属于”某一次特定的渲染，当你点击的时候，它会使用那次渲染中`counter`的状态值。

**在任意一次渲染中，props和state是始终保持不变的。**如果props和state在不同的渲染中是相互独立的，那么使用到它们的任何值也是独立的（包括事件处理函数）。它们都“属于”一次特定的渲染。即便是事件处理中的异步函数调用“看到”的也是这次渲染中的`count`值。

#### 请回答以下代码的运行结果

```javascript
function Counter() {
  const [count, setCount] = useState(0);

  const addCount = () => {
    setCount(count+1)
    setCount(count+2)
    setCount(count+3)
    setCount(count+4)
    setCount(count+5)
  }

  console.log(count)
  
  return (
    <div>
      <button onClick={addCount}>
        Click me
      </button>
    </div>
  );
}
```

> 分割线

#### 答案是

5

为什么呢？

useState的更新究竟是如何工作的呢？

我们进入`ReactHooks.js`来看看，发现`useState`的实现竟然异常简单，只有短短两行

```js
// ReactHooks.js
export function useState<S>(initialState: (() => S) | S) {
  const dispatcher = resolveDispatcher();
  return dispatcher.useState(initialState);
}
```

其实可以这样理解useState，useState其实就是useReducer的一个语法糖；但是这个不在这个问题的讨论范围内；

好，收回来。

其实我们在`const [xx, setXx] = useState(xx)`的时候就生成一个队列，我们暂时叫它为queue；所有这一轮运行读取到的state都被放到一个链表的队列里面去，然后再用do-while循环，每次都是拿到最新的值，但是不是Object.assgin的形式，而是直接赋值。话不多说直接源码。

```js
function updateReducer(reducer, initialArg, init) {
// 获取初始化时的 hook
  const hook = updateWorkInProgressHook();
  const queue = hook.queue;

  // 开始渲染更新
  if (numberOfReRenders > 0) {
    const dispatch = queue.dispatch;
    if (renderPhaseUpdates !== null) {
      // 获取Hook对象上的 queue，内部存有本次更新的一系列数据
      const firstRenderPhaseUpdate = renderPhaseUpdates.get(queue);
      if (firstRenderPhaseUpdate !== undefined) {
        renderPhaseUpdates.delete(queue);
        let newState = hook.memoizedState;
        let update = firstRenderPhaseUpdate;
        // 获取更新后的state
        do {
          const action = update.action;
          // 此时的reducer是basicStateReducer，直接返回action的值
          // 注意，这里是等于号所以
          /**
          *
          * setObj({ a: 1, b: 1, c: 1 })
        	* setObj({ a: 2, b: 2 })
        	* setObj({ a: 3 })
        	*
        	* 到最后也只有只有{ a: 3 }，而b和c全没了
          *
          **/
          newState = reducer(newState, action);
          update = update.next;
        } while (update !== null);
        // 对 更新hook.memoized 
        hook.memoizedState = newState;
        // 返回新的 state，及更新 hook 的 dispatch 方法
        return [newState, dispatch];
      }
    }
  }
```



## useEffect/useLayoutEffect

### 提示

在学习useEffect这个Hook的时候，淡化你知道的“生命周期”这个概念。

#### useEffect和useLayoutEffect两兄弟的区别是什么？

执行的时机不同

那么具体哪里不同呢？

其实在初始化useEffect和useLayoutEffect是没有区别的，他们真正的区别在于初始化之后；

举个非常形象的例子🌰：

除了初始化之后的一轮更新：

> 浏览器：我要绘制了！
>
> React：等等，我有一个哥们临时有事要处理，他是：useLayoutEffect
>
> useLayoutEffect执行....
>
> React：好了，你可以开始绘制了～@浏览器
>
> 浏览器：好的
>
> 浏览器更新UI...
>
> 浏览器：我更新好了。你有什么事要做的吗？@React
>
> React：有的，useEffect你上
>
> useEffect执行....

可能有点废话了。其实区别就是

> useLayoutEffect()
>
> 浏览器绘制
>
> useEffect()

这样其实大家也能很直接的看到弊端了。那就是useLayoutEffect如果有大量的计算的话，那样可能会阻塞UI更新，或者说UI渲染。所以还是要谨慎使用。

一般来说他们没有什么太大的区别的，如果真的要使用useLayoutEffect的话要谨慎一些。不然可能会导致UI渲染阻塞之类的问题。

但是，也不是没有使用场景。

比如下面的这个代码就很需要useLayoutEffect

```js
function App() {
  const [count, setCount] = useState(0);
  
  useLayoutEffect(() => {
    if (count === 0) {
      const randomNum = 10 + Math.random()*200
      setCount(10 + Math.random()*200);
    }
  }, [count]);

  return (
      <div onClick={() => setCount(0)}>{count}</div>
  );
}

//   我是分割线

function App() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    if (count === 0) {
      const randomNum = 10 + Math.random()*200
      setCount(10 + Math.random()*200);
    }
  }, [count]);

  return (
      <div onClick={() => setCount(0)}>{count}</div>
  );
}
```

其实明白的同学一下就看出来了，如果使用useEffect的话会出现闪烁，会先回到0然后再更新新的随机数。而反观useLayoutEffect则不会，他会很自然的过渡。

总结：

useLayoutEffect的使用场景为：有一个中间状态希望隐藏的时候再使用。

大部分情况下useEffect可以适用于99%的场景。

####  useEffect的错误事例。看看有没有你

```js
function SearchResults() {
  const [query, setQuery] = useState('react');

  // Imagine this function is also long
  function getFetchUrl() {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }

  // Imagine this function is also long
  async function fetchData() {
    const result = await axios(getFetchUrl());
    setData(result.data);
  }

  useEffect(() => {
    fetchData();
  }, []);

  // ...
}
```

#### 为什么错了？

不难看出上面代码的意思是。想要模仿componentDidMount的生命周期，在页面或者组件加载之后发送一个请求。咋一看好像没有什么问题（实际在运行的过程中也没有什么问题，在写这篇文章之前我也是这么做的。）

但是大家可以想象一下，如果这个函数组件，是现在的5倍大，这个didMount里面调用的请求，未来依赖的东西你都可以100%的察觉到吗？

我觉得难！难免会有疏忽。到时候可能就会出现state或者props读取错误的情况。因为每一次render的state和props都是独立的。

那么，该如何解决呢？

有一个很土的办法，直接把函数扔到useEffect里面去

```jsx
function SearchResults() {
  // ...
  useEffect(() => {
    // We moved these functions inside!    
    function getFetchUrl() {
      return 'https://hn.algolia.com/api/v1/search?query=react';
    }
    async function fetchData() {
      const result = await axios(getFetchUrl());
      setData(result.data);
    }
    fetchData();
  }, []); // ✅ Deps are OK
  // ...
}
```

那高级点的办法呢？

```js
function SearchResults() {
  // ✅ Preserves identity when its own deps are the same
  const getFetchUrl = useCallback((query) => {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }, []);  // ✅ Callback deps are OK

  useEffect(() => {
    const url = getFetchUrl('react');
    // ... Fetch data and do something ...
  }, [getFetchUrl]); // ✅ Effect deps are OK

  // ...
}
```

将函数用useCallback包裹，这样的话我们只需要做useEffect的依赖里面写上我们的函数，然后在useCallback里面写上我们的依赖。

#### 我们都知道，每一次effect都是全新的state和props，那我要如何获得上一轮更新的state呢？



```js
function Counter() {
  const [count, setCount] = useState(0);

  const prevCountRef = useRef();
  useEffect(() => {
    prevCountRef.current = count;
  });
  const prevCount = prevCountRef.current;

  return <h1>Now: {count}, before: {prevCount}</h1>;
}
```

其实很好理解，如果看了前面useEffect和useLayoutEffect区别的同学一下就可以知道这个的实现原理。

首先，在一切都更新之后，然后会会执行useEffect内部的回调函数，将prevCount给赋值，由于没有触发渲染，所以只是单纯的赋值。这样就看起来prevCount的值永远都慢一步。

### 总结

其实在学习useEffect的时候。应该忘记你对React的一些知识。比如生命周期，在函数组件里面没有生命周期这个概念了。

每一次的render他都有自己的state和props。state和props更应该被看作一个常量，哪怕是const bar = xx这样的常量。这样理解起来useEffect这个副作用其实会更加顺畅，也不容易进入他的“陷阱”

## useRef

### 前言

为什么我把useRef单独拎出来说，不把他和`useImperativeHandle`放在一起讲，因为

> （官网原话）它创建的是一个普通 Javascript 对象。而 `useRef()` 和自建一个 `{current: ...}` 对象的唯一区别是，`useRef` 会在每次渲染时返回同一个 ref 对象。

记住`useRef`不单单用于获取`DOM节点和组件实例`，还有一个巧妙的用法就是`作为容器保留可变变量`，可以这样说：`无法自如地使用useRef会让你失去hook将近一半的能力`

#### useRef 与 createRef 的区别

`useRef` 仅能用在 FunctionComponent，`createRef` 仅能用在 ClassComponent。

`useRef` 仅能用在 FunctionComponent，`createRef` 仅能用在 ClassComponent。

第一句话是显然的，因为 Hooks 不能用在 ClassComponent。

第二句话的原因是，`createRef` 并没有 Hooks 的效果，其值会随着 FunctionComponent 重复执行而不断被初始化：

```
function App() {
  // 错误用法，永远也拿不到 ref
  const valueRef = React.createRef();
  return <div ref={valueRef} />;
}
复制代码
```

上述 `valueRef` 会随着 App 函数的 Render 而重复初始化，**这也是 Hooks 的独特之处，虽然用在普通函数中，但在 React 引擎中会得到超出普通函数的表现，比如初始化仅执行一次，或者引用不变**。

为什么 `createRef` 可以在 ClassComponent 正常运行呢？这是因为 ClassComponent 分离了生命周期，使例如 `componentDidMount` 等初始化时机仅执行一次。

#### 如何解决每次render带来类闭包问题？

首先，题目怎么理解？[题目](https://codesandbox.io/s/w2wxl3yo0l)

如果我们希望他alert的时候可以获取到最新的值的话，可以使用useRef来解决

```js
const Counter = () => {
  const [count, setCount] = useState<number>(0)
  const countRef = useRef<number>(count)

  useEffect(() => {
    countRef.current = count
  })

  const handleCount = () => {
    setTimeout(() => {
      alert('current count: ' + countRef.current)
    }, 3000);
  }

  //...
}

export default Counter
```



## memo

#### memo没有回调函数的话是怎么浅比较的？

先来看看memo没有回调函数的时候他做了什么。

memo它是一个高阶组件（HOC）他与React.PureComponent十分相似。除了使用的地方不同（Class组件和Function组件）之外几乎一致。

Memo内部和PureComponent一样使用Object.is用于前对比，如果传入的props内存地址不变的话，那就不会渲染了（或者说复用最近的一次渲染）。

下面可以看源码事例

```js
function updateMemoComponent(
  current: Fiber | null,
  workInProgress: Fiber,
  Component: any,
  nextProps: any,
  updateExpirationTime,
  renderExpirationTime: ExpirationTime,
): null | Fiber {

  /* ...省略...*/

  // 判断更新的过期时间是否小于渲染的过期时间
  if (updateExpirationTime < renderExpirationTime) {
    const prevProps = currentChild.memoizedProps;

    // 如果自定义了compare函数，则采用自定义的compare函数，否则采用官方的shallowEqual(浅比较)函数。（下面有解析）
    let compare = Component.compare;
    compare = compare !== null ? compare : shallowEqual;

    /**
     * 1. 判断当前 props 与 nextProps 是否相等；
     * 2. 判断即将渲染组件的引用是否与workInProgress Fiber中的引用是否一致；
     *
     * 只有两者都为真，才会退出渲染。
     */
    if (compare(prevProps, nextProps) && current.ref === workInProgress.ref) {
      // 如果都为真，则退出渲染
      return bailoutOnAlreadyFinishedWork(
        current,
        workInProgress,
        renderExpirationTime,
      );
    }
  }

  /* ...省略...*/

```

shallowEqual（浅比较）

```js
// 用原型链的方法
const hasOwn = Object.prototype.hasOwnProperty

// 这个函数实际上是Object.is()的polyfill
function is(x, y) {
  if (x === y) {
    return x !== 0 || y !== 0 || 1 / x === 1 / y
  } else {
    return x !== x && y !== y
  }
}

export default function shallowEqual(objA, objB) {
  // 首先对基本数据类型的比较
  if (is(objA, objB)) return true
  // 由于Obejct.is()可以对基本数据类型做一个精确的比较， 所以如果不等
  // 只有一种情况是误判的，那就是object,所以在判断两个对象都不是object
  // 之后，就可以返回false了
  if (typeof objA !== 'object' || objA === null ||
      typeof objB !== 'object' || objB === null) {
    return false
  }

  // 过滤掉基本数据类型之后，就是对对象的比较了
  // 首先拿出key值，对key的长度进行对比

  const keysA = Object.keys(objA)
  const keysB = Object.keys(objB)

  // 长度不等直接返回false
  if (keysA.length !== keysB.length) return false
  // key相等的情况下，在去循环比较
  for (let i = 0; i < keysA.length; i++) {
  // key值相等的时候
  // 借用原型链上真正的 hasOwnProperty 方法，判断ObjB里面是否有A的key的key值
  // 属性的顺序不影响结果也就是{name:'daisy', age:'24'} 跟{age:'24'，name:'daisy' }是一样的
  // 最后，对对象的value进行一个基本数据类型的比较，返回结果
    if (!hasOwn.call(objB, keysA[i]) ||
        !is(objA[keysA[i]], objB[keysA[i]])) {
      return false
    }
  }

  return truea
}
```

由源码可以知道，加入没有传一个比较的回调函数会使用官方的浅比较。具体的可以看注释

#### memo的回调函数

我们都知道react的生命周期中有一个shouldComponentUpdate。在这个函数中返回true的话就代表本次render需要执行，而返回false就可以跳过本次的render。

而memo正好相反，返回true表示本次跳过，返回false就表示本次需要执行render。

具体怎么用呢？

大家可以自己运行一下，看看效果。一定要自己试一下，不然很容易和shouldComponentUpdate弄混了。学习还是要自己动手才行。

```js
function ChangeLog({w = ''}){
  // 省略
  console.log('====render====')
}
export default memo(ChangeLog, (prevProps, nextProps) => {
  if (prevProps.w !== nextProps.w) {
    return false
  }
  return true
});
```

## useMemo/useCallback

 ### 前言

之所以把这useMomo/useCallack两兄弟和在一起。是因为他们其实十分相似。

> 一个是缓存变量（useMemo），一个是缓存函数（useCallback）。

其实这么一说就清晰很多了。

具体的使用和useEffect一样，都是第一个参数为调用时的回调函数，第二个参数是调用判断所监听的值（可以是变量，也可以是函数）

```js
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
const memoizedValue = useCallback(() => computeExpensiveValue(a, b), [a, b]);
```

useMemo() 返回的是一个 memoized 值，只有当依赖项（比如上面的 a,b 发生变化的时候，才会重新计算这个 memoized 值）

memoized 值不变的情况下，不会重新触发渲染逻辑。

说起渲染逻辑，需要记住的是 useMemo() 是在 render 期间执行的，所以不能进行一些额外的副操作，比如网络请求等。

如果没有提供依赖数组（上面的 [a,b]）则每次都会重新计算 memoized 值，也就会 re-redner

useCallback也是一样的，这里就不多赘述了。

## useReducer/useContext

### 前言

帮他们两兄弟和在一起说主要说因为他们两兄弟在一般情况下是可以与Redux一战的。

但是！！

但是啊，但是如果你需要中间价，或者说需要“时间旅行”，又或者临时需要跨页面级的数据共享，那你还是需要redux来解决的。不过基本上的场景我们使用useReducer和useContext就可以完美的替代redux了。

### 怎么做？

其实之前由于要起一个新项目，但是突然发现有一个爷爷组件的值为需要通知给孙子组件，然后孙子组件可能会用掉回调函数调用爷爷组件的方法。那个时候其实已经用Hook写了一半了，懒得加Redux了，又不想一层层传props下去。怎么办？

通过了解，我知道了useReducer/useContext刚刚好可以解决我的需求。

```js
const TodosDispatch = React.createContext(null);

const initialState = { bar: null };

function reducer(state, action) {
  switch (action.type) {
    case 'setCenter':
      return { ...state, bar: action.bar };
    default:
      return state
  }
}

// 虽然这个是自组件，但是哪怕是曾曾曾孙子组件都可以直接用useContext拿到dispatch
function DeepChild(props) {
  // 如果我们想要执行一个 action，我们可以从 context 中获取 dispatch。
  const dispatch = useContext(TodosDispatch);

  function handleClick() {
    dispatch({ type: 'add', text: 'hello' });
  }

  return (
    <button onClick={handleClick}>Add todo</button>
  );
}

function TodosApp() {
  // 提示：`dispatch` 不会在重新渲染之间变化
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <TodosDispatch.Provider value={dispatch}>
      <DeepTree todos={todos} />
    </TodosDispatch.Provider>
  );
}
```

## useImperativeHandle/forwardRef

这里就不多说了，基本上没有什么坑点和疑难点。

说一下基本用法和ant-design form中的使用

### 正常使用

```js
useImperativeHandle(ref, createHandle, [deps])
```

`useImperativeHandle` 可以让你在使用 `ref` 时自定义暴露给父组件的实例值。在大多数情况下，应当避免使用 ref 这样的命令式代码。`useImperativeHandle` 应当与 `forwardRef`一起使用：

```js
function FancyInput(props, ref) {
  const inputRef = useRef();
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    }
  }));
  return <input ref={inputRef} ... />;
}
FancyInput = forwardRef(FancyInput);
```

在本例中，渲染<FancyInput ref={inputRef} />的父组件可以调用 `inputRef.current.focus()`。

### ant-design form

```js
// ref从第二个参数取，这里都是一致的
const Example = (props,ref) => {
  const bar = () => {}
  useImperativeHandle(ref, () => ({
    text: bar,
  }));
  return (
    <>
        <Form>
    			{//....}
        </Form>
    </>
  );
};

export default memo(Form.create()(forwardRef(Example)));

```

```js

const Parent = () => {
  return (
    <>
      <Example
      	// 注意这个不再是传ref了，而是传wrappedComponentRef。因为antd的form他返回的是一个新的对象，这个是他自定义的一个接收ref的值
        wrappedComponentRef={editTemplateRef}
      />
    </>
  );
};
```



## useDebugValue/自定义Hook

### 前言

useDebugValue是专门用于服务自定义的Hook的。

具体看看使用就好



useDebugValue，目的是能在react的浏览器调试工具上显示你的自定义hooks，或者给hooks标记一些东西
当使用一个参数的时候，就是把第一个参数标记在react的调试工具上,下面写一个简单的例子

```js
import React, { useDebugValue, useState } from 'react';

const useTest = () => {
    const [str, setStr] = useState<string>('');
    useDebugValue('debug');
    return {
        str, setStr
    }
}
export default (): JSX.Element => {
    const { str, setStr } = useTest();
    return (
        <>
            <h2>{str}</h2>
            <button onClick={() => {
                setStr('重新渲染');
            }}>这是？？？</button>
        </>
    );
}
```



![](/image/你真的懂ReactHook吗/20190818212347571.png)



会在自定义的hooks标记到react的调试工具上面,主要用于调试工具调试使用

当传入第二个参数的情况下，第二个参数是一个回调函数，会把第一个参数当成自己的形参传入，进行一系列的操作，return回去，然后才会在react调试工具的hooks中打印出来，不然不会显示

```js
import React, { useDebugValue, useState } from 'react';

const useTest = () => {
    const [str, setStr] = useState<string>('');
    useDebugValue(str, (value:string) => {
        console.log(value);
        return '这是改造后的' + value;
    });
    return {
        str, setStr
    }
}
export default (): JSX.Element => {
    const { str, setStr } = useTest();
    return (
        <>

            <h2>{str}</h2>

            <button onClick={() => {
                setStr('重新渲染');
           }}>这是？？？</button>
       </>
    );
}
```

结果:

![](/image/你真的懂ReactHook吗/2019081821194714.png)


同时在控制台上打印了一个空字符


由于str的初始值是空的，所以打印就是空的了，这只是调试使用，hooks差不多就这些了，没有其他的了



## 观看之后

如果有哪里写的不对或者有疑问的欢迎大家在评论区互动。🙏




