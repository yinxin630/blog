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

该文件定义了 `createProvider` 函数(22行), 并创建默认 Provider 组件(60行)

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
```
`Provider` 组件将 store 保存在 this 中, 渲染 children 中的第一个组件, 在 context 中返回 store 和空的 subscription, connected 组件将通过 context 获取 store

## src/components/connectAdvanced.js

该文件定义了 `connectAdvanced` 函数(40行)

```js
export default function connectAdvanced(
    selectorFactory,
    {
        getDisplayName = name => `ConnectAdvanced(${name})`,
        methodName = 'connectAdvanced',
        renderCountProp = undefined,
        shouldHandleStateChanges = true,
        storeKey = 'store',
        withRef = false,
        ...connectOptions
    } = {},
) {
    return function wrapWithConnect(WrappedComponent) { }
}
```
`connectAdvanced` 的第一个参数是一个生成选择器的函数, 格式为 `(dispatch, options) => (state, props) => ({ key: value })`, 选择器决定了最终我们传递给 connected 组件的 props. 第二参数是一个对象, `getDisplayName` 与包装组件 displayName 相关, `renderCountProp` 如果设置为字符串, 会将包装组件渲染次数以它的值为 key 传到到被包装组件 props, `shouldHandleStateChanges` 决定组件是否响应 state 变化, `withRef` 决定组件是否提供接口用于获取被包装组件实例. `connectAdvanced` 返回了一个包装函数 `wrapWithConnect`, 该函数接受被包装组件为参数, 返回包装后的组件.

### wrapWithConnect

```js
function wrapWithConnect(WrappedComponent) {
    const wrappedComponentName = WrappedComponent.displayName ||
        WrappedComponent.name ||
        'Component';
    const displayName = getDisplayName(wrappedComponentName);
    const selectorFactoryOptions = {
        ...connectOptions,
        getDisplayName,
        methodName,
        renderCountProp,
        shouldHandleStateChanges,
        storeKey,
        withRef,
        displayName,
        wrappedComponentName,
        WrappedComponent,
    };
    class Connect extends Component {}
    Connect.WrappedComponent = WrappedComponent;
    Connect.displayName = displayName;
    Connect.childContextTypes = childContextTypes;
    Connect.contextTypes = contextTypes;
    Connect.propTypes = contextTypes;
    return hoistStatics(Connect, WrappedComponent);
};
```

`wrapWithConnect` 内定义了 `Connect` 组件, 并将 WrappedComponent 的静态方法复制到了 Connect 中. 我们重点关注 `Connect` 组件的定义

### Connect 组件

```js
class Connect extends Component {
    constructor(props, context) {
        super(props, context);

        this.version = version;
        this.state = {};
        this.renderCount = 0;
        this.store = props[storeKey] || context[storeKey];
        this.propsMode = Boolean(props[storeKey]);
        this.setWrappedInstance = this.setWrappedInstance.bind(this);

        this.initSelector();
        this.initSubscription();
    }
    getChildContext() {
        const subscription = this.propsMode ? null : this.subscription;
        return {
            [subscriptionKey]: subscription || this.context[subscriptionKey],
        };
    }
    componentDidMount() {
        if (!shouldHandleStateChanges) return;
        this.subscription.trySubscribe();
        this.selector.run(this.props);
        if (this.selector.shouldComponentUpdate) this.forceUpdate();
    }
    componentWillReceiveProps(nextProps) {
        this.selector.run(nextProps);
    }
    shouldComponentUpdate() {
        return this.selector.shouldComponentUpdate;
    }
    componentWillUnmount() {
        if (this.subscription) this.subscription.tryUnsubscribe();
        this.subscription = null;
        this.notifyNestedSubs = noop;
        this.store = null;
        this.selector.run = noop;
        this.selector.shouldComponentUpdate = false;
    }
    getWrappedInstance() {
        return this.wrappedInstance;
    }
    setWrappedInstance(ref) {
        this.wrappedInstance = ref;
    }
    initSelector() {
        const sourceSelector = selectorFactory(this.store.dispatch, selectorFactoryOptions);
        this.selector = makeSelectorStateful(sourceSelector, this.store);
        this.selector.run(this.props);
    }
    initSubscription() {
        if (!shouldHandleStateChanges) return;
        const parentSub = (this.propsMode ? this.props : this.context)[subscriptionKey];
        this.subscription = new Subscription(this.store, parentSub, this.onStateChange.bind(this));
        this.notifyNestedSubs = this.subscription.notifyNestedSubs.bind(this.subscription);
    }
    onStateChange() {
        this.selector.run(this.props);
        if (!this.selector.shouldComponentUpdate) {
            this.notifyNestedSubs();
        } else {
            this.componentDidUpdate = this.notifyNestedSubsOnComponentDidUpdate;
            this.setState(dummyState);
        }
    }
    notifyNestedSubsOnComponentDidUpdate() {
        this.componentDidUpdate = undefined;
        this.notifyNestedSubs();
    }
    isSubscribed() {
        return Boolean(this.subscription) && this.subscription.isSubscribed();
    }
    addExtraProps(props) {
        if (!withRef && !renderCountProp && !(this.propsMode && this.subscription)) return props;
        const withExtras = { ...props,
        };
        if (withRef) withExtras.ref = this.setWrappedInstance;
        if (renderCountProp) withExtras[renderCountProp] = this.renderCount++;
        if (this.propsMode && this.subscription) withExtras[subscriptionKey] = this.subscription;
        return withExtras;
    }
    render() {
        const selector = this.selector;
        selector.shouldComponentUpdate = false;

        if (selector.error) {
            throw selector.error;
        } else {
            return createElement(WrappedComponent, this.addExtraProps(selector.props));
        }
    }
}
```

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
