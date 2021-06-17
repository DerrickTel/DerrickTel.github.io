---
#成年人的崩溃往往就在一瞬间title: 【重识前端】React源码阅读（一）什么是jsx
date: 2021-03-29 10:08:31
tags: [react]
category: [重拾前端]
cover: /image/cover/react.png

---

# 前言

一些废话和个人感悟，看干货的直接跳过....到`jsx`

重识前端断断续续也写了大半年了。里面基本上都是个人技术栈的积累，以及面试题的回答。感兴趣的伙伴可以点到我的主页去看。

说说今天的主角-`React`吧。`React`的源码之前一直想要去阅读学习的，也确确实实在看了，但是经常看着看着就发困。然后就开始偷懒，从一些周边的开始阅读，比如`React-Router`，`React-Redux`之类的。之前都是看完之后再开始写文章。现在想一边看一边写，换个方式看看。如果有写错的地方欢迎大家讨论，如果觉得还可以的话可以点个赞，谢谢！

作为一名`React`的重度使用者和爱好者。我将会和大家一起从「使用者」到「了解者」进步！

# React的相关信息

## 功利的角度

打开招聘软件，现在基本上的大厂，甚至是外包公司都普遍要求你会`React`或者`Vue`。给出的薪资也是非常的诱人。

（图片来自Boss某聘）

![offer](/image/react1/offer.png)



为了发财这个朴实的梦想也应该学习。

## 学习的角度

如今前端御三家：`Angular`、`React`、`Vue`三分天下。国内更青睐`React`和`Vue`。所以，从学习的角度更应该学习`React`。



https://www.npmtrends.com/react-vs-vue-vs-@angular/core



![offer](/image/react1/download.jpg)



![offer](/image/react1/stats)

## 为什么非要学习源码

这个问题我之前也问过，几乎所有的面试都是造火箭，工作拧螺丝。

我认为从两个角度来看这个问题：

1. 应聘者
   1. 吃透`React`的情况下，解决问题的上限肯定是提高了
   2. 一个优秀的框架是一位优秀的老师
2. 招聘者，既然都是能干活的，我就找一个懂得更多的人来干这个活



# JSX

废话不多说，直接开始。

JSX，再熟悉不过了，日常使用React过程中肯定会用到的。给一个demo，帮大家回一下。

```react
import React from "react";
import ReactDOM from "react-dom";

class App extends React.Component {
  render() {
    const text = "哈哈哈"
    return (
      <div>
        <p>{text}</p>
      </div>
    );
  }
}

const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);
```



我先问几个问题，如果能够清晰的快速答出来，那么恭喜你，你的基础很扎实，如果不行，那就一起看看，看完之后可以把你的答案写在评论区，和大家讨论。

- 什么是`JSX`？`JSX`和`JS`有什么关系？
- `JSX`的底层原理是什么？
- 为什么`React`要选择`JSX`？

带着问题一起来看。

## 什么是JSX

先看看`React`官网的一句话。

> ```react
> const element = <h1>Hello, world!</h1>;
> ```
>
> 它被称为 JSX，是一个 JavaScript 的语法扩展。

语法扩展，意思就是`JS`有的他都有的情况下，还有一些别的功能。可是这货，明显在`JavaScript`学习过程中没见过他呀。他能运行在`JavaScript`的环境中吗？打开`Chrome`控制台。



![console](/image/react/console.png)



上来就是劈头盖脸的语法错误，看样子这部分就是多出来的功能了呗。那咋运行啊？



```js
{
        test: /\.(js|jsx)?$/,
        // 开启缓存
        options: { cacheDirectory: true },
        loader: 'babel-loader',
},
```

怎么样，熟悉不，意思是告诉`webpack`，解析`jsx`结尾的代码，需要用到`babel`。ok，也就是说要想运行`jsx`，就必须用`babel`解析。那好，什么是babel？

### babel

拉一段`babel`官网上的一句话

> ## Babel 是一个 JavaScript 编译器
>
> Babel 是一个工具链，主要用于将 ECMAScript 2015+ 版本的代码转换为向后兼容的 JavaScript 语法，以便能够运行在当前和旧版本的浏览器或其他环境中。

`babel`，其实就是帮我们解析我们写的代码，然后变成浏览器能够认识样式。比如`jsx`，比如新鲜的`es6`语法可以运行在老版本的浏览器上，让老版本的浏览器能够认识。

那问题又来了，jsx被babel变成什么样子了？

在线转换：https://babeljs.io/repl

