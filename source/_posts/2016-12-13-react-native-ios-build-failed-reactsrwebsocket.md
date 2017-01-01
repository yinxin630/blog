---
layout: post
author: 碎碎酱
title: react native ios 构建失败, 报错为 ** BUILD FAILED ** CompileC RCTSRWebSocket.o (1 failure)
permalink: react-native-ios-build-failed-reactsrwebsocket
date: 2016-12-13
thumbnail: /img/react-native-logo.png
category: react-native
tags:
- react-native
- ios
lede: "react native项目中启动ios报错, 报错信息为 ** BUILD FAILED ** CompileC RCTSRWebSocket.o (1 failure), 解决办法为"
---


## 问题表现

执行 `react-native run-ios` 报错, app构建失败.

```
** BUILD FAILED **


The following build commands failed:
	CompileC /Users/yinxin/Github/RNP/ios/build/Build/Intermediates/RCTWebSocket.build/Debug-iphonesimulator/RCTWebSocket.build/Objects-normal/x86_64/RCTSRWebSocket.o RCTSRWebSocket.m normal x86_64 objective-c com.apple.compilers.llvm.clang.1_0.compiler
(1 failure)
```

## 解决办法

用 xcode 打开项目.

打开文件 `项目 > Libraries > RCTWebSocket.xcodeproj > RCTSRWebSocket.m`, 找到 `SecRandomCopyBytes(kSecRandomDefault, keyBytes.length, keyBytes.mutableBytes);`(494行) 这一行, 在开头加上 `(void)`.  再找到 `SecRandomCopyBytes(kSecRandomDefault, sizeof(uint32_t), (uint8_t *)mask_key);`(1334行) 这一行, 同样在开头添加 `(void)`.

打开文件 `项目 > Libraries > React.xcodeproj > React > View > RCTScrollView.m`, 在150行末尾另起一行, 添加代码 `RCTRefreshControl *_refreshControl;`