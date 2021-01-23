---
title: 【重识前端】React-Router源码阅读
date: 2020-11-14 20:29:17
tags: [源码阅读, React]
category: [重拾前端]
cover: /image/cover/web.jpeg
---

# 前言

简单介绍一下我现在分析源码是GitHub上面最新（最后提交日期为：2020-10-19）的master的代码.依赖的history是`4.9.0`的

# 学前小知识

`React-Router`其实最核心的东西是`Route`组件和由统一作者开发的`History`库来建立的。接下来跟着镜头一起走进神秘的`ßReact-Router`世界吧

