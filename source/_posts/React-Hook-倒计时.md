---
title: React Hook 倒计时
date: 2019-07-30 21:01:24
tags: [ReactHook, React, 倒计时]
category: [React]
cover: /image/cover/React.jpeg
---

```
useEffect(() => {
    setTimeout(() => {
      if (count > 0) {
        setCount((c: number) => c - 1); // ✅ 在这不依赖于外部的 `count` 变量
      }
    }, 1000);
  }, [count]); // ✅ 我们的 effect 不适用组件作用域中的任何变量
```

```
<button onClick={() => setCount(60)} >点我</button>
```

