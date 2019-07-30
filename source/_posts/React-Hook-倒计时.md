---
title: React Hook 倒计时
date: 2019-07-30 21:01:24
tags: [ReactHook, React, 倒计时]
category: [React]
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

>本人自创自用 一个好用的AntDesign Form快速生成器
>https://github.com/DerrickTel/ant-design-form
>欢迎大家提意见,本人会不定期更新.有新的需求都可以给我提issue  
>能动动你的小手点个start就更棒啦~
