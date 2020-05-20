---
title: React Hook 系列（一）初识
date: 2020-05-20 10:19:44
tags: [React, ReactHook]
category: [React]
cover: /image/cover/React.jpeg
---
## 前言

1. 这篇文章主要是讲Hook的动机，优缺点之类的；
2. 有需要看实战的伙伴欢迎直接跳到第二章

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

**Class组件：**



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



**函数式组件（使用了React Hook）：**



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

Class组件

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

React Hook：

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



## 结论

简单的说一下他的优点吧。

1. 复用代码更加简单（需要什么就“钩”进来）
2. 清爽的代码风格，一目了然。（useState支持数组和对象，可以清晰的定义特殊的字段等等）
3. 代码量更少（可以看一下我之前的子父组建的例子）
4. 更愿意去写一些小组件复用（我个人喜欢React就是因为他的组件写起来非常的顺手！ps：没有贬低其他框架的意思。。）

## 感谢

其实我个人认为React Hook在宣扬一个观念“按需加载”。

## 观看之后

点赞，可以和更多的人一起讨论学习。

如果有哪里写的不对的欢迎大家在评论区指出来。🙏




