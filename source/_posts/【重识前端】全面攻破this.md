---
title: 【重识前端】全面攻破this
date: 2020-07-16 22:02:55
tags: [this, JavaScript]
category: [重拾前端]
cover: /image/cover/web.jpeg
---

## 前言

其实说起this，这个几乎是前端面试必考题，也是前端最多“脑经急转弯”的地方，也是让无数前端人烦恼的地方。今天我们就彻底的深入this，全面的攻破它！

## 绑定规则

我们来看看在函数的执行过程中调用位置如何决定 this 的绑定对象。

你必须找到调用位置，然后判断需要应用下面四条规则中的哪一条。我们首先会分别解释 这四条规则，然后解释多条规则都可用时它们的优先级如何排列。

this的绑定规则总共有以下5种：

1. 默认绑定（最令人头疼的）
   1. 严格模式
   2. 非严格模式
2. 隐式绑定
3. 显式绑定
   1. bind
   2. call
   3. apply
4. new绑定

### 默认绑定

默认绑定顾名思义，就是没人要的“孤儿”就会应用默认绑定。

思考🤔一下👇的代码会输出什么？

```js
var a = 'out'
function fnc() {
  var a = 'in'
  console.log(this.a)
}
fnc();
```

**答案是：'out'**

我们可以看到当调用 `fnc()` 时，`this.a` 被解析成了全局变量 `a`。为什么?因为在本 例中，函数调用时应用了 `this` 的默认绑定，因此 `this` 指向全局对象。

那我们怎么知道他是应用了默认绑定呢？

我们发现 `fnc` 他只是孤零零的被调用，没有任何的修饰*（调用位置是否有上下文对象，或者说是否被某个对象拥有或者包含，不过这种说法可能会造成一些误导。）*，所以它就相当于是没有人要的`孩子`，只能去`福利院`---默认绑定。

这样说可能还不太明白默认绑定究竟是怎么一回事。等到后面的规则介绍的多了，你就会发现没有应用其他规则的“孤儿”只能来福利院 ---默认绑定的怀抱

#### 严格模式

在严格下，默认绑定不是绑定到`window`，而是`undefined`。

```js
var a = 'out'
function fnc() {
	"use strict";

  var a = 'in'
  console.log(this.a)
}
fnc();
```

**答案是：报错**

为什么？因为我们想在`undefined`里面找`a`，那肯定是报错的

### 隐式绑定

隐式绑定，顾名思义就是悄悄的绑定，或者说公认的，但是没有明说的绑定。就好比一个男人和一个女人手牵手走在一起，他们基本上可以认定就是在情侣，但是还是有可能会是“偷情”（映射到隐式绑定的隐式丢失，等会会说到。）

思考🤔一下👇的代码会输出什么？

```js
var a = 'out'
function fnc() {
  console.log(this.a)
}
var obj = {
  a: 'in',
  fnc: fnc
}
obj.fnc();
```

**答案是：'in'**

我们可以注意到，`fnc` 是在什么情况下被调用的？是`obj` 调用的，说明这个`fnc`已经有主了，护花使者是`obj`，所以`fnc`说的话肯定是向着`obj`的，胳膊肘不会往外拐。

其实这里面有一个小的点需要注意，那就是如果是多个对象引用的调用，那这个时候的执行上下文又是谁的呢？

其实我们用常理就可以解释这个问题，A叫B去吃饭，但是B想叫C一起去，结果A和B起了冲突，你作为这个C你会向着谁？那肯定是B，因为有C叫你去，你们才会出现在这场聚会。所以这个问题不难回答。我们直接看一个例子🌰

```js
var name = 'aa'
function fnc() {
  console.log(this.name)
}
var bb = {
  name: 'bb',
  fnc: fnc
}
var cc = {
  name: 'cc',
  bb: bb
}
cc.bb.fnc();
```

**答案是：bb**

为什么？因为能让`fnc`被调用的是`bb`，没有`bb`，`fnc`根本没有机会登场。

#### 隐式丢失

前面说过，虽然很多东西表上面看起来都很正常，但是也有可能有一些其他状况的出现。比如隐式绑定里面的隐式丢失。

我们把之前的例子修改一下

```js
var name = 'aa'
function fnc() {
  console.log(this.name)
}
var bb = {
  name: 'bb',
  fnc: fnc
}
var cc = {
  name: 'cc',
  bb: bb
}
var aa = cc.bb.fnc;
aa()
```

**答案是：aa**

为什么？这个其实也很好理解，这个`fnc`并没有被`bb`调用，真正调用的地方是在`window`里面声明的一个变量`aa`。所以`bb`丢失了`fnc`的信任，`fnc`无处可去，只能去孤儿院。

还是刚刚吃饭的例子，我们修改一下场景就很好理解了。

