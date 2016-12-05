---
layout: post
author: 碎碎酱
title: 使用redux-devtools来调试你的redux
permalink: use-redux-devtools-to-debug-redux
date: 2016-11-17
thumbnail: /img/react-native-logo.png
category: react-native
tags:
- react-native
- redux
- redux-devtool
lede: "redux-devtool是一个有效的调试redux的配套工具, 可以显示action列表, 可以在时间轴上自由控制action前进后退."
---


# 功能

* 显示所有action操作, 可以查看action的内容, action执行前后state的diff, action执行后的state状态
* 可以手动触发action
    ![2016-11-17-use-redux-devtools-to-debug-redux-01](/img/2016-11-17-use-redux-devtools-to-debug-redux-01.png)
* 可以在项目重新运行后, 保持上次的action时间轴
* 可以临时取消某个action, 去掉某个action, 重置整个store
* 可以在action时间轴上自由的前进/后退, 可以以0.5x/1x/2x速度重新执行action时间轴(由于我的测试代码路由没有保存到store, 导致action后退时组件仍然是当前组件, 因为没有了相应数据而报错, 所以建议将页面路由状态也保存在store中)
* 可以以图表的形式查看store, 并且可以直接查看任意节点的值, 省去了debug时打印store的麻烦.
    ![2016-11-17-use-redux-devtools-to-debug-redux-02](/img/2016-11-17-use-redux-devtools-to-debug-redux-02.png)

# 使用 (该方法适用所有使用redux的项目)

* 安装chrome插件 redux-devtools-extension, 直接在谷歌商店搜索redux-devtools
* 在代码中创建store时添加 remote-redux-devtools

    ```
    import devToolsEnhancer from 'remote-redux-devtools';
    export default createStore(reducers, devToolsEnhancer({ realtime: true, port: 8000 }));
    ```

* 点击redux-devtools插件, 选择 Open Remote Devtools, 启动应用, 插件中会按时间顺序显示所有触发的action (由于该库和插件默认通过remotedev.io通讯, 速度较慢, 可能需要等待较长时间才看到action列表)
    ![2016-11-17-use-redux-devtools-to-debug-redux-03](/img/2016-11-17-use-redux-devtools-to-debug-redux-03.png)
* 使用本地server加速通讯过程
    * 创建 server.js
    * `npm install remotedev-server`
    * 添加如下代码并运行 

        ```javascript
        var remotedev = require('remotedev-server'); 
        remotedev({ hostname: 'localhost', port: 8000 });
        ```

    * 在 devtool 中点击设置, 勾选 Use custom server, 设置 host 和 port
