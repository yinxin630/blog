---
layout: post
author: 碎碎酱
title: react-native hybrid开发工作流, 如何优化向native交付方式和debug姿势
permalink: react-native-hybrid-develop-workflow-optimize
date: 2017-01-22
thumbnail: /img/react-native-logo.png
category: react-native
tags:
- react-native
lede: "我司移动端app采用native + react-native(以下简称RN)的hybrid开发架构, RN端编写一个个独立的模块, 交由native端调度执行. 我们原本的协作方式是, 由native端拉取RN项目代码, 因为ios和android配置RN package server方式不同, ios组是将RN部署在一台局域网机器上, 所有ios开发人员在项目中配置使用该机器运行的package server. 但是android配置package server是在app的dev菜单中的设置里面指定的, 所有android开发人员是每个人在本地跑一份package server"
---

## 痛点

我司移动端app采用native + react-native(以下简称RN)的hybrid开发架构, RN端编写一个个独立的模块, 交由native端调度执行. 我们原本的协作方式是, 由native端拉取RN项目代码, 因为ios和android配置RN package server方式不同, ios组是将RN部署在一台局域网机器上, 所有ios开发人员在项目中配置使用该机器运行的package server. 但是android配置package server是在app的dev菜单中的设置里面配置的, 所有android开发人员是每个人在本地跑一份package server.

在debug和发包的过程中, 经常出现下面这些情景:

0. native端想要发包, native童鞋: 发车啦, RN的朋友还有人要提交代码吗...
0. 运行测试包, native童鞋: 哎这地方怎么不对啊, 图片都没有. RN童鞋: 这个地方需要额外一些静态资源, 是不是没加上...
0. 测试童鞋: 哎这个问题你昨天不是说今天发包解决吗, 怎么还不行. RN童鞋: 我的提交到底打包进去了没...
0. RN童鞋要测试与native交互的接口, 去找ios童鞋, xxx帮我安装一个连我本地server的最新版app呗, android类似...

## 造成痛点的原因

* 打包过程交由native处理, native不知道哪些是需要额外处理的静态资源
* native需要自己拉RN代码, 自己跑server, server跑了很多份
* 发布测试包时, RN是打包进去的, 不能及时更新
* RN童鞋开发中只是跑单独模块, 交互的部分没法本地测试

## 解决痛点

### 打包的问题

RN的打包命令只能打包项目中require的内容, 比如项目中存在webview, 会有一些html/css/js/image资源, 这些需要自己手动处理. 
可以写一个配置化的静态资源处理脚本, 如下所示: 
```javascript
// bundleStatic.js
const path = require('path');
const copydir = require('copy-dir');
const dependences = require('./dependences.json');

const root = path.resolve(__dirname, '../../');
const dest = path.resolve(root, 'bundle/ios/assets');

console.log('start copy static dependences');
for (const dependence of dependences.list) {
  copydir(path.resolve(root, dependence), path.resolve(dest, dependence), (err) => {
    if (err) {
      console.log(err);
      return;
    }
    console.log(`"${dependence}" -> copy success`);
  });
}
```

配置静态资源目录:
```javascript
// dependences.json
{
    "list": [
        "htmlPages/testPage",
        "...更多内容"
    ]
}
```

在package.json中添加打包script:
```
"bundleIos": "react-native bundle --entry-file index.ios.js --platform ios --dev true --bundle-output ./bundle/ios/ios.dev.jsbundle --assets-dest ./bundle/ios --sourcemap-output ./bundle/ios/ios-source.map && node bundle/bin/bundleStatic.js"
```

由RN童鞋负责打包. 执行`npm run bundleIos`, 目录`/bundle/ios`将会包含完整打包内容.

### native访问package server和测试包不能及时更新问题

我们选择将打包的内容上传到七牛cdn, native端开发时和测试包均连接此cdn地址, 我们引入了CI系统, 使得打包和上传cdn的过程自动化完成. 保证他人始终能访问到最新的RN代码.

CI系统我们选择travis ci. 我司项目是托管在github上的私有项目, 在 `https://travis-ci.com/` 将项目启用CI构建, 并配置:
```yml
language: node_js
node_js:
- '6'
install:
- npm install
script:
- npm test
- npm run bundleIos
- npm run bundleAndroid
- npm run upload
```

在package.json中定义upload script:
```
"upload": "node bundle/bin/upload.js"
```

