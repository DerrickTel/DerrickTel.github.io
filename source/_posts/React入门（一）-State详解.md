---
title: React入门（一） State详解
date: 2019-06-19 11:22:03
tags: [React, State]
category: [React]
---

@[TOC](React入门（一） State详解)

# React入门（一） State详解

我们都是程序员，废话不多说马上开始！

## 一、 Demo创建/下载

两种方法创建新的React APP

### 1. github下载

- 下载[git地址](https://github.com/DerrickTel/ReactDemo1.git).
 - 解压-打开
  - npm install
  - npm start 
  - ![如果有询问](http://i2.tiimg.com/691643/b41e573850b474a3.png)
  - y（3000端口任务在运行是否愿意运行在别的端口上？）



### 2. 自己的命令行创建
- 找到自己**心仪的文件夹**   *（全英文）*
 - 用命令行打开并抵达**心仪文件夹**
  - npx create-react-app my-app
  - cd my-app
  - npm start
> **注意**
> npx在第一行不是一个错字 - 它是一个包转发工具，附带npm 5.2+。
> 创建的时候已经默认运行过（npm install ）可以直接start

然后把 src/app.js的文件内容先改一下

```javascript
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';

class App extends Component {
  render() {
    return (
      <div className="App">
        <header className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <p>
            Edit <code>src/App.js</code> and save to reload.
          </p>
          <p>
            修改文件夹 <code>src/App.js</code> 保存时候之后自动加载.
          </p>
        </header>
      </div>
    );
  }
}

export default App;

```
Ctrl + C 

Ctrl + V

## 二、State是什么？
![在这里插入图片描述](http://i2.tiimg.com/691643/fa5b342683cef063.png)

英文翻译是状态。

其实也可以理解为状态，一段文字中的某个值改变了，也可以理解为状态改变了。
看一个简单的例子


把 src/app.js的文件内容先改一下

```javascript
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';

class App extends Component {
  state={Text:'我是state'}
  render() {
    return (
      <div className="App">
        <header className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <p>
            State输出：{this.state.Text}
          </p>
        </header>
      </div>
    );
  }
}

export default App;
```
```javascript
  state={Text:'我是state'}
```

这里的state是给state设置初值，不然会报错。不相信的话自己可以试一下。*（因为state是undefined，所以他里面取不到Text）*

通过上面的例子可以知道state里面放的是XX的状态

key value对应的

this.state.xx就可以取到对应key的**状态**的*值*


## 三、State如何改变？

通常在JS里面，我们要改变某一个值可能就只需要  X = XX; 就可以。


这里的state是一个状态。状态改变了，页面会自动刷新为最新的页面用最新的状态显示。
下面我们做一个小实验。


把 src/app.js的文件内容先改一下

```javascript
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';

class App extends Component {
  state={Text:'我是state'}
  render() {
    return (
      <div className="App">
        <header className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <p>
            State输出：{this.state.Text}
          </p>
          <button onClick={()=>{
            this.state.Text = '我是通过this.state.Text改变的State';
            alert(this.state.Text);
          }}>
            按我改变state
          </button>
        </header>
      </div>
    );
  }
}

export default App;
```

（不要在意细节！！！我们是测试按钮的样式不重要！！！）

点击按钮之后，我们发现，state的值是改变了，但是页面上面显示的值不是我们想要的啊。



那我要怎么才可以刷新页面出现我想要的state的值呢？

官方提供的方法

setState(updater, [callback])


可以这么理解，针对上面的例子


```javascript
this.setState({Text:'我是通过this.state.Text改变的State'}, function(){})
```

后面的function，可以有，也可以没有。不需要的话就可以不用写-----------稍后会说

所以只需要

```javascript
this.setState({Text:'我是通过this.state.Text改变的State'})
```


把 src/app.js的文件内容先改一下

```javascript
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';

class App extends Component {
  state={Text:'我是state'}
  render() {
    return (
      <div className="App">
        <header className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <p>
            State输出：{this.state.Text}
          </p>
          <button onClick={()=>{
            this.setState({Text:'我是通过this.state.Text改变的State'})
            alert(this.state.Text);
          }}>
            按我改变state
          </button>
        </header>
      </div>
    );
  }
}

export default App;
```
保存-热加载
之后我们发现，状态是改变了，可是alert的值不是我们想要的，还是老的值（状态）的。

可能你不知道我在说什么

通过一个小例子我们仔细感受一下。官方的setState究竟在干什么。

把 src/app.js的文件内容先改一下

```javascript
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';

class App extends Component {
  state={
    Text:'大家一起看log吧，我这次啥也不做',
    conut:0,
  }
  render() {
    return (
      <div className="App">
        <header className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <p>
            State输出：{this.state.Text}
          </p>
          <button onClick={()=>{
            console.log(this.state.conut);
            this.setState({conut: this.state.conut + 1});
            console.log(this.state.conut);
          }}>
            按我改变state
          </button>
        </header>
      </div>
    );
  }
}

export default App;
```

打开我们的react页面
按Ctrl + Shift + I
看log
然后按钮

![在这里插入图片描述](http://i2.tiimg.com/691643/52f1dcf23ee2a658.png)

诶，我不是setState了吗，为什么值没有改变还是初值呢？

我们换一个方法试一下
```javascript
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';

class App extends Component {
  state={
    Text:'大家一起看log吧，我这次啥也不做',
    conut:0,
  }
  render() {
    return (
      <div className="App">
        <header className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <p>
            State输出：{this.state.Text}
          </p>
          <button onClick={()=>{
            console.log(this.state.conut);
            this.setState({conut: this.state.conut + 1})
            setTimeout(()=>{
              console.log(this.state.conut);
            },1000);
          }}>
            按我改变state
          </button>
        </header>
      </div>
    );
  }
}

export default App;
```


那我们是不是可以认为setState不是一个立刻生效的函数。

有点类似异步的网络请求。



这是一个坑，很多新手都会遇到的坑。我曾经也遇到所以写出来。

那既然是类似异步的网络请求肯定也有callback咯？

是的！

我们把console.log放到刚刚我们没有写function里面。

```javascript
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';

class App extends Component {
  state={
    Text:'大家一起看log吧，我这次啥也不做',
    conut:0,
  }
  render() {
    return (
      <div className="App">
        <header className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <p>
            State输出：{this.state.Text}
          </p>
          <button onClick={()=>{
            console.log(this.state.conut);
            this.setState({conut: this.state.conut + 1}, ()=>{
              console.log(this.state.conut);
            })
            
          }}>
            按我改变state
          </button>
        </header>
      </div>
    );
  }
}

export default App;
```

这样的好处我们不需要手动的控制等待时间的大小，因为根据设备的不同这个时间可能会太多或者太少。

本人React小菜，有说的不对的地方还请大神指出。
