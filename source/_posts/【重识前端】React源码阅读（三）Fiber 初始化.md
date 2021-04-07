---
title: 【重识前端】React源码阅读（三）4千字告诉你 Fiber 初始化
date: 2021-04-06 14:08:31
tags: [react]
category: [重拾前端]
cover: /image/cover/react.png
---

# 前言

在第一章中，我们解读到ReactDOM.render之后就没有继续了。因为我看了一下设计到了Fiber的一些知识，所以在第二章补充了Fiber的基础知识之后，我将Fiber拆分成四块来讲

- 动机（或者说初衷
- 初始化-Fiber树的准备工作
- render-Fiber树的构建
- commit-Fiber树映射到DOM

现在是初始化阶段

我们现在直接进入Fiber的初始化。Action！

# 前情回顾

回顾一下之前学习JSX的时候，阅读到ReactDOM.render。如果你没有阅读过之前的文章，强烈建议你到我的主页去阅读我之前的博客。下面我们开始。

ReactDOM.render的源码。

```typescript
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
  
  // 这里面涉及到了fiber的一些架构，开始补坑
  return legacyRenderSubtreeIntoContainer(
    null,
    element,
    container,
    false,
    callback,
  );
}
```

最后return的这个函数有点意思。

# 源码攻读

## legacyRenderSubtreeIntoContainer

我们首先翻译一下这个函数的名字，进行驼峰拆分：

> Google翻译：legacy Render Subtree Into Container（旧版渲染子树放入容器

先别急着翻译。我们先看看源码。

```typescript
function legacyRenderSubtreeIntoContainer(parentComponent, children, container, forceHydrate, callback) {
  // container 对应的是我们传入的真实 DOM 对象
  var root = container._reactRootContainer;
  // 初始化 fiberRoot 对象
  var fiberRoot;
  // DOM 对象本身不存在 _reactRootContainer 属性，因此 root 为空
  if (!root) {
    // 若 root 为空，则初始化 _reactRootContainer，并将其值赋值给 root
    // legacyCreateRootFromDOMContainer 主要的功能是创建，并且进行一些“脏判断”。感兴趣的，我在后面也会详细给出
    root = container._reactRootContainer = legacyCreateRootFromDOMContainer(container, forceHydrate);
    // legacyCreateRootFromDOMContainer 创建出的对象会有一个 _internalRoot 属性，将其赋值给 fiberRoot
    fiberRoot = root._internalRoot;

    // 这里处理的是 ReactDOM.render 入参中的回调函数，你了解即可
    if (typeof callback === 'function') {
      var originalCallback = callback;
      callback = function () {
        var instance = getPublicRootInstance(fiberRoot);
        originalCallback.call(instance);
      };
    } // Initial mount should not be batched.
    // 进入 unbatchedUpdates 方法
    unbatchedUpdates(function () {
      updateContainer(children, fiberRoot, parentComponent, callback);
    });
  } else {
    // else 逻辑处理的是非首次渲染的情况（即更新），其逻辑除了跳过了初始化工作，与楼上基本一致
    fiberRoot = root._internalRoot;
    if (typeof callback === 'function') {
      var _originalCallback = callback;
      callback = function () {
        var instance = getPublicRootInstance(fiberRoot);
        _originalCallback.call(instance);
      };
    } // Update

    updateContainer(children, fiberRoot, parentComponent, callback);
  }
  return getPublicRootInstance(fiberRoot);
}
```

梳理一下整个流程，然后我们再一一看这些函数。

以下流程是指初始化，也就是`if/else`判断里面的`if`。而不是`else`，`else`是非首次渲染，走的更新流程，注释里面写了。

1. 创建`container._reactRootContainer`对象，并赋值给`root`
2. 将`root`上的`_inernalRoot`赋值给`fiberRoot`
3. 将`legacyRenderSubtreeIntoContainer`的一些参数和`fiberRoot`一起传给`updateContainer`
4. `将updateConatiner`作为回调函数传给`unbatechedUpdates`

