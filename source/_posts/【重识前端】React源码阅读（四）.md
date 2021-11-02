---
title: 【重识前端】React源码阅读（四）
date: 2021-06-30 14:08:31
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

这里先补充一下，`initializeUpdateQueue`内部的东西，因为也有会用了，肯定会有疑问。

```typescript
export function initializeUpdateQueue<State>(fiber: Fiber): void {
  const queue: UpdateQueue<State> = {
    baseState: fiber.memoizedState,
    firstBaseUpdate: null,
    lastBaseUpdate: null,
    shared: {
      pending: null,
      interleaved: null,
      lanes: NoLanes,
    },
    effects: null,
  };
  fiber.updateQueue = queue;
}
```



## enqueueUpdate

```tsx
export function enqueueUpdate<State>(
  fiber: Fiber,
  update: Update<State>,
  lane: Lane,
) {
  const updateQueue = fiber.updateQueue;
  if (updateQueue === null) {
    // 只在fiber 卸载的时候会发生
    // Only occurs if the fiber has been unmounted.
    return;
  }

  const sharedQueue: SharedQueue<State> = (updateQueue: any).shared;
  // 比较fiber lane和lane，相同时更新，具体内容可以看下面的源码。这里不加讨论
  // render初始化时不执行
  if (isInterleavedUpdate(fiber, lane)) {
    const interleaved = sharedQueue.interleaved;
    if (interleaved === null) {
      // 如果是第一次更新，创建双向链表
      // This is the first update. Create a circular list.
      update.next = update;
      //在当前渲染结束时，将显示此队列的交错更新
      //被转移到挂起队列。
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
      //这是第一次更新。创建单向链表。
      // This is the first update. Create a circular list.
      update.next = update;
    } else {
      // 定义双向列表
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



### isInterleavedUpdate

```typescript
export function isInterleavedUpdate(fiber: Fiber, lane: Lane) {
  return (
    // TODO: Optimize slightly by comparing to root that fiber belongs to.
    // Requires some refactoring. Not a big deal though since it's rare for
    // concurrent apps to have more than a single root.
    //TODO：通过与fiber所属的根进行比较来稍微优化。
    //需要一些重构。不过没什么大不了的，因为它很少见
    //具有多个根的并发应用程序。 
    workInProgressRoot !== null &&
    (fiber.mode & ConcurrentMode) !== NoMode &&
    // If this is a render phase update (i.e. UNSAFE_componentWillReceiveProps),
    // then don't treat this as an interleaved update. This pattern is
    // accompanied by a warning but we haven't fully deprecated it yet. We can
    // remove once the deferRenderPhaseUpdateToNextBatch flag is enabled.
    //如果这是渲染阶段更新（即 UNSAFE_componentWillReceiveProps），
    //然后不要将其视为交错更新。这个图案是
    //伴随着警告，但我们还没有完全弃用它。我们可以
    //启用 deferRenderPhaseUpdateToNextBatch 标志后删除。 
    (deferRenderPhaseUpdateToNextBatch ||
      (executionContext & RenderContext) === NoContext)
  );
}
```

上面提到的。函数的大概是判断是否是交错更新。



## scheduleUpdateOnFiber

```typescript
export function scheduleUpdateOnFiber(
  fiber: Fiber,
  lane: Lane,
  eventTime: number,
): FiberRoot | null {
  // 查询是否超出最大的渲染深度，并给出警告
	// const NESTED_UPDATE_LIMIT = 50; 最大深度
  checkForNestedUpdates();
  // 判断是否在 render 阶段进行更新,
  warnAboutRenderPhaseUpdatesInDEV(fiber);

  // 更新过期时间,并从当前节点返回根节点
  const root = markUpdateLaneFromFiberToRoot(fiber, lane);
  if (root === null) {
    // 开发模式，没有fiber给出警告，就不在下面贴代码了。
    warnAboutUpdateOnUnmountedFiberInDEV(fiber);
    return null;
  }

  // 判断是否有在跟踪
  if (enableUpdaterTracking) {
    // 是否是开发环境
    if (isDevToolsPresent) {
      // 添加Fiber的Map关系
      addFiberToLanesMap(root, fiber, lane);
    }
  }

  // Mark that the root has a pending update.
  // 在root上标记更新，将update的lane放到root.pendingLanes
  markRootUpdated(root, lane, eventTime);

  if (enableProfilerTimer && enableProfilerNestedUpdateScheduledHook) {
    if (
      (executionContext & CommitContext) !== NoContext &&
      root === rootCommittingMutationOrLayoutEffects
    ) {
      if (fiber.mode & ProfileMode) {
        let current = fiber;
        while (current !== null) {
          if (current.tag === Profiler) {
            const {id, onNestedUpdateScheduled} = current.memoizedProps;
            if (typeof onNestedUpdateScheduled === 'function') {
              onNestedUpdateScheduled(id);
            }
          }
          current = current.return;
        }
      }
    }
  }

  // TODO: Consolidate with `isInterleavedUpdate` check
  if (root === workInProgressRoot) {
    // Received an update to a tree that's in the middle of rendering. Mark
    // that there was an interleaved update work on this root. Unless the
    // `deferRenderPhaseUpdateToNextBatch` flag is off and this is a render
    // phase update. In that case, we don't treat render phase updates as if
    // they were interleaved, for backwards compat reasons.
    if (
      deferRenderPhaseUpdateToNextBatch ||
      (executionContext & RenderContext) === NoContext
    ) {
      workInProgressRootUpdatedLanes = mergeLanes(
        workInProgressRootUpdatedLanes,
        lane,
      );
    }
    if (workInProgressRootExitStatus === RootSuspendedWithDelay) {
      // The root already suspended with a delay, which means this render
      // definitely won't finish. Since we have a new update, let's mark it as
      // suspended now, right before marking the incoming update. This has the
      // effect of interrupting the current render and switching to the update.
      // TODO: Make sure this doesn't override pings that happen while we've
      // already started rendering.
      markRootSuspended(root, workInProgressRootRenderLanes);
    }
  }

  if (lane === SyncLane) {
    if (
      // Check if we're inside unbatchedUpdates
      (executionContext & LegacyUnbatchedContext) !== NoContext &&
      // Check if we're not already rendering
      (executionContext & (RenderContext | CommitContext)) === NoContext
    ) {
      // This is a legacy edge case. The initial mount of a ReactDOM.render-ed
      // root inside of batchedUpdates should be synchronous, but layout updates
      // should be deferred until the end of the batch.
      performSyncWorkOnRoot(root);
    } else {
      ensureRootIsScheduled(root, eventTime);
      if (
        executionContext === NoContext &&
        (fiber.mode & ConcurrentMode) === NoMode
      ) {
        // Flush the synchronous work now, unless we're already working or inside
        // a batch. This is intentionally inside scheduleUpdateOnFiber instead of
        // scheduleCallbackForFiber to preserve the ability to schedule a callback
        // without immediately flushing it. We only do this for user-initiated
        // updates, to preserve historical behavior of legacy mode.
        resetRenderTimer();
        flushSyncCallbacksOnlyInLegacyMode();
      }
    }
  } else {
    // Schedule other updates after in case the callback is sync.
    ensureRootIsScheduled(root, eventTime);
  }

  return root;
}
```



### checkForNestedUpdates

这个其实就是我们平常写了无限更新循环的警告和打断。有没有感觉这些文字的警告似曾相识！

```typescript
function checkForNestedUpdates() {
  if (nestedUpdateCount > NESTED_UPDATE_LIMIT) {
    nestedUpdateCount = 0;
    rootWithNestedUpdates = null;
    invariant(
      false,
      'Maximum update depth exceeded. This can happen when a component ' +
        'repeatedly calls setState inside componentWillUpdate or ' +
        'componentDidUpdate. React limits the number of nested updates to ' +
        'prevent infinite loops.',
    );
  }

  if (__DEV__) {
    if (nestedPassiveUpdateCount > NESTED_PASSIVE_UPDATE_LIMIT) {
      nestedPassiveUpdateCount = 0;
      console.error(
        'Maximum update depth exceeded. This can happen when a component ' +
          "calls setState inside useEffect, but useEffect either doesn't " +
          'have a dependency array, or one of the dependencies changes on ' +
          'every render.',
      );
    }
  }
}
```



### markUpdateLaneFromFiberToRoot

```typescript
function markUpdateLaneFromFiberToRoot(
  sourceFiber: Fiber,
  lane: Lane,
): FiberRoot | null {
  // Update the source fiber's lanes
  // 更新lanes, 更新当前fiber 的lines
  sourceFiber.lanes = mergeLanes(sourceFiber.lanes, lane);
  // 旧的fiber 
  let alternate = sourceFiber.alternate;
  if (alternate !== null) {
    alternate.lanes = mergeLanes(alternate.lanes, lane);
  }
  if (__DEV__) {
    if (
      alternate === null &&
      (sourceFiber.flags & (Placement | Hydrating)) !== NoFlags
    ) {
      warnAboutUpdateOnNotYetMountedFiberInDEV(sourceFiber);
    }
  }
  // Walk the parent path to the root and update the child lanes.
  // 当前的fiber
  let node = sourceFiber;
  // 上级fiber
  let parent = sourceFiber.return;
  // 递归父节点合并childrenLanes
  // child（第一个子节点）、sibling（兄弟节点）、return（父节点）
  // 可以回头看第三章
  while (parent !== null) {
    parent.childLanes = mergeLanes(parent.childLanes, lane);
    alternate = parent.alternate;
    if (alternate !== null) {
      alternate.childLanes = mergeLanes(alternate.childLanes, lane);
    } else {
      if (__DEV__) {
        if ((parent.flags & (Placement | Hydrating)) !== NoFlags) {
          warnAboutUpdateOnNotYetMountedFiberInDEV(sourceFiber);
        }
      }
    }
    node = parent;
    parent = parent.return;
  }
  // export const HostRoot = 3; // Root of a host tree. Could be nested inside another node.
  // 判断是否为根节点
  if (node.tag === HostRoot) {
    const root: FiberRoot = node.stateNode;
    return root;
  } else {
    return null;
  }
}

```



### addFiberToLanesMap

```typescript
export function addFiberToLanesMap(
  root: FiberRoot,
  fiber: Fiber,
  lanes: Lanes | Lane,
) {
  // 设置updater追踪
  if (!enableUpdaterTracking) {
    return;
  }
  // 是否是开发环境
  if (!isDevToolsPresent) {
    return;
  }
  // // 取得等待updater的Lane的Map
  const pendingUpdatersLaneMap = root.pendingUpdatersLaneMap;
  while (lanes > 0) {
    const index = laneToIndex(lanes);
    const lane = 1 << index;
		// 循环取得updater，并把fiber压入map
    const updaters = pendingUpdatersLaneMap[index];
    updaters.add(fiber);
		// 剔除循环过的车道
    lanes &= ~lane;
  }
}
```