upload脚本代码:
```javascript
// upload.js
const qiniu = require('qiniu');
const path = require('path');
const fs = require('fs');
const promiseify = require('es6-promisify');

const readdir = promiseify(fs.readdir);
const stat = promiseify(fs.stat);

const config = {
    bucket: 'test',
    bucketUrl: 'ojwvxk714.bkt.clouddn.com',
};

qiniu.conf.ACCESS_KEY = process.env.accessKey;
qiniu.conf.SECRET_KEY = process.env.secretKey;

const extra = new qiniu.io.PutExtra();
const client = new qiniu.rs.Client();

function uptoken(bucket, key) {
    return new qiniu.rs.PutPolicy(`${bucket}:${key}`).token();
}

function uploadFile(key, localFile) {
    return new Promise((resolve, reject) => {
        qiniu.io.putFile(uptoken(config.bucket, key), key, localFile, extra, (err, ret) => {
            if (err) {
                reject(err);
            }
            resolve(ret);
        });
    });
}

function getFileInfo(key) {
    return new Promise((resolve, reject) => {
        client.stat(config.bucket, key, (err, ret) => {
            if (err) {
                if (err.error === 'no such file or directory') {
                    return resolve(null);
                }
                reject(err);
            }
            resolve(ret);
        });
    });
}

function deleteFile(key) {
    return new Promise((resolve, reject) => {
        client.remove(config.bucket, key, (err, ret) => {
            if (err) {
                reject(err);
            }
            resolve(ret);
        });
    });
}

function uploadDir(dir, prefix) {
    const list = fs.readdirSync(dir);
    for (const file of list) {
        const filePath = path.resolve(dir, file);
        const stat = fs.statSync(filePath);
        if (stat.isDirectory()) {
            uploadDir(filePath, prefix + file + '/');
        }
        else {
            const key = prefix + file;
            uploadFile(key, filePath).then(result => {
                console.log(`${key} --> success`);
            }).catch(err => {
                console.log(`${key} --> fail`);
                console.log(err);
            });
        }
    }
}

uploadDir(path.resolve(__dirname, '../ios'), '');
uploadDir(path.resolve(__dirname, '../android'), '');
```

注意: 打包的内容应直接放在cdn bucket根目录, 我最初将ios资源加上`ios/xxx`的前缀, 会使得bundle外的资源无法正确加载.

七牛的 ACCESS_KEY 和 SECRET_KEY 应该从环境变量传入, travis ci的设置页面可以配置环境变量.

通过CI系统, 可以在提交代码后自动化的打包并更新cdn资源. (CI build案例: [https://travis-ci.org/yinxin630/react-native-with-travis-ci/builds/194140382](https://travis-ci.org/yinxin630/react-native-with-travis-ci/builds/194140382))

### RN端不方便测试native接口的问题

RN端大多时候仅仅是查看接口返回的数据是否正确, 我们选择让RN童鞋把native端跑起来, 让RN端也可以跑完整的项目. 在接口联调初期, native童鞋可以直接连RN童鞋的server合作解决问题.
后续的小问题RN童鞋直接自己运行app并向native反馈问题即可, 不用每次都拿着手机让native童鞋装上最新app.

### 从bundle启动RN时, 报错信息不能直接显示源码位置

从bundle启动, 报错信息都是bundle中的位置, 不方便debug. 观察前面的打包脚本, 其中有`--sourcemap-output ./bundle/ios/ios-source.map`, 我们可以依靠bundle对应的source map来得到源码位置.
```javascript
// findSourceLocation.js
const sourceMap = require('source-map');
const fs = require('fs');
const path = require('path');
const commandLineArgs = require('command-line-args');

const optionDefinitions = [
  { name: 'line', alias: 'l', type: Number },
  { name: 'column', alias: 'c', type: Number }
];
const options = commandLineArgs(optionDefinitions);

fs.readFile(path.resolve(__dirname, '../ios/ios.source.map'), 'utf8', (err, data) => {
  const smc = new sourceMap.SourceMapConsumer(data);
  console.log(smc.originalPositionFor({
    line: options.line || 0,
    column: options.column || 0
  }));
});
```

执行 `node findSourceLocation.js -line 行数 -column 列数` 得到结果.

## 后话

这是我司碰到痛点后所能想到的更好的姿势, 不知道大家有没有什么黑科技能让工作流更舒服呢?