ok，现在看不太懂没关系。我们将里面用到的函数一一解读，然后再回来补充整个流程。

## legacyCreateRootFromDOMContainer

大家现在一定会晕头转向，因为我看的时候也是这样，他的名字和函数名字取的太像了，我们需要给他们独自用自己的方式命名。我会写下Google翻译， 帮助大家取名字。

> Google翻译：legacy Create Root From DOM Container（从DOM容器创建根目录

看了翻译之后是不是焕然大悟？别急，我们先看看源码。

```typescript
function legacyCreateRootFromDOMContainer(
  container: Container,
  forceHydrate: boolean,
): RootType {
    // 如果forceHydrate是false，就会调用后面的方法。
  const shouldHydrate =
    forceHydrate || shouldHydrateDueToLegacyHeuristic(container);
  // First clear any existing content.
  if (!shouldHydrate) {
    let warned = false;
    let rootSibling;
    // 这个循环，是为了清理所有的内容，然后找到我们要渲染的最初的根div。通常我们的html会有一个
    // <div id="root"></div> 为的就是保障他的纯净。
    while ((rootSibling = container.lastChild)) {
      if (__DEV__) {
        if (
          !warned &&
          rootSibling.nodeType === ELEMENT_NODE &&
          (rootSibling: any).hasAttribute(ROOT_ATTRIBUTE_NAME)
        ) {
          warned = true;
          console.error(
            'render(): Target node has markup rendered by React, but there ' +
              'are unrelated nodes as well. This is most commonly caused by ' +
              'white-space inserted around server-rendered markup.',
          );
        }
      }
      container.removeChild(rootSibling);
    }
  }
  if (__DEV__) {
    if (shouldHydrate && !forceHydrate && !warnedAboutHydrateAPI) {
      warnedAboutHydrateAPI = true;
      console.warn(
        'render(): Calling ReactDOM.render() to hydrate server-rendered markup ' +
          'will stop working in React v18. Replace the ReactDOM.render() call ' +
          'with ReactDOM.hydrate() if you want React to attach to the server HTML.',
      );
    }
  }

  return createLegacyRoot(
    container,
    shouldHydrate
      ? {
          hydrate: true,
        }
      : undefined,
  );
}
```

里面又用到两个函数，我为大家召出来。

### shouldHydrateDueToLegacyHeuristic



```typescript
function getReactRootElementInContainer(container: any) {
  // 如果没有就返回null
  if (!container) {
    return null;
  }
  // 内部定义的一个常量
  // export const DOCUMENT_NODE = 9;看名字就顾名思义了
  // 如果是内部一个document节点就返回document节点
  // 感兴趣可以看看MDN的介绍：https://developer.mozilla.org/zh-CN/docs/Web/API/Document
  // 常量的值也有：https://developer.mozilla.org/zh-CN/docs/Web/API/Node/nodeType
  if (container.nodeType === DOCUMENT_NODE) {
    return container.documentElement;
  } else {
    // 否则返回第一个元素
    return container.firstChild;
  }
}

function shouldHydrateDueToLegacyHeuristic(container) {
  // 将container传入函数，进行判断类型，并得到节点（有可能为空，看注释
  const rootElement = getReactRootElementInContainer(container);
  // 强转为布尔值
  return !!(
    rootElement &&
    rootElement.nodeType === ELEMENT_NODE &&
    // export const ROOT_ATTRIBUTE_NAME = 'data-reactroot'; 
    rootElement.hasAttribute(ROOT_ATTRIBUTE_NAME)
  );
}
```

我们看到一个hasAttribute为"data-reactroot"。这是 React 在服务端渲染的时候加上去的。之前有说过Hydrate这个操作是服务端渲染的时候需要的东西。

> Google翻译：getReactRootElementInContainer（获取容器中的React根元素
>
> Google翻译：shouldHydrateDueToLegacyHeuristic（应当因传统启发而水合
>
> 说人话就是是否需要进行hydrate操作

