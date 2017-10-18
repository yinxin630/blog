---
layout: post
author: 碎碎酱
title: react-redux 原理分析
permalink: react-redux-source-code-analyse
date: 2017-08-23
thumbnail: /img/react-logo.png
category: react
tags:
- javascript
- react
- redux
lede: 探究 react-redux 源码, 解密 Provide 和 connect 到底做了什么
featured: true
---

`react-redux` 的当前最新版本为 `v5.0.6`, 本文将会以该版本进行分析

`react-redux` 对外导出了四个接口 `Provider` `connect` `connectAdvanced` `createProvider`, 大多数情况我们只需要使用 `Provider` 和 `connect`

## Provider

`Provider` 是一个 react component, 接受 store 参数, 并将 store 实例放到 context 中, 因此它的子组件可以通过 connect 获取 store 中的 state

`Provider` 只会渲染第一个子组件, 如果你传递了多个子组件, 多余的会被丢弃

如果不使用 Provider, 你必须自己将 store 传递到 connected 组件, 在单元测试和不包含完整 react 的环境中可能会这样做

## connectAdvanced

`connectAdvanced` 是 connect 的基础, 用于创建自定义的 connect, 允许你自由的控制 state props 和 dispatch 组成最终传递到被包装组件的 props

`connectAdvanced` 接收两个参数, 第一个参数是一个生成选择器的函数, 格式为 `(dispatch, options) => (state, props) => ({ key: value })`, 选择器决定了最终我们传递给 connected 组件的 props. 第二参数是一个对象, `getDisplayName` 与包装组件 displayName 相关, `renderCountProp` 如果设置为字符串, 会将包装组件渲染次数以该字符串为 key 传到到被包装组件 props, `shouldHandleStateChanges` 决定组件是否响应 state 变化, `withRef` 决定组件是否提供接口用于获取被包装组件实例

`connectAdvanced` 返回了一个包装函数 `wrapWithConnect`, `wrapWithConnect` 接受被包装组件为参数, 返回包装后的组件, 并将被包装组件上定义的静态方法复制到包装组件上

`wrapWithConnect` 函数内包含 `Connect` 组件定义, `Connect` 组件就是我们调用 connect 后拿到的组件, 它负责订阅 store 变化. 当接收到 store 变化事件或者收到新的 props 时, `Connect` 组件会调用选择器生成 nextProps, 选择器就是 `connectAdvanced` 第一个参数生成的, `Connect` 组件将 nextProps 与当前 props 对比, 如果有变化就会保存新的 props, 并调用 setState 使其 rerender, 这样被包装组件就拿到了更新后的组件

`Connect` 组件通过 redux 提供的 `store.subscribe()` 接口订阅 store 变化, 同时它还将监听器实例放到了 context 中, 并提供了 api 供子组件添加自己的监听器, 当 props 变化时, 会逐个触发已订阅的监听器, 当然大多数情况我们不需要它

## connect

`connect` 用于将组件连接到 redux store, 它是对 connectAdvanced 的包装, 为大多数的使用场景提供了方便的 api. 它不会修改被连接的组件, 而是返回一个新的, 已连接store的, 保持原本功能的组件

`connect` 有四个参数, [mapStateToProps], [mapDispatchToProps], [mergeProps], [options]
* `[mapStateToProps]` 用于将 state 中仅被包装组件所需的部分挑出来
* `[mapDispatchToProps]` 用于定义一些具体 action 的函数
* `[mergeProps]` 决定了如何组织将被传递的 props
* `[options]` 是一些选项内容

## createProvider

`createProvider` 用于生成自定义 Provider 组件, 与 Provider 组件的区别是它允许你控制 store 在 context 中的 key 值, 在 Provide 中 key 的值为 "store". 你仅在存在多 store 时会需要它, 但多 store 是不被推荐的做法
