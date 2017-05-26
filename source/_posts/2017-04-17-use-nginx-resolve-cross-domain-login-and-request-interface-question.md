---
layout: post
author: 碎碎酱
title: 使用nginx解决开发时跨域登录和接口请求问题
permalink: use-nginx-resolve-cross-domain-login-and-request-interface-question
date: 2017-04-17
thumbnail: /img/nginx-logo.png
category: linux
tags:
- nginx
- linux
lede: "开发时, 我们通常从本地启动应用, 并访问非本机的后端接口, 因为会触发浏览器跨域限制"
---

在开发的时候, 我们通常在本地启动打包服务, 比如使用 `webpac-dev-server` 时我们从 `http://localhost:8080` 访问, 由于此时的域为 `localhost:8080`, 而后端接口的域为 `xxx.com:80`, 同时登录需要跳转到登录页, 登录后会在 `xxx.com:80` 域注入cookie, 但是 `localhost:8080` 并没有该cookie, 因此无法进入登录状态.

## 解决办法

修改 hosts 文件, 使 `xxx.com:80` 的任一没使用的子域指向 `localhost`.

比如:
```
127.0.0.1 test.xxx.com
```

配置 nginx, 代理该子域名, 并将接口请求转发到正确的后端地址, 接口请求均为 `/api`开头, 配置如下:

```
server {
    listen       80;
    server_name  test.xxx.com;

    location / {
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header   Host             $http_host;
        proxy_set_header   X-NginX-Proxy    true;
        proxy_set_header   Connection "";
        proxy_http_version 1.1;
        proxy_pass         http://localhost:8080;
    }
    location /api {
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header   Host             xxx.com;
        proxy_set_header   X-NginX-Proxy    true;
        proxy_set_header   Connection "";
        proxy_http_version 1.1;
        proxy_pass http://xxx.com;
    }
}
```

保存配置并重启 nginx, 从 `http://test.xxx.com` 访问本地开发服务, 可以正常访问后端接口和登录.

## chrome跨域请求插件

如果只是为了解决跨域请求接口的问题, 可以按照 chrome 插件 `Allow-Control-Allow-Origin`, 安装后并启用, 就可以正常的发起跨域请求了. 注意该插件默认对所有域名启用跨域, 这会导致 socket.io 创建连接失败, 因此推荐依照该插件语法规则, 针对特定的域启用. 注意该插件不能解决跳转页面的登录问题.