A叫B去吃饭，B把C的联系方式不小心弄丢了，结果被一个陌生人D捡去了，刚刚好D也要参加聚会，就打电话叫了D一起参加聚会。但是C不认识D，叫他去的不是熟人B，所以谁也信不过。最终只能应用默认规则--默认绑定了。



还有一种情况是非常常见的，也是隐式丢失，那就是回调函数。

看下面的例子：

```js
var name = 'window'
function fnc() {
  console.log(this.name)
}
function ffnncc(fn) {
  fn()
}
var obj = {
  name: 'obj',
  fnc: fnc,
}
ffnncc(obj.fnc)
```

**答案是：widnow**

为什么？这和之前的那个例子一样，表面上看起来好像是`obj`调用的，但是其实是`obj`把调用`fnc`的方法转交给了别人，由`fnc`不认识的来调用了。

再来看一个面试题经常考的题目：

```js
var name = 'window'
function fnc() {
  console.log(this.name)
}
var obj = {
  name: 'obj',
  fnc: fnc,
}
setTimeout(obj.fnc, 100)
```

**答案是：widnow**

我相信不用解释你也找到为什么了吧？`fnc`的联系方式被`obj`转交给别人了！

哈哈哈哈，有没有感觉this其实也不过如此，so easy！

> 思考题：那么react中的函数调用为什么要用箭头函数或者为什么要在constructor里面bind一下。

### 显式绑定

我们刚刚看到的是隐式绑定，是偷偷摸摸的那种。接下来我们就介绍一下显式绑定，光明正大的那种。

那么如果我们不想在对象内部包含函数引用，而想在某个对象上强制调用函数，该怎么

做呢?

JavaScript 中的“所有”函数都有一些有用的特性(这和它们的 [[ 原型 ]] 有关——之后我 们会详细介绍原型)，可以用来解决这个问题。具体点说，可以使用函数的 call(..) 和 apply(..) 方法。严格来说，JavaScript 的宿主环境有时会提供一些非常特殊的函数，它们 并没有这两个方法。但是这样的函数非常罕见，JavaScript 提供的绝大多数函数以及你自 己创建的所有函数都可以使用 call(..) 和 apply(..) 方法。

看下面的例子：

```js
var a = 'window'
function foo() { 
	console.log( this.a );
}
var obj = { 
	a:'obj'
};
foo.call( obj );
foo()
```

**答案的先后顺序是：obj，window**

通过 foo.call(..)，我们可以在调用 foo 时强制把它的 this 绑定到 obj 上。

可惜，显式绑定仍然无法解决我们之前提出的丢失绑定问题。

```js
function sayHi(){
    console.log('Hello,', this.name);
}
var person = {
    name: 'YvetteLau',
    sayHi: sayHi
}
var name = 'Wiliam';
var Hi = function(fn) {
    fn();
}
Hi.call(person, person.sayHi); 
```

**答案是：Hello, Wiliam**

其实我们不难发现，这个`fn`不是直接被调用的，所以造成了隐式丢失。我们的`call`绑定的`person`其实绑定在了`Hi`上面了，不信的话我们改一下你就会发现了。

```js
function sayHi(){
    console.log('Hello,', this.name);
}
var person = {
    name: 'YvetteLau',
    sayHi: sayHi
}
var name = 'Wiliam';
var Hi = function(fn) {
  	console.log(this.name)
    fn();
}
Hi.call(person, person.sayHi); 
```

输出了什么？

**YvetteLau**

**Hello, Wiliam**

那么咋办呢？别怕显示绑定的变异版，硬绑定可以解决这个问题。

#### 硬绑定

硬绑定其实就是在最后一层给他进行显示绑定。

请看代码：

```js
function sayHi(){
    console.log('Hello,', this.name);
}
var person = {
    name: 'YvetteLau',
    sayHi: sayHi
}
var name = 'Wiliam';
var Hi = function(fn) {
  	console.log(this.name)
    fn.call(person);
}
Hi(person.sayHi); 
```

输出了什么？

**YvetteLau**

**Hello, Wiliam**

虽然我们调用`Hi`的时候，在`call`之前`this`还是指向了`window`，但是我们用`call`给他绑定上了`person`。无论之后如何调用函数 `fn`，它 总会手动在 `person` 上调用 `fn`。这种绑定是一种显式的强制绑定，因此我们称之为硬绑定。

```js
function foo() {
    console.log( this.a );
}

var obj = {
    a: 2
};

var bar = function() {
    foo.call( obj );
};

bar(); // 2
setTimeout( bar, 100 ); // 2

// 硬绑定的bar不可能再修改它的this
bar.call( window ); // 2
```

典型应用场景是创建一个包裹函数，负责接收参数并返回值。