所以`legacyCreateRootFromDOMContainer`这个函数是保障`container`的纯净。并且返回`createLegacyRoot`函数。ok继续看。

### createLegacyRoot

```typescript
function ReactDOMLegacyRoot(container: Container, options: void | RootOptions) {
  // 是不是看见老朋友了。_internalRoot。继续看下面的内容，我们来解析一下createRootImpl
  this._internalRoot = createRootImpl(container, LegacyRoot, options);
}

export function createLegacyRoot(
  container: Container,
  options?: RootOptions,
): RootType {
  // 又是熟悉的操作，返回某个东西
  return new ReactDOMLegacyRoot(container, options);
}
```

### createRootImpl

需要注意的是，我们通篇都是以非`hydrate`来看的，因为我们不是服务端渲染。但是我也会介绍如果有`hydrate`的内容。所以大家看的时候可以稍微带入一点去看，不然很容易走神！

```typescript
// 创建并返回一个fiberRoot
function createRootImpl(
  container: Container,
  tag: RootTag,
  options: void | RootOptions,
) {
  // Tag is either LegacyRoot or Concurrent Root
  // 判断是否为hydrate模式
  const hydrate = options != null && options.hydrate === true;
  const hydrationCallbacks =
    (options != null && options.hydrationOptions) || null;
  const mutableSources =
    (options != null &&
      options.hydrationOptions != null &&
      options.hydrationOptions.mutableSources) ||
    null;
  const strictModeLevelOverride =
    options != null && options.unstable_strictModeLevel != null
      ? options.unstable_strictModeLevel
      : null;
  // 创建一个fiberRoot
  const root = createContainer(
    container,
    tag,
    hydrate,
    hydrationCallbacks,
    strictModeLevelOverride,
  );
  // 给container附加一个内部属性用于指向fiberRoot的current属性对应的rootFiber节点
  markContainerAsRoot(root.current, container);

  const rootContainerElement =
    container.nodeType === COMMENT_NODE ? container.parentNode : container;
  listenToAllSupportedEvents(rootContainerElement);

  if (mutableSources) {
    for (let i = 0; i < mutableSources.length; i++) {
      const mutableSource = mutableSources[i];
      registerMutableSourceForHydration(root, mutableSource);
    }
  }

  return root;
}
```

为什么我在注释里面会加上「创建一个fiberRoot」，不是一个是创建一个container吗？我们看createContainer里面做了什么。



### createContainer

```typescript
export function createFiberRoot(
  containerInfo: any,
  tag: RootTag,
  hydrate: boolean,
  hydrationCallbacks: null | SuspenseHydrationCallbacks,
  strictModeLevelOverride: null | number,
): FiberRoot {
  // 通过FiberRootNode构造函数创建一个fiberRoot实例
  const root: FiberRoot = (new FiberRootNode(containerInfo, tag, hydrate): any);
  // 如果有，就补充进这个回调函数
  if (enableSuspenseCallback) {
    root.hydrationCallbacks = hydrationCallbacks;
  }

  // Cyclic construction. This cheats the type system right now because
  // stateNode is any.
  // 通过createHostRootFiber方法创建fiber tree的根结点，即rootFiber
  // fiber节点也会像DOM树结构一样形成一个fiber tree单链表树结构
  // 每个DOM节点或者组件都会生成一个与之对应的fiber节点，在后续的调和(reconciliation)阶段起着至关重要的作用
  const uninitializedFiber = createHostRootFiber(tag, strictModeLevelOverride);
  // 创建完rootFiber之后，会将fiberRoot的实例的current属性指向刚创建的rootFiber
  root.current = uninitializedFiber;
  // 同时rootFiber的stateNode属性会指向fiberRoot实例，形成相互引用
  uninitializedFiber.stateNode = root;

  if (enableCache) {
    const initialCache = new Map();
    root.pooledCache = initialCache;
    const initialState = {
      element: null,
      cache: initialCache,
    };
    uninitializedFiber.memoizedState = initialState;
  } else {
    const initialState = {
      element: null,
    };
    uninitializedFiber.memoizedState = initialState;
  }

  initializeUpdateQueue(uninitializedFiber);
	// 最后将创建的fiberRoot实例返回
  return root;
}

// 内部调用createFiberRoot方法返回一个fiberRoot实例
export function createContainer(
  containerInfo: Container,
  tag: RootTag,
  hydrate: boolean,
  hydrationCallbacks: null | SuspenseHydrationCallbacks,
  strictModeLevelOverride: null | number,
): OpaqueRoot {
  return createFiberRoot(
    containerInfo,
    tag,
    hydrate,
    hydrationCallbacks,
    strictModeLevelOverride,
  );
}
```

