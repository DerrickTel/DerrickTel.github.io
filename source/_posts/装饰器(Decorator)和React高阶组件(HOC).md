---
title: 装饰器(Decorator)和React高阶组件(HOC)
date: 2019-09-15 21:59:48
tags: [React, ES6]
category: [ES6]
cover: https://github.com//DerrickTel/DerrickTel.github.io/blob/master/img/%E8%A3%85%E9%A5%B0%E5%99%A8(Decorator)%E5%92%8CReact%E9%AB%98%E9%98%B6%E7%BB%84%E4%BB%B6(HOC)/decorator.png?raw=true
---
## 什么是装饰器(Decorator)
装饰器（Decorator）是一种`与类（class）相关`的语法，用来注释或修改类和类方法。
装饰器是一种函数，写成`@ + 函数名`。它可以放在类和类方法的定义前面。
其实只是一个语法糖. 还没有正式发布, 还需要插件`babel-plugin-transform-decorators-legacy`使用
## 装饰器(Decorator)使用

### 类的装饰器

```javascript
@testable
class MyTestableClass {
  // ...
}

function testable(target) {
  target.isTestable = true;
}

MyTestableClass.isTestable // true
```

上面代码中，`@testable`就是一个装饰器。它修改了`MyTestableClass`这个类的行为，为它加上了静态属性`isTestable`。`testable`函数的参数`target`是`MyTestableClass`类本身。

也就是说，装饰器是一个对类进行处理的函数。装饰器函数的第一个参数，就是所要装饰的目标类。

```javascript
function testable(target) {
  // ...
}
```
如果想传参，可以在装饰器外面再封装一层函数。

```javascript
function testable(isTestable) {
  return function(target) {
    target.isTestable = isTestable;
  }
}

@testable(true)
class MyTestableClass {}
MyTestableClass.isTestable // true

@testable(false)
class MyClass {}
MyClass.isTestable // false
```
上面代码中，装饰器`testable`可以接受参数，这就等于可以修改装饰器的行为。

注意，装饰器对类的行为的改变，是代码编译时发生的，而不是在运行时。这意味着，装饰器能在编译阶段运行代码。也就是说，装饰器本质就是编译时执行的函数。

前面的例子是为类添加一个静态属性，如果想添加实例属性，可以通过目标类的`prototype`对象操作。

```javascript
function testable(target) {
  target.prototype.isTestable = true;
}

@testable
class MyTestableClass {}

let obj = new MyTestableClass();
obj.isTestable // true
```

上面代码中，装饰器函数`testable`是在目标类的`prototype`对象上添加属性，因此就可以在实例上调用。



```javascript
function testable(target) {
  target.prototype.isTestable = true;
}

@testable
class MyTestableClass {}

let obj = new MyTestableClass();
obj.isTestable // true
```


 - 实际开发中，React 与 Redux 库结合使用时

常常需要写成下面这样

```javascript
class MyReactComponent extends React.Component {}

export default connect(mapStateToProps, mapDispatchToProps)(MyReactComponent);
```

有了装饰器，就可以改写上面的代码。

```javascript
@connect(mapStateToProps, mapDispatchToProps)
export default class MyReactComponent extends React.Component {}
```
### 类的方法的装饰器

```javascript
function readonly(target, name, descriptor){
  // descriptor对象原来的值如下
  // {
  //   value: specifiedFunction,
  //   enumerable: false,
  //   configurable: true,
  //   writable: true
  // };
  descriptor.writable = false;
  return descriptor;
}

readonly(Person.prototype, 'name', descriptor);
// 类似于
Object.defineProperty(Person.prototype, 'name', descriptor);
```
装饰器第一个参数是类的原型对象，上例是`Person.prototype`，装饰器的本意是要“装饰”类的实例，但是这个时候实例还没生成，所以只能去装饰原型（这不同于类的装饰，那种情况时`target`参数指的是类本身）；第二个参数是所要装饰的属性名，第三个参数是该属性的描述对象。

> 其余使用方法与类的装饰器相同(参数变为3个了~)

### 多个装饰器
如果同一个方法有多个装饰器，会像剥洋葱一样，先从外到内进入，然后由内向外执行。

```javascript
function dec(id){
  console.log('evaluated', id);
  return (target, property, descriptor) => console.log('executed', id);
}

class Example {
    @dec(1)
    @dec(2)
    method(){}
}
// evaluated 1
// evaluated 2
// executed 2
// executed 1
```
### 装饰器不能作用于函数

装饰器只能用于类和类的方法，不能用于函数，因为存在函数提升。

## React高阶组件(HOC)

```react
import React from 'react';

export default Component => class extends React.Component {
  render() {
    return <div style={{cursor: 'pointer', display: 'inline-block'}}>
      <Component/>
    </div>
  }
}
```
这个装饰器（高阶组件）接受一个 React 组件作为参数，然后返回一个新的 React 组件。实现很简单，就是包裹了一层 div，添加了一个 style，就这么简单。以后所有被它装饰的组件都会具有这个特征。
除了style还可以传参数
```react
import React from 'react';

export default Component => class extends React.Component {
  render() {
    return <div test={'qwe'}>
      <Component/>
    </div>
  }
}
```
以后所有被它装饰的组件都可以从`props`里面获取到`test`. 他的值是`'qwe'`。

## 扩展
发挥你的想象, 你可以写无数个很方便的高阶组件, 通过装饰器的方式, 让你的代码更简洁, 更帅

索引

> http://es6.ruanyifeng.com/#docs/decorator