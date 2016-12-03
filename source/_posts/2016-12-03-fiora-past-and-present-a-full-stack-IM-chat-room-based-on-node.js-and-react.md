---
layout: post
author: 碎碎酱
title: Fiora的前世今生 - 一个基于node.js和react的全栈IM聊天室
permalink: fiora-past-and-present-a-full-stack-IM-chat-room-based-on-node.js-and-react
date: 2016-12-03
thumbnail: /img/nodejs-logo.png
category: node.js
tags:
- node.js
- react
- fiora
lede: "这个项目是我一直坚持在做的, 现在这版是重写的第三版. 查看下commit历史, 第一版创建于15年11月4号, 距今一年过去了. 当时我刚开始学习html, css, js以及node, 就想着写个什么小玩意来练练手, 觉得聊天室是一个很容易让自己和用户参与进来的东西, 于是决定搞个小聊天室玩玩."
featured: true
---

## 先附上地址

在线地址: [fiora.suisuijiang.com](http://fiora.suisuijiang.com/)
源码地址: [yinxin630/fiora](https://github.com/yinxin630/fiora)

(注: 请使用pc端现代浏览器访问, 移动端版本仅在chrome和safari能用, 且功能落后过多)

## 大致的技术栈

后端使用 koa 提供一个简易的 http 服务器, 并将所有请求重定向到入口 index.html 处理. 使用 socket.io 前后端通讯, 后端 API 采用类似 Restful 风格的自定义格式接口, 数据库是 MongoDB, 使用 bluebird 和 generator 处理异步, 用户认证采用 jwt token 的方式.

前端基于 react 框架, 资源大部分在 cdn 上(背景图打包进了 js 里), 使用 immutable 和 redux 处理数据, react-router 提供路由, 未使用第三方 UI 库, 页面样式均使用 sass 编写.

## 功能

* 创建用户, 创建群组, 加入群组, 私聊, 群聊
* 文本, 图片(粘贴发送), 代码, url 等多种类型消息
* 支持炸弹 / 系统消息 / 翔 / 精灵球等多种恶搞消息(`xxx(username)`)
* 消息桌面通知, 声音提醒, 通知开关控制
* 用户信息修改, 头像修改, 表情收藏, 群公告修改
* 消息内容过滤, 消息长度限制, 消息发送频率限制
* 提供第三方消息的实现接口, 使用中间件系统修改原消息体, 针对消息体自定义渲染逻辑, 恶搞消息即依此实现

## 截图

![2016-12-03-fiora-past-and-present-a-full-stack-IM-chat-room-based-on-node.js-and-react-01](/img/2016-12-03-fiora-past-and-present-a-full-stack-IM-chat-room-based-on-node.js-and-react-01.png)

## 关于Fiora的前世今生

这个项目是我一直坚持在做的, 现在这版是重写的第三版. 查看下commit历史, 第一版创建于15年11月4号, 距今一年过去了. 当时我刚开始学习html, css, js以及node, 就想着写个什么小玩意来练练手, 觉得聊天室是一个很容易让自己和用户参与进来的东西, 于是决定搞个小聊天室玩玩.

最初的时候, 我使用简单的html和css构建了界面, 使用jquery来处理dom事件和页面交互, 并使用node + socket.io来搞定后端通讯. 程序跑了起来并且能用, 但是所有js代码都堆积在一个文件内了, 包括dom事件, 页面逻辑以及后端交互部分, 并且当时react框架十分热门, 我也抽时间学习了一下, 用react重构了项目, 仅仅使用了react, 没有redux, 没有react-router, 甚至没有webpack. 于是自然而然的, 我碰到了一些坑, 我去查询react的相关内容, 了解到redux等工具能解决我的痛点, 于是把这些工具库学习了一下, 并且决定重写这个项目.

2016年2月15日我开启了第二版的新坑, 这次我分割了项目把前后端分别创建了一个仓库. 前端继续使用react栈, 加上redux, webpack等库, 重写设计了UI页面, 并使用了AmuzeUI React的组件(后续换成了阿里的antd), 后端使用了sails框架(实习公司用的这个), 因为它封装了socket.io并且具备实用的数据库操作. 在这个基础上, 我渐渐的拓展着功能, 因为公司(换了一家实习)把前端切换到react技术栈, 我对react的使用越来越熟练, 深入, 了解了react的最佳实践. 由于技术栈上, 功能上, UI上, 代码结构上等各种各样的问题, 我于是决定把项目再重写一遍.

7月31号, 我又一次重开了坑.. 这次合并了前后端放在同一仓库, 并且mobile web和react-native app端也都放了进去, 以便直接复用除UI部分的代码. 重新设计了UI, 去掉了第三方组件, 全部自己来写, css方面改用sass, 前端除了原来的react, redux, react-router, webpack, 又加上了immutable, 用上了es7的装饰器语法, 加上了pureRender优化. 后端换用koa框架, 结合sockt.io, 用jwt token做了用户验证, 乱七八糟的一大堆东西, 就不一一细谈了.

这一年来, 我从不会html成长为一个合格的小前端, 收获颇丰, 没有了刚开始实习时候的迷茫(最初做的java实习). 关于这个项目有什么想聊的, 欢迎进来交流哦. 😆

感谢 [blackmiaool](https://github.com/blackmiaool) 为fiora提交的代码, 顺便推荐下他的图片爆炸效果jq插件([blackmiaool/jquery-image-explode](https://github.com/blackmiaool/jquery-image-explode))