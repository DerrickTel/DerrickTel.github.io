---
title: 【重识前端】React源码阅读（三）4千字告诉你 Fiber 初始化
date: 2021-04-06 14:08:31
tags: [react]
category: [重拾前端]
cover: /image/cover/react.png
---

# 前言

我将Fiber拆分成四块来讲

- 动机（或者说初衷
- 初始化-Fiber树的准备工作
- render-Fiber树的构建
- commit-Fiber树映射到DOM

render-Fiber树的构建

Fiber树的构建。Action！

## 一些废话，可以跳过

距离上一篇文章也有2个多月的时间了。（确实搬砖比较忙没空）现在回头看，确实一脸懵逼。但是有些原理确实是刻在脑子里的，重新捡起来也很快。

# 前景回顾

我们知道一个最普通的React代码是这样的：

```jsx
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

然后我们阅读了ReactDOM.render的源码。知道了render内部其实是调用`legacyRenderSubtreeIntoContainer`这里面其实是在做为Fiber树做初始化。

然后我们`updateContainer`的最后两个函数还没有解读，继续看。

## updateContainer

```typescript
function updateContainer(element, container, parentComponent, callback) {
  ......

  // 这是一个 event 相关的入参，此处不必关注
  var eventTime = requestEventTime();

  ......

  // 这是一个比较关键的入参，lane 表示优先级
  var lane = requestUpdateLane(current);
  // 结合 lane（优先级）信息，创建 update 对象，一个 update 对象意味着一个更新
  var update = createUpdate(eventTime, lane); 

  // update 的 payload 对应的是一个 React 元素
  update.payload = {
    element: element
  };

  // 处理 callback，这个 callback 其实就是我们调用 ReactDOM.render 时传入的 callback
  callback = callback === undefined ? null : callback;
  if (callback !== null) {
    {
      if (typeof callback !== 'function') {
        error('render(...): Expected the last optional `callback` argument to be a ' + 'function. Instead received: %s.', callback);
      }
    }
    update.callback = callback;
  }

  // 将 update 入队
  enqueueUpdate(current, update, lane);
  // 调度 fiberRoot 
  scheduleUpdateOnFiber(current, lane, eventTime);
  // 返回当前节点（fiberRoot）的优先级
  return lane;
}
```

# 源码攻读

还剩下两个函数`enqueueUpdate`和`scheduleUpdateOnFiber`

这两个兄弟弄完不知道后面还剩下什么，一起看看吧。

## enqueueUpdate

```tsx
export function enqueueUpdate<State>(
  fiber: Fiber,
  update: Update<State>,
  lane: Lane,
) {
  const updateQueue = fiber.updateQueue;
  if (updateQueue === null) {
    // Only occurs if the fiber has been unmounted.
    return;
  }

  const sharedQueue: SharedQueue<State> = (updateQueue: any).shared;

  if (isInterleavedUpdate(fiber, lane)) {
    const interleaved = sharedQueue.interleaved;
    if (interleaved === null) {
      // This is the first update. Create a circular list.
      update.next = update;
      // At the end of the current render, this queue's interleaved updates will
      // be transfered to the pending queue.
      pushInterleavedQueue(sharedQueue);
    } else {
      update.next = interleaved.next;
      interleaved.next = update;
    }
    sharedQueue.interleaved = update;
  } else {
    const pending = sharedQueue.pending;
    if (pending === null) {
      // This is the first update. Create a circular list.
      update.next = update;
    } else {
      update.next = pending.next;
      pending.next = update;
    }
    sharedQueue.pending = update;
  }
	// 开发模式
  if (__DEV__) {
    if (
      currentlyProcessingQueue === sharedQueue &&
      !didWarnUpdateInsideUpdate
    ) {
      console.error(
        'An update (setState, replaceState, or forceUpdate) was scheduled ' +
          'from inside an update function. Update functions should be pure, ' +
          'with zero side-effects. Consider using componentDidUpdate or a ' +
          'callback.',
      );
      didWarnUpdateInsideUpdate = true;
    }
  }
}
```

