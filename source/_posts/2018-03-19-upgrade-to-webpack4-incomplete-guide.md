---
layout: post
author: 碎碎酱
title: 升级到webpack4不完全指南
permalink: upgrade-to-webpack4-incomplete-guide
date: 2018-03-19
thumbnail: /img/webpack.png
category: javascript
tags:
- webpack
- javascript
lede: 
---

webpack在2018年2月25号发布最新的[4.0版本](https://github.com/webpack/webpack/releases/tag/v4.0.0), 更新了以下内容:

* 性能大幅提升(没感觉..)
* 新增mode配置, 可选择设置为development和production, 实现零配置启动(意义不大...)
* 废弃CommonsChunkPlugin, 使用optimize.splitChunks和optimization.runtimeChunk替代
* 支持WebAssembly
* 支持CommonJS, AMD, ESM等模块系统, 可以直接导入.mjs扩展名的模块文件, 对wasm模块也有实验性的支持

今天我将自己维护的React脚手架从webpack2升级到了4, 记录一下需要改动的地方. 本篇是"不完全指南", 欢迎补充更多内容

## 升级依赖版本

* webpack: 2.2.1 -> 4.1.1
* webpack-dev-middleware: 1.10.0 -> 3.0.1
* html-webpack-plugin: 2.28.0 -> 3.0.6
* extract-text-webpack-plugin: 2.0.0 -> 4.0.0-beta.0
* copy-webpack-plugin: 4.0.1 -> 4.5.1
* react-hot-loader: 3.0.0-beta.7 -> 4.0.0

## webpack config添加mode字段

```js
// webpack.config.js
module.exports = {
    mode: 'development' // 或者 production
}
```

## 删除UglifyJsPlugin插件

在mode: production下, 会自动进行代码压缩. 可以修改配置项 `optimization.minimize: 'on'/'off'` 控制是否启用, 修改配置项 `optimization.minimizer: {}` 调整默认行为

## 删除CommonsChunkPlugin插件

替代为配置项 `optimization.splitChunks`, 可配置内容如下:

### 最简单的配置
```js
optimization: {
    splitChunks: {
        chunks: 'all',
    },
}
```

### 手动定制vendor内容
```js
module.exports = {
    entry: {
        vendor: ['react', 'react-dom', 'redux']
    },
    optimization: {
        splitChunks: {
            cacheGroups: {
                vendor: {
                    chunks: 'initial',
                    name: 'vendor',
                    test: 'vendor',
                    enforce: true,
                },
            },
        },
    },
}
```

### 字符串或者正则匹配控制vendor内容
```js
module.exports = {
    entry: {
        vendor: ['react', 'react-dom', 'redux']
    },
    optimization: {
        splitChunks: {
            cacheGroups: {
                vendor: {
                    chunks: 'initial',
                    name: 'vendor',
                    test: path.resolve(__dirname, '../node_modules'), // 或者 test: /node_modules/
                    enforce: true,
                },
            },
        },
    },
}
```

### 函数控制vendor内容
```js
module.exports = {
    entry: {
        vendor: ['react', 'react-dom', 'redux']
    },
    optimization: {
        splitChunks: {
            cacheGroups: {
                vendor: {
                    chunks: 'initial',
                    name: 'vendor',
                    test: module => /node_modules/.test(module.context),
                    enforce: true,
                },
            },
        },
    },
}
```

## react-hot-loader

原来的写法是:
```js
// index.js
import React from 'react';
import ReactDom from 'react-dom';
import { AppContainer } from 'react-hot-loader';
import App from './App';

const render = (Component) => {
    ReactDom.render(
        <AppContainer>
            <Component />
        </AppContainer>,
        document.getElementById('app'),
    );
};
render(App);

if (module.hot) {
    module.hot.accept('./App', () => { render(App); });
}

// App.jsx
import React, { Component } from 'react';
export default class App extends Component {
    render() {
        return <div>App</div>
    }
}
```

现在改为:
```js
// index.js
import React from 'react';
import ReactDom from 'react-dom';
import App from './App';

ReactDom.render(
    <App />,
    document.getElementById('app'),
);

// App.jsx
import React, { Component } from 'react';
import { hot } from 'react-hot-loader';
import Hello from './components/Hello';

class App extends Component {
    render() {
        return <div>App</div>
    }
}

export default hot(module)(App);
```

还有, 在webpack `html-webpack-plugin-after-emit`事件回调中, 删除 `hotMiddleware.publish({ action: 'reload' });` 语句

## 引用内容  

> [webpack@4.0.0 release log](https://github.com/webpack/webpack/releases/tag/v4.0.0)  
> [webpack 4.0.0-alpha.5 feedback #6357](https://github.com/webpack/webpack/issues/6357)