这里也比较简单，只是单纯的调用函数，然后返回。真正的秘密就在这些函数里面，耐心点继续看！黎明就在眼前



#### FiberRootNode

一个非常恐怖的内容。但是看名字也能猜个大概。还记得我们之前说过。一个fiber节点要存的内容还蛮多的。比如被暂停时的状态，恢复时要做的事情，优先级等等....这里我尽可能的去写注释。帮助大家还要我自己理解。

```typescript
function FiberRootNode(containerInfo, tag, hydrate) {
  //root 节点类型()
  this.tag = tag;
  //root根节点，render方法的第二个参数
  this.containerInfo = containerInfo;
  //在持久更新中会用到，不支持增量更新的平台，react-dom 是整个应用更新，所以用不到
  this.pendingChildren = null;
  //当前应用root节点对应的Fiber对象
  this.current = null;
  //缓存
  this.pingCache = null;
  //已经完成任务的FiberRoot对象，在commit(提交)阶段只会处理该值对应的任务
  this.finishedWork = null;
  //在任务被挂起的时候通过setTimeout设置的返回内容，用来下一次如果有新的任务挂起时清理还没触发的timeout
  this.timeoutHandle = noTimeout;
  //顶层 context 对象，只有主动调用renderSubtreeIntoContainer才会生效
  this.context = null;
  this.pendingContext = null;
  // 老朋友了，用来确定第一次渲染的时候是否需要注水
  this.hydrate = hydrate;
  //回调节点
  this.callbackNode = null;
  //回调属性
  this.callbackPriority = NoLane;
  // export const NoLanes: Lanes = 0b0000000000000000000000000000000;
  /*
  export function createLaneMap<T>(initial: T): LaneMap<T> {
    // Intentionally pushing one by one.
    // https://v8.dev/blog/elements-kinds#avoid-creating-holes
    const laneMap = [];
    for (let i = 0; i < TotalLanes; i++) {
      laneMap.push(initial);
    }
    return laneMap;
  }
  */
  //任务时间。由于该字段在未来会重构，当前我们不需要理解他。
  this.eventTimes = createLaneMap(NoLanes);
  // 过期时间。函数上面的注释有。export const NoTimestamp = -1;
  this.expirationTimes = createLaneMap(NoTimestamp);

  // 优先级的初始化
  this.pendingLanes = NoLanes;
  this.suspendedLanes = NoLanes;
  this.pingedLanes = NoLanes;
  this.mutableReadLanes = NoLanes;
  this.finishedLanes = NoLanes;

  this.entangledLanes = NoLanes;
  this.entanglements = createLaneMap(NoLanes);

  if (enableCache) {
    this.pooledCache = null;
    this.pooledCacheLanes = NoLanes;
  }

  if (supportsHydration) {
    this.mutableSourceEagerHydrationData = null;
  }

  if (enableSchedulerTracing) {
    this.interactionThreadID = unstable_getThreadID();
    this.memoizedInteractions = new Set();
    this.pendingInteractionMap = new Map();
  }
  if (enableSuspenseCallback) {
    this.hydrationCallbacks = null;
  }

  if (enableProfilerTimer && enableProfilerCommitHooks) {
    this.effectDuration = 0;
    this.passiveEffectDuration = 0;
  }

  if (__DEV__) {
    switch (tag) {
      case ConcurrentRoot:
        this._debugRootType = 'createRoot()';
        break;
      case LegacyRoot:
        this._debugRootType = 'createLegacyRoot()';
        break;
    }
  }
}
```



