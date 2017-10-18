---
layout: post
author: 碎碎酱
title: pm2设置程序的运行环境
permalink: pm2-set-application-node-env
date: 2016-09-28
thumbnail: /img/linux-logo.png
category: software
tags:
- pm2
- linux
- node.js
lede: "本文介绍了如何设置pm2中运行应用的NODE_ENV环境变量."
---


使用`export NODE_ENV="production"`只能设置pm2的运行环境. 要设置pm2运行的应用的环境变量, 需要执行`pm2 start app.js --env production`