大家可以把上面的小demo，扔进去。



![bebel](/image/react/bebel.png)



发现没有。我们`return`出去的东西变成了`React.createElement`。哈哈，这就是为什么我们即使没有用到`React`(解决了一到面试题！)，也需要在上面`import`，因为`jsx`最后是会被转化为`React.createElement`，会用到的！（不过现在最新的好像不需要引用也不会报错了，因为他自己识别了。

官网介绍：https://zh-hans.reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html

> ```
> // 由编译器引入（禁止自己引入！）
> import {jsx as _jsx} from 'react/jsx-runtime';
> 
> function App() {
>   return _jsx('h1', { children: 'Hello world' });
> }
> ```
>
> 注意，此时源代码**无需引入 React** 即可使用 JSX 了！（但仍需引入 React，以便使用 React 提供的 Hook 或其他导出。）



回到之前`React`官网提到的

> 它被称为 JSX，是一个 JavaScript 的语法扩展。

有没有呼应起来了，确实`js`他都有，但是`js`没有的，他还有比如转化为`React.createElement`。



## createElement

我们是来读源码学习的，我们既然知道了`jsx`的底层是调用`react`的函数，我们就来看看这个函数是个什么情况。

```javascript
export function createElement(type, config, children) {
  // propName 变量用于储存后面需要用到的元素属性
  let propName; 
  // props 变量用于储存元素属性的键值对集合
  const props = {}; 
  // key、ref、self、source 均为 React 元素的属性，此处不必深究
  let key = null;
  let ref = null; 
  let self = null; 
  let source = null; 

  // config 对象中存储的是元素的属性
  if (config != null) { 
    // 进来之后做的第一件事，是依次对 ref、key、self 和 source 属性赋值
    if (hasValidRef(config)) {
      ref = config.ref;
    }
    // 此处将 key 值字符串化
    if (hasValidKey(config)) {
      key = '' + config.key; 
    }
    self = config.__self === undefined ? null : config.__self;
    source = config.__source === undefined ? null : config.__source;
    // 接着就是要把 config 里面的属性都一个一个挪到 props 这个之前声明好的对象里面
    for (propName in config) {
      if (
        // 筛选出可以提进 props 对象里的属性
        hasOwnProperty.call(config, propName) &&
        !RESERVED_PROPS.hasOwnProperty(propName) 
      ) {
        props[propName] = config[propName]; 
      }
    }
  }
  // childrenLength 指的是当前元素的子元素的个数，减去的 2 是 type 和 config 两个参数占用的长度
  const childrenLength = arguments.length - 2; 
  // 如果抛去type和config，就只剩下一个参数，一般意味着文本节点出现了
  if (childrenLength === 1) { 
    // 直接把这个参数的值赋给props.children
    props.children = children; 
    // 处理嵌套多个子元素的情况
  } else if (childrenLength > 1) { 
    // 声明一个子元素数组
    const childArray = Array(childrenLength); 
    // 把子元素推进数组里
    for (let i = 0; i < childrenLength; i++) { 
      childArray[i] = arguments[i + 2];
    }
    // 最后把这个数组赋值给props.children
    props.children = childArray; 
  } 

  // 处理 defaultProps
  if (type && type.defaultProps) {
    const defaultProps = type.defaultProps;
    for (propName in defaultProps) { 
      if (props[propName] === undefined) {
        props[propName] = defaultProps[propName];
      }
    }
  }

  // 最后返回一个调用ReactElement执行方法，并传入刚才处理过的参数
  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props,
  );
}
```



先不看参数，我们就先看返回值的名字和这个函数的名字。我们初步猜测，`jsx`是通过`createElement`这个函数来创建一个`React`元素，然后由`ReactDOM.render`来渲染到我们指定名字的元素上。





ok，那么我们平常写的jsx元素就是`ReactElement`生成的东西。我们看看`ReactElement`生成的是个什么东西。

我们先把我们平常写的`jsx`给打印一下。



![element](/image/react/element.png)



凭直觉，是不是感觉这个是一个`虚拟DOM`的节点，他有类型然后又是`div`，对应的会不会是`html`的`div`标签？还有`className`，这个很明显就是对应`class`。我们去一看究竟。

## ReactElement



