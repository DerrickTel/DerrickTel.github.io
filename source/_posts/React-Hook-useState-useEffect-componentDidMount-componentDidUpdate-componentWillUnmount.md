---
title: >-
  React Hook useState useEffect componentDidMount componentDidUpdate
  componentWillUnmount
date: 2019-07-30 21:00:00
tags: [ReactHook, React]
category: [React]
cover: https://github.com/DerrickTel/DerrickTel.github.io/blob/master/img/cover/React.jpeg?raw=true
---
## 介绍
Hook 是 React 16.8 的新增特性。它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性。

## 缘由

Hook的初衷是为了解决原本无状态组建需要使用state, 必须改造为class这个痛点.

## useState

```
import React, { useState } from 'react';

function Example() {
  // 声明一个叫 "count" 的 state 变量
  const [count, setCount] = useState(0);

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
这个是官方提供的最简单的例子.

不难理解, 按钮每次点击都会调用一次setCount, 从而改变count的值

和以下的例子等价

```
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

我用注释来解释可能更好理解useState每个参数的意义，稍微改造一下第一个例子

```
import React, { useState } from 'react';

function Example() {
  // 声明一个叫 "count" 的 state 变量
  const [
	  count, 　// 在state里面的名字
	  setCount  // 改变这个名字的函数
  ] = useState(
  	0  // 初值count的初值
  );

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => 
      setCount(
      count + 1  //  准备把count该成什么样子
      )}>
        Click me
      </button>
    </div>
  );
}
```

在以前class的形式, 所有的可变数据都放在一个state内部进行维护, 这样这个state会越来越大...越来越臃肿...越来越难以维护..如果没有注释可能就难以理解...这样就诞生了**Redux**

我本人认为, useState可以直接解决这样的一个痛点, 下面是我在新项目中使用**hook**的例子

```
// 表格loading
  const [loading, setLoading] = useState(true);

  // 表格数据
  const [listData, setListData] = useState({ list: [], total: 0 });

  // 当前页码
  const [current, setCurrent] = useState(0);

  // 搜索数据
  const [searchData, setSearchData] = useState({});

  // 医生职称
  const [jobTitle, setJobTitle] = useState([]);

  // 科室
  const [dept, setDept] = useState([]);

  // 弹窗显隐
  const [visible, setVisible] = useState(false);

  // 弹窗数据
  const [showData, setShowData] = useState({});
```
可以很直观的看到基本上一个数据享受一个useState...配合正确的注释, 调用正确的方法, 使代码可读性大大增强.

## useEffect

> 如果你熟悉 React class 的生命周期函数，你可以把 useEffect Hook 看做 `componentDidMount，componentDidUpdate 和 componentWillUnmount` 这三个函数的组合。

这句话来自官网的原画.

接下来我就为大家解释useEffect

> useEffect 会在每次渲染后都执行吗？ 是的，默认情况下，它在第一次渲染之后和每次更新之后都会执行。

这个是官网的原话, 不难理解,这样就可以模拟出componentDidUpdate..你可以在hook里面写你想要逻辑. .

直接上官网代码

```
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

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

可以看到每次点击按钮, 都重新set了Count的值, 因为每次更新都会走到useEffect(后面会说到怎么样不每次都进入useEffect ).
他是useEffect的逻辑是,每次都修改`document.title`

这样就模拟了`componentDidUpdate`

**componentDidMount  componentWillUnmount**

useEffect其实有两个参数, 第一个是调用函数, 第二个是监听值.

```
useEffect(
  () => { // 我叫A函数
    const subscription = props.source.subscribe();
    return () => {
      subscription.unsubscribe();
    };
  },
  [props.source],
);
```

这段代码可以理解为, 程序一运行, 就调用了一次A函数,之后每次渲染虽然都会走到这个useEffect. 因为他有第二个参数.所以只有在 `[props.source]` 变化的时候.才会再次调用A函数.

我们可以灵活的调用起来, 这个值可以来自useState控制.你想他变化的时候,你就用useState改变一下他的值.

最典型的例子就是, **短信的倒计时**

那我怎么样才可以优雅的让这个useEffect只调用一次.像componentDidMount呢?

可以这样在第二个值传一个控制进去.

```
useEffect(() => {
    const firstGet = async () => {
      const [z, x, c] = await Promise.all([
        requestZ(),
        requestX(),
        requestZ(),
      ]);
      // 做你想做的事情
    };
    firstGet();
  }, []);
```

这样就可以很优雅的模仿componentDidMount.而不需要在后面搞什么没人知道的花里胡哨的值.

那怎么样才可以模仿componentWillUnmount呢?

```
useEffect(() => {
    // Specify how to clean up after this effect:
    return function cleanup() {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  }, []);
```

> 为什么要在 effect 中返回一个函数？ 这是 effect 可选的清除机制。每个 effect 都可以返回一个清除函数。如此可以将添加和移除订阅的逻辑放在一起。它们都属于 effect 的一部分。

> React 何时清除 effect？ React 会在组件卸载的时候执行清除操作。正如之前学到的，effect 在每次渲染的时候都会执行。这就是为什么 React 会在执行当前 effect 之前对上一个 effect 进行清除。

这些都是官网的原话.  代码中的return 就是清除..同样的,在第二个值放入一个空.这样就会很优雅的清除了. 最明显就是短信倒计时的`setInerval`, clear一下才不会一直占用资源

>本人自创自用 一个好用的AntDesign Form快速生成器
>https://github.com/DerrickTel/ant-design-form
>欢迎大家提意见,本人会不定期更新.有新的需求都可以给我提issue  
>能动动你的小手点个赞就更棒啦~