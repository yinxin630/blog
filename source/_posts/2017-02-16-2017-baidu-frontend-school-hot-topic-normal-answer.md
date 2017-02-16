---
layout: post
author: 碎碎酱
title: 2017百度前端学院热身题非"主流"答案
permalink: 2017-baidu-frontend-school-hot-topic-normal-answer
date: 2017-02-16
thumbnail: /img/javascript-logo.png
category: javascript
tags:
- javascript
- baidu
- debug
lede: ""
---

## 前言

今天注意到17年[百度前端学院](http://ife.baidu.com/)活动开放注册了, 作为一只入行一年的小前端, 希望能参与这次活动锻炼一下自己. 按照要求报名后回到首页, 看到有热身任务, 于是点开来玩玩.

但是... 看到第一题后傻眼, 原来是智力题而不是技术题, 😂 然而耗尽250的智商我也没能把题目解出来.

![图一](/img/2017-02-16-2017-baidu-frontend-school-hot-topic-normal-answer-01.png)

## 分析页面

于是, 智商不够技术来凑, 打开 dev tool, 注意到console有输出 `You are running Vue in development mode.`, 说明该页面是用 vue.js 写的单页应用, 观察 url, url末尾包含 #, 所以路由控制是用 vue-router 的 hash 模式.

打开 sources 页查看源文件, 项目应该是使用了 vue-cli 的 webpack 模板, build 生成了三个js文件, 分别为 `app-<hash>.js`, `manifest-<hash>.js`, `vendor-<hash>.js`, vendor 和 manifest 是项目基础框架和库文件, 我们只需要关注 app.js 即可.

## 分析源码

首先格式化代码, 得到1641行的代码. 搜索第一题的文案内容: `一笔画出折线`, 找到位于1612行的位置, 向上看找到该代码块开头的地方, 观察1555 ~ 1618行, 可以推断出这里是一个 vue componet, 并且它就是我们看到的第一关, 代码如下: 

```javascript
function(t, e) {
    t.exports = {
        render: function() {
            var t = this
              , e = t.$createElement
              , s = t._self._c || e;
            return s('div', {
                staticClass: 'index-view'
            }, [t._m(0), t._v(' '), s('div', {
                staticClass: 'points-container',
                attrs: {
                    id: 'points-container'
                },
                on: {
                    mousemove: t.mousemove,
                    click: t.click,
                    contextmenu: t.contextmenu
                }
            }, [s('svg', [s('polyline', {
                attrs: {
                    points: t.linePath
                }
            })]), t._v(' '), t._l(t.points, function(t) {
                return s('i', {
                    style: {
                        left: t.x + 'px',
                        top: t.y + 'px'
                    }
                });
            })], 2), t._v(' '), s('p', {
                staticClass: 'note'
            }, [t._v('请使用pc浏览器打开，chrome / firefox / safari / ie9+')]), t._v(' '), s('alert', {
                attrs: {
                    title: t.alertTitle,
                    show: t.showAlert
                },
                on: {
                    click: function(e) {
                        t.showAlert = !1;
                    }
                }
            }), t._v(' '), s('alert', {
                attrs: {
                    title: 'well done!',
                    show: t.showSuccess
                },
                on: {
                    click: t.doNext
                }
            })], 1);
        },
        staticRenderFns: [function() {
            var t = this
              , e = t.$createElement
              , s = t._self._c || e;
            return s('h3', [s('b', {
                staticClass: 'title-tip'
            }, [t._v('Unlock')]), t._v('一笔画出折线，穿过图中的9个点，'), s('b', {
                staticStyle: {}
            }, [t._v('折线个数尽量少')]), t._v('。（按`enter`确定，按`右键`重新画线，按`esc`取消上一条线段）')]);
        }
        ]
    };
}
```

但是, 该部分代码仅仅是UI渲染部分, 并不包含任务成功失败的逻辑判断, 我们再尝试搜索下答错题目后显示的文案内容: `还可以用更少的笔画`, 位于328行, 216 ~ 363就是第一关的逻辑判断部分, 其中我们重点关注 computeThrough 这个方法, 代码如下:

```javascript
computeThrough: function() {
    var t = this
        , e = this;
    setTimeout(function() {
        // 判断线段条数
        if (t.lines.length > 5)
            return void e.showMessage('还可以用更少的笔画，再接再厉哦！');
        for (var s = 0, n = t.lines.length - 1, a = t.points.slice(), r = t.lines; n > 0; ) {
            for (var o = r[n], c = r[n - 1], l = a.length - 1; l >= 0; l--) {
                var u = a[l]
                    , f = Math.pow((c.y - o.y) * u.x + (o.x - c.x) * u.y + (c.x * o.y - o.x * c.y), 2) / (Math.pow(c.y - o.y, 2) + Math.pow(o.x - c.x, 2));
                f <= 64 && i(u, o, c) && (a.splice(l, 1),
                s++);
            }
            n--;
        }
        // 判断穿过的点的个数
        s >= 9 ? (astorage.setItem('startTime-0', Date.now() - t.startTime),
        t.showSuccess = !0) : s ? e.showMessage('不错呦，穿过了' + s + '个点！') : e.showMessage('瞎了。。。');
    }, 100);
},
```

观察代码逻辑后, 我们分别在 `if (t.lines.length > 5)` 和 `s >= 9 ? (astorage.setItem('startTime-0', Date.now() - t.startTime),` 的位置下断点, 然后回到题目再次提交答案. 果然代码执行到了我们刚才设置的第一个断点位置, 如下图所示:

![图二](/img/2017-02-16-2017-baidu-frontend-school-hot-topic-normal-answer-02.png)

`lines` 是一个 vue 组件实例的 data, 因为其是一个数组类型, 我们修改它的 length 为0, 继续执行代码

![图三](/img/2017-02-16-2017-baidu-frontend-school-hot-topic-normal-answer-03.png)

由于不满足条件, 代码和预料的一样, 停留在第二个断点位置, 这次我们修改 s 的值为233. 然后继续向后执行, 跳出 well done! 弹窗, 证明我们做的事情是对的!

## 后续题目

### 第二题

找出下一关的地址, 这种题目一定是把地址隐藏在了页面里, 直接查看dom即可找出. 不过, 由于页面是 vue 写的, 刚好我又安装了 vue debug 插件, 打开插件可以看到如下内容:

![图四](/img/2017-02-16-2017-baidu-frontend-school-hot-topic-normal-answer-04.png)

地址拿到了 `Iy9CQzY3QTA4MzM1NkM4MzU5NEJDQzU3RUQxMzNEN0EwMjM2N0U0Qg==`, 但是我们直接拿该地址替换url中的hash部分的话, 页面显示 `当前URL已失效，请返回首页重新进行闯关。`, 沃特? 什么鬼? 我想百度的童鞋应该不会搞错题目的, 我们再来观察下拿到的地址, 它比第二关的hash长度多了一部分, 并且末尾以==结尾, 这似乎很像是常用的 bash64 编码的格式啊! 打开 base64 解码工具, 反解得到真正的地址 `BC67A083356C83594BCC57ED133D7A02367E4B`, 拼接地址并访问.

### 第三题

观察代码, 每个题目的代码是从上往下排列的, 我们查看第一题后面的第二个function, 得出该题判断条件位于434行, 我们修改 a/c/i/l/r/u 六个变量均为0即可.

```javascript
this.timer = setInterval(function() {
    var n = $(window).height() + ''
        , i = -n.slice(0, 1)
        , a = -n.slice(1, 2)
        , r = -n.slice(2)
        , o = $('.lock-panel')
        , c = parseInt(o.find('.n1').css('top')) - e
        , l = parseInt(o.find('.n2').css('top')) - e
        , u = parseInt(o.find('.n3').css('top')) - e;
    // 判断条件
    i * s > c - 10 && i * s < c + 10 && a * s > l - 10 && a * s < l + 10 && r * s > u - 10 && r * s < u + 10 && (astorage.setItem('startTime-2', Date.now() - t.startTime),
    t.showSuccess = !0);
}, 1e3);
```

### 第四题

该题判断条件在487行, 直接修改 s.length 值为0即可

```javascript
fgarefa: function() {
    var t = $('.svg-panel')
        , e = t.offset()
        , s = '109 580 280 660 185 664 390.4 807 95 872 294 824'.split(' ').map(function(t) {
            return +t;
        })
        , i = [];
    t.find('i').each(function(t, s) {
        var n = $(s).offset();
        i.push(n.top - e.top),
        i.push(n.left - e.left);
    });
    // 判断条件
    for (var a = 0; a < s.length; a++)
        if (Math.abs(i[a] - s[a]) > 2)
            return void (this.balabal = '');
    this.balabal = '#/' + n.maps.d;
}
```

### 第五题

观察569 ~ 590行, 代码如下:

```javascript
this.ball.on('start', function(t) {
    s.css({
        left: t.x,
        top: t.y
    });
}).on('move', function(t) {
    s.css({
        left: t.x,
        top: t.y
    });
}).on('impact', function() {
    n.showMessage('撞车了。。。。');
}).on('outbound', function() {
    n.showMessage('出界了。。。。');
}).on('star', function(t) {
    n.stars.splice(t.starIndex, 1);
}).on('success', function(t) {
    return t.starCount < 3 ? void n.showMessage('至少要`3`个星星呦。。。') : t.time > 6e4 ? void n.showMessage('跑的比蜗牛还慢，该减减肥了。。。') : (astorage.setItem('startTime-4', Date.now() - this.startTime),
    astorage.setItem('step-4-stars', t.starCount),
    n.successTitle = '很牛逼啊，吃了' + t.starCount + '个星星，耗时' + Math.round(t.time / 1e3) + '秒',
    void (n.showSuccess = !0));
}),
```

该题成功失败是基于事件的, 所以我们搜索 success 事件, 看看是在哪里触发的. 得到代码如下:

```javascript
// 判断逻辑
if (Math.abs(this.x - this.endX) < 10 && Math.abs(this.y - this.endY) < 10)
return this.stop(),
this.emit('success', {
    // 判断逻辑
    starCount: this.starCount,
    time: this.endTime - this.startTime
}),
!0;
```

判断条件位于1037行和1040行, 我们修改 this.x / this.endX / this.y / this.endY 的值均为0, 修改 this.starCount 的值为233即可.

## 胜利

顺利通关 2333, 心满意足. 对[百度前端学院](http://ife.baidu.com/)有兴趣的小朋友们, 一起来玩耍吧~~ (免费广告, 去百度面试过两次前端不要俺~~)