```javascript
const ReactElement = function(type, key, ref, self, source, owner, props) {
  const element = {
    // 这个是一个React结点的标识符
    $$typeof: REACT_ELEMENT_TYPE,

    // 很明显就是对应的html的标签
    type: type,
    key: key,
    ref: ref,
    props: props,

    // 记录由谁创建的，暂时也不知道什么用。先挖个坑
    _owner: owner,
  };

  // 开发环境下的配置，主要就是Object.freeze，可以在MDN上面查一下。我就不多介绍了
  if (__DEV__) {
    // The validation flag is currently mutative. We put it on
    // an external backing store so that we can freeze the whole object.
    // This can be replaced with a WeakMap once they are implemented in
    // commonly used development environments.
    element._store = {};

    // To make comparing ReactElements easier for testing purposes, we make
    // the validation flag non-enumerable (where possible, which should
    // include every environment we run tests in), so the test framework
    // ignores it.
    Object.defineProperty(element._store, 'validated', {
      configurable: false,
      enumerable: false,
      writable: true,
      value: false,
    });
    // self and source are DEV only properties.
    Object.defineProperty(element, '_self', {
      configurable: false,
      enumerable: false,
      writable: false,
      value: self,
    });
    // Two elements created in two different places should be considered
    // equal for testing purposes and therefore we hide it from enumeration.
    Object.defineProperty(element, '_source', {
      configurable: false,
      enumerable: false,
      writable: false,
      value: source,
    });
    if (Object.freeze) {
      Object.freeze(element.props);
      Object.freeze(element);
    }
  }

  return element;
};
```



看过之后，是不是和我们看到的节点一样，首先，标注一下`$$typeof`为`react节点`。然后就是正常的传参，比如`key`，`ref`，`props`等等。

## ReactDOM.render

这个就是React节点渲染为真是节点的操作。话不多说，上源码！



```javascript
export function isValidContainer(node: mixed): boolean {
  return !!(
    node &&
    (node.nodeType === ELEMENT_NODE ||
      node.nodeType === DOCUMENT_NODE ||
      node.nodeType === DOCUMENT_FRAGMENT_NODE ||
      (node.nodeType === COMMENT_NODE &&
        (node: any).nodeValue === ' react-mount-point-unstable '))
  );
}

export function render(
  element: React$Element<any>,
  container: Container,
  callback: ?Function,
) {
  // 用于抛出错误的, 就是判断这个container是不是找得到
  invariant(
    isValidContainer(container),
    'Target container is not a DOM element.',
  );
  if (__DEV__) {
    const isModernRoot =
      isContainerMarkedAsRoot(container) &&
      container._reactRootContainer === undefined;
    if (isModernRoot) {
      console.error(
        'You are calling ReactDOM.render() on a container that was previously ' +
          'passed to ReactDOM.createRoot(). This is not supported. ' +
          'Did you mean to call root.render(element)?',
      );
    }
  }
  
  // 这里面涉及到了fiber的一些架构，我想之后再说，再挖个坑
  return legacyRenderSubtreeIntoContainer(
    null,
    element,
    container,
    false,
    callback,
  );
}
```



有没有感觉很简单，去除`__DEV__`情况的话。也是返回一个函数的结果。。。



## ReactDOM.hydrate



这个函数我发现很有意思，与ReactDOM.render只有一行代码不一样

```diff
legacyRenderSubtreeIntoContainer(
    null,
    element,
    container,
-   false,
+   true,
    callback,
  );
```



首先，这个`hydrate`的意思是注水。大致的意思是给一个结点增加水分，增添色彩。之前是给`SSR`过程中添加点击事件之类使用的。在`React17`之后会淘汰掉`ReactDOM.Render`，全部用`ReactDOM.hydrate`来代替。

React官网的一句话

> 使用 `ReactDOM.render()` 对服务端渲染容器进行 hydrate 操作的方式已经被废弃，并且会在 React 17 被移除。作为替代，请使用 [`hydrate()`](https://zh-hans.reactjs.org/docs/react-dom.html#hydrate)。



## 为什么要使用JSX

这里分享一个解题的小技巧。为什么要XX其实就是要问你，XX的好处。ok，那接下来就很好回答了

首先，因为如果不用`jsx`，我们就要在代码里面写上百个`React.crearteElement`。

第二，这个语法糖可以让开发者使用熟悉的`HTML`标签来创建`虚拟DOM`，降低学习成本。提升开发效率与开发体验





![bebel](/image/react/bebel.png)



# 剩余的坑

- legacyRenderSubtreeIntoContainer（涉及fiber架构
- _owner（创建的时候，这个属性有什么用

# 索引

https://zh-hans.reactjs.org/docs

https://www.babeljs.cn/docs/







