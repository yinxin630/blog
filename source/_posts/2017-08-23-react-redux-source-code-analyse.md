---
layout: post
author: 碎碎酱
title: react-redux 源码分析
permalink: react-redux-source-code-analyse
date: 2017-08-23
thumbnail: /img/react-logo.png
category: react
tags:
- javascript
- react
- redux
---

`react-redux` 的当前最新版本为 `v5.0.6`, 本文将会以该版本进行分析.

## src/index.js

该文件是入口文件, 对外导出了四个接口 `Provider` `connect` `connectAdvanced` `createProvider`

* `Provider` 是一个组件, 接受 store 参数, 它的子组件可以通过 connect 获取 store 中的数据. 如果不使用 Provider, 你必须自己将 store 传递到 connected 组件, 在单元测试和不包含完整 react 的环境中可能会这样做
* `connect` 用于将组件连接到 redux store, 它是对 connectAdvanced 的包装, 为大多数的使用场景提供了方便的 api. 它不会修改被连接的组件, 而是返回一个新的, 已连接store的, 保持原本功能的组件
* `connectAdvanced` 是 connect 的基础, 允许你自由的控制 state props 和 dispatch 组成最终传递到被连接组件的 props
* `createProvider` 用于生成自定义 Provider 组件, 与上述 Provider 组件的区别是它允许你控制 store 在 context 中的 key 值, 在 Provide 中 key 的值为 "store". 你仅在存在多 store 时会需要它, 但多 store 是不被推荐的

## src/components/Provider.js

该文件定义了 `createProvider` 函数(22行), 创建默认 Provider 组件(60行)

```js
export function createProvider(storeKey = 'store', subKey) {
    const subscriptionKey = subKey || `${storeKey}Subscription`
    class Provider extends Component {}
    return Provider
}
export default createProvider()
```
`createProvider` 定义并返回 Provider 组件, 我们将目光放到 Provider 组件上

```js
class Provider extends Component {
    getChildContext() {
        return { [storeKey]: this[storeKey], [subscriptionKey]: null }
    }

    constructor(props, context) {
        super(props, context)
        this[storeKey] = props.store;
    }

    render() {
        return Children.only(this.props.children)
    }
}

if (process.env.NODE_ENV !== 'production') {
    Provider.prototype.componentWillReceiveProps = function (nextProps) {
    if (this[storeKey] !== nextProps.store) {
        warnAboutReceivingStore()
    }
    }
}

Provider.propTypes = {
    store: storeShape.isRequired,
    children: PropTypes.element.isRequired,
}
Provider.childContextTypes = {
    [storeKey]: storeShape.isRequired,
    [subscriptionKey]: subscriptionShape,
}
```
`Provider` 组件将 store 实例保存在 this 中, 渲染 children 中的第一个组件, 在 context 中返回 store 实例给子组件

## src/connect/connect.js

该文件定义了 `createConnect` 函数(40行), 创建默认 connect 函数(90行)

```js
export function createConnect({
    connectHOC = connectAdvanced,
    mapStateToPropsFactories = defaultMapStateToPropsFactories,
    mapDispatchToPropsFactories = defaultMapDispatchToPropsFactories,
    mergePropsFactories = defaultMergePropsFactories,
    selectorFactory = defaultSelectorFactory
} = {}) { }
```
`createConnect` 接受一个 object 参数, 并提供了一系列默认值, 

未完待续, 敬请期待~~~
