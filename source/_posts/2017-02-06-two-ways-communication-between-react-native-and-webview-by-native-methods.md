---
layout: post
author: 碎碎酱
title: 丢掉烦人的webview-bridge, 使用react-native原生方法与webview双向通讯
permalink: two-ways-communication-between-react-native-and-webview-by-native-methods
date: 2017-02-06
thumbnail: /img/react-native-logo.png
category: react-native
tags:
- react-native
lede: "react native的WebView组件在v0.37版本之前, 并没有与react native通讯的方法, 我们通常使用react-native-webview-bridge这个组件来实现双端通讯, 奈何react native版本更新飞快, 而该组件的作者通常并不能及时跟进升级, 给项目的进展造成了很多麻烦."
---

react native的WebView组件在`v0.37`版本之前, 并没有与react native通讯的方法, 我们通常使用[react-native-webview-bridge](https://github.com/alinz/react-native-webview-bridge)这个组件来实现双端通讯, 奈何react native版本更新飞快, 而该组件的作者通常并不能及时跟进升级, 给项目的进展造成了很多麻烦.

根据react native的pull request: [Implement a postMessage function and an onMessage event for webviews …](https://github.com/facebook/react-native/pull/9762), 从`v0.37`版本开发, 添加了`postMessage` api用于双端通讯.

## 使用postMessage

我使用的react native版本是`v0.40`

```javascript
// react-native 代码
/**
 * Sample React Native App
 * https://github.com/facebook/react-native
 * @flow
 */

import React, { Component } from 'react';
import {
  AppRegistry,
  StyleSheet,
  Text,
  View,
  TouchableHighlight,
  WebView
} from 'react-native';

export default class WebviewDataCommunicate extends Component {
  constructor(props) {
    super(props);
    this.state = {
      webViewData: ''
    };
    this.data = 0;
    this.sendMessage = this.sendMessage.bind(this);
    this.handleMessage = this.handleMessage.bind(this);
  }
  sendMessage() {
    this.webview.postMessage(++this.data);
  }
  handleMessage(e) {
    this.setState({ webViewData: e.nativeEvent.data });
  }
  render() {
    return (
      <View style={styles.container}>
        <TouchableHighlight
          style={styles.button}
          onPress={this.sendMessage}
        >
          <Text>发送数据到WebView</Text>
        </TouchableHighlight>
        <View>
          <Text>来自WebView的数据: <Text>{ this.state.webViewData }</Text></Text>
        </View>
        <WebView
          style={styles.webview}
          source={require('./index.html')}
          ref={webview => this.webview = webview}
          onMessage={this.handleMessage}
        />
      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#F5FCFF',
    paddingTop: 40
  },
  button: {
    width: 150,
    height: 40,
    backgroundColor: 'blue',
    justifyContent: 'center',
    alignItems: 'center',
  },
  webview: {
    width: 250,
    height: 250
  }
});

AppRegistry.registerComponent('WebviewDataCommunicate', () => WebviewDataCommunicate);
```

```html
// webview 代码(index.html)
<!DOCTYPE html>
<html lang="en">
    <head>
        <title></title>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
    </head>
    <body>
        <div style="text-align: center">
            <button id="button">发送数据到react native</button>
            <p style="text-align: center">收到react native发送的数据: <span id="data"></span></p>
        </div>

        <script>
            var data = 0;
            function sendData(data) {
                if (window.originalPostMessage) {
                    window.postMessage(data);
                } else {
                    throw Error('postMessage接口还未注入');
                }
            }
            window.onload = function() {
                document.addEventListener('message', function(e) {
                    document.getElementById('data').textContent = e.data;
                });
                document.getElementById('button').onclick = function() {
                    data += 100;
                    sendData(data);
                }
            }
        </script>
    </body>
</html>
```

运行效果如下图所示:
<img src="/img/2017-02-06-two-ways-communication-between-react-native-and-webview-by-native-methods-01.png" alt="运行效果" style="max-width: 400px">

## 问题

从webview向react native传数据时, 有较明显的延迟感, 延迟时间不稳定, 简单测试大约0.1s ~ 1s.