#### createHostRootFiber

通篇大论都是在判断mode类型。感兴趣的可以再深挖。这里我就略过了。眼睛已经看花了。。。

```typescript
export function createHostRootFiber(
  tag: RootTag,
  strictModeLevelOverride: null | number,
): Fiber {
  let mode;
  if (tag === ConcurrentRoot) {
    mode = ConcurrentMode;
    if (strictModeLevelOverride !== null) {
      if (strictModeLevelOverride >= 1) {
        mode |= StrictLegacyMode;
      }
      if (enableStrictEffects) {
        if (strictModeLevelOverride >= 2) {
          mode |= StrictEffectsMode;
        }
      }
    } else {
      if (enableStrictEffects && createRootStrictEffectsByDefault) {
        mode |= StrictLegacyMode | StrictEffectsMode;
      } else {
        mode |= StrictLegacyMode;
      }
    }
  } else {
    mode = NoMode;
  }

  if (enableProfilerTimer && isDevToolsPresent) {
    // Always collect profile timings when DevTools are present.
    // This enables DevTools to start capturing timing at any point–
    // Without some nodes in the tree having empty base times.
    mode |= ProfileMode;
  }
	// export const HostRoot = 3;
  return createFiber(HostRoot, null, null, mode);
}
```



#### createFiber

也是包一个壳子。主要还是为了创建FiberNode。

```typescript
const createFiber = function(
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode,
): Fiber {
  // $FlowFixMe: the shapes are exact here but Flow doesn't like constructors
  return new FiberNode(tag, pendingProps, key, mode);
};

```



#### FiberNode

官方其实已经注释了属性的要素。但是我还是会尽可能的完善。

```typescript
function FiberNode(
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode,
) {
  // Instance
  // 用于标记fiber节点的类型
  this.tag = tag;
  // 用于唯一标识一个fiber节点
  this.key = key;
  // 节点类型
  this.elementType = null;
  // ReactNode的节点类型
  this.type = null;
  // FiberNode会通过stateNode绑定一些其他的对象，例如FiberNode对应的Dom、FiberRoot、ReactComponent实例
  this.stateNode = null;

  // Fiber
  // 表示父级 FiberNode
  this.return = null;
  // 表示第一个子 FiberNode
  this.child = null;
  // 表示紧紧相邻的下一个兄弟 FiberNode
  this.sibling = null;
  // 下标
  this.index = 0;

  this.ref = null;

  // 表示新的props
  this.pendingProps = pendingProps;
  // 表示经过所有流程处理后的新props
  this.memoizedProps = null;
  // 更新队列
  this.updateQueue = null;
  // 表示经过所有流程处理后的新state
  this.memoizedState = null;
  // 依赖
  this.dependencies = null;

  this.mode = mode;

  // Effects
  // 标志更新的类型：删除、新增、修改。
  this.flags = NoFlags;
  // 子Fiber树的标志更新的类型：删除、新增、修改。
  this.subtreeFlags = NoFlags;
  // 需要删除的内容
  this.deletions = null;

  // 优先级
  this.lanes = NoLanes;
  // 子的优先级
  this.childLanes = NoLanes;

  // 双缓冲：防止数据丢失，提高效率（之后Dom-diff的时候可以直接比较或者使用
  this.alternate = null;

  if (enableProfilerTimer) {
    // Note: The following is done to avoid a v8 performance cliff.
    //
    // Initializing the fields below to smis and later updating them with
    // double values will cause Fibers to end up having separate shapes.
    // This behavior/bug has something to do with Object.preventExtension().
    // Fortunately this only impacts DEV builds.
    // Unfortunately it makes React unusably slow for some applications.
    // To work around this, initialize the fields below with doubles.
    //
    // Learn more about this here:
    // https://github.com/facebook/react/issues/14365
    // https://bugs.chromium.org/p/v8/issues/detail?id=8538
    this.actualDuration = Number.NaN;
    this.actualStartTime = Number.NaN;
    this.selfBaseDuration = Number.NaN;
    this.treeBaseDuration = Number.NaN;

    // It's okay to replace the initial doubles with smis after initialization.
    // This won't trigger the performance cliff mentioned above,
    // and it simplifies other profiler code (including DevTools).
    this.actualDuration = 0;
    this.actualStartTime = -1;
    this.selfBaseDuration = 0;
    this.treeBaseDuration = 0;
  }

  if (__DEV__) {
    // This isn't directly used but is handy for debugging internals:
    this._debugID = debugCounter++;
    this._debugSource = null;
    this._debugOwner = null;
    this._debugNeedsRemount = false;
    this._debugHookTypes = null;
    if (!hasBadMapPolyfill && typeof Object.preventExtensions === 'function') {
      Object.preventExtensions(this);
    }
  }
}
```