```js
function foo(something) {
    console.log( this.a, something );
    return this.a + something;
}

var obj = {
    a: 2
};

var bar = function() {
    return foo.apply( obj, arguments );
};

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

创建一个可以重复使用的辅助函数。

```js
function foo(something) {
    console.log( this.a, something );
    return this.a + something;
}

// 简单的辅助绑定函数
function bind(fn, obj) {
    return function() {
        return fn.apply( obj, arguments );
    }
}

var obj = {
    a: 2
};

var bar = bind( foo, obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

ES5内置了`Function.prototype.bind`，bind会返回一个硬绑定的新函数，用法如下。

```js
function foo(something) {
    console.log( this.a, something );
    return this.a + something;
}

var obj = {
    a: 2
};

var bar = foo.bind( obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

API调用的“上下文”

JS许多内置函数提供了一个可选参数，被称之为“上下文”（context），其作用和`bind(..)`一样，确保回调函数使用指定的this。这些函数实际上通过`call(..)`和`apply(..)`实现了显式绑定。

```js
function foo(el) {
	console.log( el, this.id );
}

var obj = {
    id: "awesome"
}

var myArray = [1, 2, 3]
// 调用foo(..)时把this绑定到obj
myArray.forEach( foo, obj );
// 1 awesome 2 awesome 3 awesome
```

### new 绑定

使用 new 来调用函数，或者说发生构造函数调用时，会自动执行下面的操作。

1. 创建(或者说构造)一个全新的对象。
2. 这个新对象会被执行[[原型]]连接。
3. 这个新对象会绑定到函数调用的this。
4. 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象。

思考下面的代码:

```js
var a = 4
function foo(a) { 
  this.a = a;
}
var bar = new foo(2); 
console.log( bar.a );  // 2
```

使用`new`来调用`foo(..)`时，会构造一个新对象并把它（`bar`）绑定到`foo(..)`调用中的this。

### 绑定优先级

看完规则，我们肯定想知道如果这些规则撞在一起，我们又该听谁的呢？

毫无疑问，默认绑定的优先级是四条规则中最低的，因为是没人应用任何规则才会去应用默认绑定。所以我们可以先不考虑它。 隐式绑定和显式绑定哪个优先级更高?我们来测试一下:



```js
function foo() { 
	console.log( this.a );
}
var obj1 = { 
	a: 2,
	foo: foo 
};
var obj2 = { 
	a: 3,
	foo: foo 
};
obj1.foo(); // 2 
obj2.foo(); // 3
obj1.foo.call( obj2 ); // 3 
obj2.foo.call( obj1 ); // 2
```

可以看到，显式绑定优先级更高，也就是说在判断时应当先考虑是否可以应用显式绑定。 现在我们需要搞清楚 new 绑定和隐式绑定的优先级谁高谁低:

```js
function foo(something) { 	
	this.a = something;
}
var obj1 = { 
	foo: foo
};
var obj2 = {};
obj1.foo( 2 );
console.log( obj1.a ); // 2
obj1.foo.call( obj2, 3 ); 
console.log( obj2.a ); // 3
var bar = new obj1.foo( 4 ); 
console.log( obj1.a ); // 2 
console.log( bar.a ); // 4
```

可以看到 new 绑定比隐式绑定优先级高。但是 new 绑定和显式绑定谁的优先级更高呢?

> new 和 call/apply 无法一起使用，因此无法通过 new foo.call(obj1) 来直接 进行测试。但是我们可以使用硬绑定来测试它俩的优先级。
>
> 这是因为函数内部有两个不同的方法：`[[Call]]`和`[[Constructor]]`。 当使用普通函数调用时，`[[Call]]`会被执行。当使用构造函数调用时，`[[Constructor]]`会被执行。`call`、`apply`、`bind`和箭头函数内部没有`[[Constructor]]`方法。

```js

function foo(something) { 
	this.a = something;
}
var obj1 = {};
var bar = foo.bind( obj1 ); 
bar( 2 );
console.log( obj1.a ); // 2
var baz = new bar(3); 
console.log( obj1.a ); // 2 
console.log( baz.a ); // 3
```

我们可以看到new把bind的硬绑定给顶掉了

所以new  > 硬绑定

#### 总结

> new 绑定 > 显示绑定 > 隐式绑定 > 默认绑定

### 绑定例外

#### 被忽略的this

把`null`或者`undefined`作为`this`的绑定对象传入`call`、`apply`或者`bind`，这些值在调用时会被忽略，实际应用的是默认规则。

下面两种情况下会传入`null`

- 使用`apply(..)`来“展开”一个数组，并当作参数传入一个函数
- `bind(..)`可以对参数进行柯里化（预先设置一些参数）

```js
function foo(a, b) {
    console.log( "a:" + a + "，b:" + b );
}

// 把数组”展开“成参数
foo.apply( null, [2, 3] ); // a:2，b:3

// 使用bind(..)进行柯里化
var bar = foo.bind( null, 2 );
bar( 3 ); // a:2，b:3 
```

总是传入`null`来忽略this绑定可能产生一些副作用。如果某个函数确实使用了this，那默认绑定规则会把this绑定到全局对象中。

> 更安全的this

安全的做法就是传入一个特殊的对象（空对象），把this绑定到这个对象不会对你的程序产生任何副作用。

JS中创建一个空对象最简单的方法是**`Object.create(null)`**，这个和`{}`很像，但是并不会创建`Object.prototype`这个委托，所以比`{}`更空。

```js
function foo(a, b) {
    console.log( "a:" + a + "，b:" + b );
}

// 我们的空对象
var ø = Object.create( null );

// 把数组”展开“成参数
foo.apply( ø, [2, 3] ); // a:2，b:3

// 使用bind(..)进行柯里化
var bar = foo.bind( ø, 2 );
bar( 3 ); // a:2，b:3 
```

#### 间接引用

间接引用下，调用这个函数会应用默认绑定规则。间接引用最容易在**赋值**时发生。

```js
// p.foo = o.foo的返回值是目标函数的引用，所以调用位置是foo()而不是p.foo()或者o.foo()
function foo() {
    console.log( this.a );
}

var a = 2;
var o = { a: 3, foo: foo };
var p = { a: 4};

o.foo(); // 3
(p.foo = o.foo)(); // 2
```

####  软绑定

- 硬绑定可以把this强制绑定到指定的对象（`new`除外），防止函数调用应用默认绑定规则。但是会降低函数的灵活性，使用**硬绑定之后就无法使用隐式绑定或者显式绑定来修改this**。
- **如果给默认绑定指定一个全局对象和undefined以外的值**，那就可以实现和硬绑定相同的效果，同时保留隐式绑定或者显示绑定修改this的能力。

```js
// 默认绑定规则，优先级排最后
// 如果this绑定到全局对象或者undefined，那就把指定的默认对象obj绑定到this,否则不会修改this
if(!Function.prototype.softBind) {
    Function.prototype.softBind = function(obj) {
        var fn = this;
        // 捕获所有curried参数
        var curried = [].slice.call( arguments, 1 ); 
        var bound = function() {
            return fn.apply(
            	(!this || this === (window || global)) ? 
                	obj : this,
                curried.concat.apply( curried, arguments )
            );
        };
        bound.prototype = Object.create( fn.prototype );
        return bound;
    };
}
```

使用：软绑定版本的foo()可以手动将this绑定到obj2或者obj3上，但如果应用默认绑定，则会将this绑定到obj。

```js
function foo() {
    console.log("name:" + this.name);
}

var obj = { name: "obj" },
    obj2 = { name: "obj2" },
    obj3 = { name: "obj3" };

// 默认绑定，应用软绑定，软绑定把this绑定到默认对象obj
var fooOBJ = foo.softBind( obj );
fooOBJ(); // name: obj 

// 隐式绑定规则
obj2.foo = foo.softBind( obj );
obj2.foo(); // name: obj2 <---- 看！！！

// 显式绑定规则
fooOBJ.call( obj3 ); // name: obj3 <---- 看！！！

// 绑定丢失，应用软绑定
setTimeout( obj2.foo, 10 ); // name: obj
```

### 箭头函数

ES6新增一种特殊函数类型：箭头函数，箭头函数无法使用上述四条规则，而是根据外层（函数或者全局）作用域（**词法作用域**）来决定this。

- `foo()`内部创建的箭头函数会捕获调用时`foo()`的this。由于`foo()`的this绑定到`obj1`，`bar`(引用箭头函数)的this也会绑定到`obj1`，**箭头函数的绑定无法被修改**(`new`也不行)。

```js
function foo() {
    // 返回一个箭头函数
    return (a) => {
        // this继承自foo()
        console.log( this.a );
    };
}

var obj1 = {
    a: 2
};

var obj2 = {
    a: 3
}

var bar = foo.call( obj1 );
bar.call( obj2 ); // 2，不是3！
```



箭头函数最常用于回调函数中，例如事件处理器或者定时器：



```js

function foo() { 
  setTimeout(() => {
    // 这里的 this 在此法上继承自 foo()
    console.log( this.a ); },100);
  }
var obj = { 
	a:2
};
foo.call( obj ); // 2

```



ES6之前和箭头函数类似的模式，采用的是词法作用域取代了传统的this机制。



```js
function foo() {
    var self = this; // lexical capture of this
    setTimeout( function() {
        console.log( self.a ); // self只是继承了foo()函数的this绑定
    }, 100 );
}

var obj = {
    a: 2
};

foo.call(obj); // 2
```

## 实现call和apply

### 了解call和apply

为什么会有call和apply？ call和apply两个方法的作用基本相同，它们都是为了改变某个函数**执行时的上下文**（context）而建立的， 他的真正强大之处就是能够扩充函数赖以运行的作用域。通俗一点讲，就是改变函数体内部**this 的指向**。



举个栗子：

```
window.color = "red";
var o = {color: "blue"};
function sayColor(){
	alert(this.color);
}
sayColor();//red
sayColor.call(this);//red，把函数体sayColor内部的this，绑到当前环境（作用域）(这段代码所处的环境)
sayColor.call(window);//red，把函数体sayColor内部的this，绑到window（全局作用域）
sayColor.call(o);//blue复制代码
```



> **解释：**上面的栗子，很明显函数sayColor是在全局作用域（环境/window）中调用的，而全局作用域中有一个color属性，值为"red"，sayColor.call(this)这一行代码就是表示**把函数体sayColor内部的this，绑到当前环境（作用域）**，而sayColor.call(window)这一行代码就是表示**把函数体sayColor内部的this，绑到window（全局作用域）**，之所以这两行的输出都是"red"就是因为他当前作用域的this就是window（this === window）； 最后，sayColor.call(o)这一行代码就表示**把函数体sayColor内部的this，绑到o这个对象的执行环境（上下文）中来**，也就是说sayColor内部的this——>**o**

### call和apply的区别

`call()` 和 `apply()`的区别在于，`call()`方法接受的是**若干个参数的列表**，而`apply()`方法接受的是**一个包含多个参数的数组**

举个例子：

```js
var func = function(arg1, arg2) {
     ...
};

func.call(this, arg1, arg2); // 使用 call，参数列表
func.apply(this, [arg1, arg2]) // 使用 apply，参数数组
```

### 应用场景

其实这些应用场景都有新的方法可以快速解决， 我只是想告诉大家一些小的知识点， 而且这些东西虽然有新的方法代替，但是面试的时候很可能会被问到。

#### 合并两个数组



```js
var a = ['a', 'aa'];
var b = ['b', 'bb'];

Array.prototype.push.apply(a, b);
console.log(a) // ['a', 'aa', 'b', 'bb']
```

其实现在有`concat`来代替了。或者其他奇淫技巧。但是，都不是我们的重点。

> 当第二个数组(如示例中的 b )太大时不要使用这个方法来合并数组，因为**一个函数能够接受的参数个数是有限制**的。不同的引擎有不同的限制，JS核心限制在 65535，有些引擎会抛出异常，有些不抛出异常但丢失多余参数。

那么要如何解决呢？

我们可以把数组进行切割。然后分批次的调用。



```js
function myConcat(arr1, arr2, max = 32768) {
  for(var i = 0; i < arr2.length; i += max) {
    Array.prototype.push.apply(
    	arr1,
      arr2.slice(i, i + max) // 小的知识点，第二个参数大于数组长度就一直取到数组的末尾
    )
  }
}

var a = [-2, -1];
var b = [];
for(var i = 0; i < 999999; i ++) {
  b.push(i)
}

Array.prototype.push.apply(a, b) // Uncaught RangeError: Maximum call stack size exceeded


myConcat(a, b)
```

#### 验证是否是数组

#### 

```js
var arr = [];
Object.prototype.toString.call(arr); // [object Array]
//把函数体Object.prototype.toString()方法内部的this，绑到arr的执行环境（作用域）复制代码
```



#### 同样是检测对象类型，arr.toString()的结果和Object.prototype.toString.call(arr)的结果不一样，这是为什么？

> 这是因为toString()为Object的原型方法，而Array ，function等引用类型作为Object的实例，都重写了toString方法。不同的对象类型调用toString方法时，根据原型链的知识，调用的是对应的**重写**之后的toString方法（function类型返回内容为函数体的字符串，Array类型返回元素组成的字符串.....），而不会去调用Object上原型toString方法，所以采用arr.toString()不能得到其对象类型，只能将arr转换为字符串类型；因此，在想要得到对象的具体类型时，应该调用Object上原型toString方法。

#### 类数组对象（Array-like Object）使用数组方法

```js
var domNodes = document.getElementsByTagName("*");
domNodes.unshift("h1");
// TypeError: domNodes.unshift is not a function

var domNodeArrays = Array.prototype.slice.call(domNodes);
domNodeArrays.unshift("h1"); // 505 不同环境下数据不同
// (505) ["h1", html.gr__hujiang_com, head, meta, ...] 
```

类数组对象有下面两个特性

- 1、具有：指向对象元素的数字索引下标和 `length` 属性
- 2、不具有：比如 `push` 、`shift`、 `forEach` 以及 `indexOf`等数组对象具有的方法

要说明的是，类数组对象是一个**对象**。JS中存在一种名为**类数组**的对象结构，比如 `arguments` 对象，还有DOM API 返回的 `NodeList` 对象都属于类数组对象，类数组对象不能使用 `push/pop/shift/unshift` 等数组方法，通过 `Array.prototype.slice.call` 转换成真正的数组，就可以使用 `Array`下所有方法。

**类数组对象转数组**的其他方法：

```js
// 上面代码等同于
var arr = [].slice.call(arguments)；

ES6:
let arr = Array.from(arguments);
let arr = [...arguments];
```

`Array.from()` 可以将两类对象转为真正的数组：**类数组**对象和**可遍历**（iterable）对象（包括ES6新增的数据结构 Set 和 Map）。

### 实现call

这里会使用隐式绑定来实现。

先丢出我们要测试的例子：

```js
var a = 1;
function f () {
  console.log(this.a)
}
var b = {
  a: 2
}
f(); // 1
f.call(b) // 2
```

OK, 我们现在就来实现以下自定义的call。应用隐式绑定的话就可以直接绑定上了。

```js

var a = 1;
function f () {
  console.log(this.a)
}

Function.prototype.myCall = function (context) {
  context.fn = f;
  context.fn();
  delete context.fn;
}

var b = {
  a: 2
}
f(); // 1
f.myCall(b) // 2
```

很潇洒的完成了。但是好像不能接受参数诶，接受参数又要考虑边界情况，比如`undefined`，`null`之类的。而且！！而且！！而且`call`不传的话是默认应用`window`的；

所以他有我们也要有！冲！

```js
Function.prototype.myCall = function(context) {
  // 取得传入的对象（执行上下文），比如上文的foo对象，这里的context就相当于上文的foo
  // 不传第一个参数，默认是window,
  var context = context || window;
  // 给context添加一个属性，这时的this指向调用myCall的函数，比如上文的bar函数
  context.fn = this;//这里的context.fn就相当于上文的bar函数
  // 通过展开运算符和解构赋值取出context后面的参数，上文的例子没有传入参数列表
  var args = [...arguments].slice(1);
  // 执行函数（相当于上文的bar(...args)）
  var result = context.fn(...args);
  // 删除函数
  delete context.fn;
  return result;
};

```

###  实现apply

因为他们两兄弟就参数不一样所以就不解释了，直接看代码吧。

```js
Function.prototype.myApply = function(context) {
      var context = context || window;
      context.fn = this;
      var result;
      // 判断第二个参数是否存在，也就是context后面有没有一个数组
      // 如果存在，则需要展开第二个参数
      if (arguments[1]) {
        result = context.fn(...arguments[1]);
      } else {
        result = context.fn();
      }
      delete context.fn;
      return result;
}

```



## 实现bind

#### 了解bind

> `bind() 方法创建一个新的函数，在 `bind()` 被调用时，这个新函数的 `this` 被指定为 `bind()` 的第一个参数，而其余参数将作为新函数的参数，供调用时使用。
>
> 语法：`fun.bind(thisArg[, arg1[, arg2[, ...]]])`
>
> MDN

#### bind和apply，call两兄弟的区别

`bind` 方法与 `call / apply` 最大的不同就是前者返回一个绑定上下文的**函数**，而后两者是**直接执行**了函数。

上代码！

```js
var value = 2;

var foo = {
    value: 1
};

function bar(name, age) {
    return {
		value: this.value,
		name: name,
		age: age
    }
};

bar.call(foo, "Jack", 20); // 直接执行了函数
// {value: 1, name: "Jack", age: 20}

var bindFoo1 = bar.bind(foo, "Jack", 20); // 返回一个函数
bindFoo1();
// {value: 1, name: "Jack", age: 20}

var bindFoo2 = bar.bind(foo, "Jack"); // 返回一个函数
bindFoo2(20);
// {value: 1, name: "Jack", age: 20}
```

通过上述代码可以看出`bind` 有如下特性：

- 1、可以指定`this`
- 2、返回一个函数
- 3、可以传入参数
- 4、柯里化

#### 实现bind

直接开干！

```js
Function.prototype.myBind = function (context) {
  var that = this; // 这里利用的闭包保存了此时此刻的this，否则后面调用的话会隐式丢失
  return function () { 
    return that.apply(context)
  }
}

// 测试用例
var value = 2;
var foo = {
    value: 1
};

function bar() {
	return this.value;
}

var bindFoo = bar.myBind(foo);

console.log(bindFoo()); // 1
```

Ok,接下来我们处理一下参数和柯里化吧。

```js
Function.prototype.myBind = function (context) {
  // 因为参数是类数组没有slice方法，通过call绑定this之后使用slice拷贝数组，除了第一个需要绑定的this 之外，下面会用到
  var args = Array.prototype.slice.call(arguments, 1)
  var that = this; // 这里利用的闭包保存了此时此刻的this，否则后面调用的话会隐式丢失
  return function () { 
    // 和上面一样拷贝参数
    var nowArgs = Array.prototype.slice.call(arguments)
    return that.apply(context, args.concat(nowArgs))
  }
}
// 测试用例
var value = 2;

var foo = {
    value: 1
};

function bar(name, age) {
    return {
		value: this.value,
		name: name,
		age: age
    }
};

var bindFoo = bar.myBind(foo, "Jack");
bindFoo(20);
// {value: 1, name: "Jack", age: 20}

```

到现在已经完成大部分了，但是还有一个难点，`bind` 有以下一个特性

> 一个绑定函数也能使用new操作符创建对象：这种行为就像把原函数当成构造器，提供的 this 值被忽略，同时调用时的参数被提供给模拟函数。

举例说明：正规的bind

```js
var value = 2;
var foo = {
    value: 1
};
function bar(name, age) {
    this.habit = 'shopping';
    console.log(this.value);
    console.log(name);
    console.log(age);
}
bar.prototype.friend = 'kevin';

var bindFoo = bar.bind(foo, 'Jack');
var obj = new bindFoo(20);
// undefined
// Jack
// 20

obj.habit;
// shopping

obj.friend;
// kevin
```

上面例子中，运行结果`this.value` 输出为 `undefined`，这不是全局`value` 也不是`foo`对象中的`value`，这说明 `bind` 的 `this` 对象失效了，`new` 的实现中生成一个新的对象，这个时候的 `this`指向的是 `obj`。

所以我们返回的时候需要判断一下他是不是作为了构造函数返回，如果是就返回当前的this，如果不是就绑定当前输入的`context`

```js
Function.prototype.myBind = function (context) {
  // 因为参数是类数组没有slice方法，通过call绑定this之后使用slice拷贝数组，除了第一个需要绑定的this 之外，下面会用到
  var args = Array.prototype.slice.call(arguments, 1)
  var that = this; // 这里利用的闭包保存了此时此刻的this，否则后面调用的话会隐式丢失
  var fnc = function () { 
    // 和上面一样拷贝参数
    var nowArgs = Array.prototype.slice.call(arguments)
    //当作为构造函数时，this 指向实例，此时 this instanceof fBound 结果为 true，可以让实例获得来自绑定函数的值，即上例中实例会具有 habit 属性。
//当作为普通函数时，this 指向 window，此时结果为 false，将绑定函数的 this 指向 context
    return that.apply(
      this instanceof fnc ? this : context, 
      args.concat(nowArgs)
    )
  }
  // 修改返回函数的 prototype 为绑定函数的 prototype，实例就可以继承绑定函数的原型中的值，即上例中 obj 可以获取到 bar 原型上的 friend。
  fnc.prototype = this.prototype
  return fnc
}
```

但是其实这样会有一个问题，我如果实例修改了原型，那么接下来的继承就会出现问题。

这个时候我们需要拷贝一下我们原型上面的参数。会用到

> 道格拉斯·克罗克福德在 2006 年写了一篇文章，题为 Prototypal Inheritance in JavaScript (JavaScript 10 中的原型式继承)。在这篇文章中，他介绍了一种实现继承的方法，这种方法并没有使用严格意义上的 构造函数。他的想法是借助原型可以基于已有的对象创建新对象，同时还不必因此创建自定义类型。为 了达到这个目的，他给出了如下函数。
>
> ```js
> function object(o){
>   function F(){}
>   F.prototype = o
>   return new F
> }
> ```
>
> 看起来非常简单
>
> 先在`object`函数内部创建一个临时的构造函数`F`, 然后将传入的这个对象`o`作为这个临时构造函数的原型, 最后返回这个临时构造函数的实例. 
>
> 简单来说就是`object`对传入的对象进行了浅复制.

这句话在我的《原型原型链和继承》里面有具体的介绍。想看的话可以点击头像查找文章。

这边可以直接使用ES5的 `Object.create()`方法生成一个新对象

```js
fBound.prototype = Object.create(this.prototype);
```

不过 `bind` 和 `Object.create()`都是ES5方法，部分IE浏览器（IE < 9）并不支持，Polyfill中不能用 `Object.create()`实现 `bind`，不过原理是一样的。

ok。那我们就修改一下

```js
Function.prototype.myBind = function (context) {
  // 因为参数是类数组没有slice方法，通过call绑定this之后使用slice拷贝数组，除了第一个需要绑定的this 之外，下面会用到
  var args = Array.prototype.slice.call(arguments, 1)
  var that = this; // 这里利用的闭包保存了此时此刻的this，否则后面调用的话会隐式丢失
  var fNOP = function () {};
  var fnc = function () { 
    // 和上面一样拷贝参数
    var nowArgs = Array.prototype.slice.call(arguments)
    //当作为构造函数时，this 指向实例，此时 this instanceof fBound 结果为 true，可以让实例获得来自绑定函数的值，即上例中实例会具有 habit 属性。
//当作为普通函数时，this 指向 window，此时结果为 false，将绑定函数的 this 指向 context
    return that.apply(
      this instanceof fNOP ? this : context, 
      args.concat(nowArgs)
    )
  }
  // 修改返回函数的 prototype 为绑定函数的 prototype，实例就可以继承绑定函数的原型中的值，即上例中 obj 可以获取到 bar 原型上的 friend。
  fNOP.prototype = this.prototype
  fnc.prototype = new FNOP()
  return fnc
}
```

到这里其实已经差不多了，突然想起来还有一个问题是调用 `bind` 的不是函数，这时候需要抛出异常。

```js
if (typeof this !== "function") {
  throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
}
```

所以

#### 顶配

```js
Function.prototype.myBind = function (context) {
  if (typeof this !== "function") {
  	throw new Error("Function.prototype.bind - what is trying to 	be bound is not callable");
	}
  // 因为参数是类数组没有slice方法，通过call绑定this之后使用slice拷贝数组，除了第一个需要绑定的this 之外，下面会用到
  var args = Array.prototype.slice.call(arguments, 1)
  var that = this; // 这里利用的闭包保存了此时此刻的this，否则后面调用的话会隐式丢失
  var fNOP = function () {};
  var fnc = function () { 
    // 和上面一样拷贝参数
    var nowArgs = Array.prototype.slice.call(arguments)
    //当作为构造函数时，this 指向实例，此时 this instanceof fBound 结果为 true，可以让实例获得来自绑定函数的值，即上例中实例会具有 habit 属性。
//当作为普通函数时，this 指向 window，此时结果为 false，将绑定函数的 this 指向 context
    return that.apply(
      this instanceof fNOP ? this : context, 
      args.concat(nowArgs)
    )
  }
  // 修改返回函数的 prototype 为绑定函数的 prototype，实例就可以继承绑定函数的原型中的值，即上例中 obj 可以获取到 bar 原型上的 friend。
  fNOP.prototype = this.prototype
  fnc.prototype = new FNOP()
  return fnc
}
```





## 实现new

#### 了解new

> **new 运算符**创建一个用户定义的对象类型的实例或具有构造函数的内置对象的实例。 ——（来自于MDN）

我们先试试现在有的new，他有什么功能，然后我们总结之后，在开始实现。

看个🌰：

```js
function NEW (a) {
  this.a = a;
}
NEW.prototype.sayA = function () {
  console.log(this.a+'b')
}
var A = new NEW(1);
A.sayA()
A.a

```

我们可以看到A是NEW的一个实例。继承了构造函数（NEW）的属性和原型上的属性（sayA）。

> **`new`** 关键字会进行如下的操作：
>
> 1. 创建一个空的简单JavaScript对象（即{}）；
> 2. 链接该对象（即设置该对象的构造函数）到另一个对象 ；
> 3. 将步骤1新创建的对象作为`this`的上下文 ；
> 4. 如果该函数没有返回对象，则返回`this`。
>
> MDN



#### 实现一个new

说干就干：

```js
// new 是关键词，不可以直接覆盖。这里使用 create 来模拟实现 new 的效果。
function create () {
  // 1.
  var obj = new Object();
  
  // 获取构造函数
  // 因为arguments是一个类数组对象，没有unshift方法，所以用了call，我们没有new这关键字，只能模拟，所以的一个参数我们认为他需要传递一个构造函数。
  var Con = [].unshift.call(arguments)
  
  // 2.
  obj.__proto__ = Con.prototype 
  
  // 3、 绑定this。因为之前arguments已经被我们删除了第一个元素，所以剩下的就是我们所需要的参数了。
  Con.apply(obj, arguments)
  
  // 4. 简陋版
  return obj
  
}
```

构造函数返回值有如下三种情况：

- 1、返回一个对象
- 2、没有 `return`，即返回 `undefined`
- 3、返回`undefined` 以外的基本类型

```js
function create() {
	// 1、获得构造函数，同时删除 arguments 中第一个参数
    Con = [].shift.call(arguments);
	// 2、创建一个空的对象并链接到原型，obj 可以访问构造函数原型中的属性
    var obj = Object.create(Con.prototype);
	// 3、绑定 this 实现继承，obj 可以访问到构造函数中的属性
    var ret = Con.apply(obj, arguments);
	// 4、优先返回构造函数返回的对象
	return ret instanceof Object ? ret : obj;
};
```

