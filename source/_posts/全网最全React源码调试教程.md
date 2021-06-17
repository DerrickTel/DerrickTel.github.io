---
title: 全网最全React源码调试傻瓜式教程
date: 2020-06-17  21:41:06
tags: [React]
category: [React]
cover: /image/cover/react.png
---
# 前言

之前在阅读React源码的时候，想调试一下，然后debugger看看变量已经数据是怎么传递的，无奈。还要装这么多东西。踩了不少坑，记录一下。帮助大家少踩坑，也为自己做一个笔记。

# 准备工作

- [Node](https://nodejs.org/) v8.0.0+、[Yarn](https://yarnpkg.com/en/) v1.2.0+。
- 已安装 [JDK](https://www.oracle.com/technetwork/java/javase/downloads/index.html)。
- 你已安装 `gcc`（或者你在有必要安装编译器的情况下也不觉得麻烦），因为一些依赖可能得经过编译，而在 OS X，Xcode 命令行工具会帮你处理；在 Ubuntu，`apt-get install build-essential` 会安装所需的 package（其它 Linux 发行版的类似命令也有效）；在 Windows 上得做些额外步骤，请参考 [`node-gyp` 安装步骤](https://github.com/nodejs/node-gyp#installation)。
- 熟悉 Git。

- 一个可以运行React项目（本文采用create-react-app）

# 开始

## 1.获取源代码

1. 打开源码地址：https://github.com/facebook/react
2. 克隆到自己的本地
   1. 可以下载zip解压
   2. 也可以Fork到自己的GitHub然后Clone
      1. 个人建议是Fork，然后可以修改，添加注释等等。可以记录一下自己学习历程
   3. 也可以直接Clone

## 2. 建立依赖连接

1. 打开项目
2. 运行yarn安装依赖
3. 运行`yarn build react/index,react/jsx,react-dom/index,scheduler --type=NODE`重新打包
4. `cd build/node_modules/react`进入React这个包内部
5. `yarn link`建立连接
6. `cd 到原来的项目路径`返回，为了下一次进入react-dom做准备
7. `cd build/node_modules/react-dom`
8. `yarn link`

## 3.找到一个可以调试的React项目

这里我采用的是`create-react-app`

1. `npx create-react-app my-app`
2. `cd my-app`

## 4. 连接项目和React库

1. 删除刚刚建立的项目的`node_modules`中的`react`和`react-dom`
2. 打开React项目
3. `yarn link react react-dom`

## 5. 开始调试

1. 打开我们clone的项目

2. 找到想要调试的地方

   1. 这里改的地方是`packages/react-dom/src/client/ReactDOMLegacy.js`
   2. 内容如下

   ```diff
   export function render(
     element: React$Element<any>,
     container: Container,
     callback: ?Function,
   ) {
   +  console.log(222);
     if (__DEV__) {
       console.error(
         'ReactDOM.render is no longer supported in React 18. Use createRoot ' +
           'instead. Until you switch to the new API, your app will behave as ' +
           "if it's running React 17. Learn " +
           'more: https://reactjs.org/link/switch-to-createroot',
       );
     }
   
     invariant(
       isValidContainerLegacy(container),
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
     return legacyRenderSubtreeIntoContainer(
       null,
       element,
       container,
       false,
       callback,
     );
   }
   ```

3. 打开React源码的项目
4. 运行`yarn build react/index,react/jsx,react-dom/index,scheduler --type=NODE`重新打包
5. 打开我们创建的React项目
6. `yarn start`
7. 打开控制台

![console](/image/reactDebugger/console.png)

## 6. 后续

每次修改源代码都需要

1. 运行`yarn build react/index,react/jsx,react-dom/index,scheduler --type=NODE`重新打包
2. 刷新一下我们的页面
3. 确实很麻烦目前没有好的主意
   1. 可能这就是修行吧！

# 结语

有问题可以在下方留言，感觉有帮助。帮忙点个赞