#### initializeUpdateQueue

createContainer还有最后一个函数，初始化更新队列。

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



# 小结一番

大费周章的做这些阅读，其实回过神来看。就是为了创建`fiberNode`（`FiberRootNode实例`）和`rootFiber`对象。这两兄弟是整个Fiber树构建的起点。

- `fiberNode`（`FiberRootNode实例`） -> 的关联对象是真实 DOM 的容器节点
- `rootFiber` -> 虚拟 DOM 的根节点

然后再回过神来看，确实。他们各自创建都是为他们所代表的东西而创建，无论是属性或者打入方法。

# 收尾

最后他们都会被`unbatchedUpdates`所收编（看看最上面梦开始的地方「legacyRenderSubtreeIntoContainer」）。最后我们所创建的都会归入到它的手上。一起看看最后一个函数吧。

## 方便阅读

```typescript
unbatchedUpdates(function () {
  updateContainer(children, fiberRoot, parentComponent, callback);
});
```



## unbatchedUpdates

```typescript
function unbatchedUpdates(fn, a) {
  // 这里是对上下文的处理，不必纠结
  var prevExecutionContext = executionContext;
  executionContext &= ~BatchedContext;
  executionContext |= LegacyUnbatchedContext;
  try {
    // 重点在这里，直接调用了传入的回调函数 fn，对应当前链路中的 updateContainer 方法
    return fn(a);
  } finally {
    // finally 逻辑里是对回调队列的处理，此处不用太关注
    executionContext = prevExecutionContext;
    if (executionContext === NoContext) {
      // Flush the immediate callbacks that were scheduled during this batch
      resetRenderTimer();
      flushSyncCallbackQueue();
    }
  }
}
```



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
  enqueueUpdate(current, update);
  // 调度 fiberRoot 
  scheduleUpdateOnFiber(current, lane, eventTime);
  // 返回当前节点（fiberRoot）的优先级
  return lane;
}
```



updateContainer 的逻辑相对来说丰富了点，但大部分逻辑也是在干杂活，它做的最关键的事情可以总结为三件：

- 请求当前 Fiber 节点的 lane（优先级）；

- 结合 lane（优先级），创建当前 Fiber 节点的 update 对象，并将其入队；

- 调度当前节点（rootFiber）。

看到整理的伙伴，谢谢你的耐心。



# 最后

因为我也是在一边看，一边记录，如果有问题，大家一定要及时提出来。我也会及时更新的。这部分我应该会屡次阅读。毕竟大神写了几年，我不可能几天就看明白的。



# 索引

https://zh-hans.reactjs.org/docs









