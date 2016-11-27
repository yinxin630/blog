---
layout: post
author: 碎碎酱
title: MAC中使用MongoHub客户端连接远程mongoDB数据库
date: 2016-09-28
thumbnail: /img/mongodb-logo.jpg
category: software
tags:
- mongodb
lede: "MongoHub是一个开源, 免费, 易用的Mac系统mongoDB客户端. 比命令行环境提供更便利的操作."
---


MongoHub是一个开源, 免费, 易用的Mac系统mongoDB客户端. 比命令行环境提供更便利的操作.
[下载MongoHub](https://github.com/jeromelebel/MongoHub-Mac)

# 客户端设置

点击左下角+号添加数据库, 填写配置信息.
```
Name: 配置名
Address: 数据库地址:端口
User: 用户名
Password: 密码
```

# 服务器设置

## 开启远程连接

mongoDB默认是监听`127.0.0.1`的, 远程链接会拒绝访问. 编辑配置文件 `vim /etc/mongod.conf`, 修改`bind_ip = 127.0.0.1`, 添加你需要使用ip, 内网访问则填写LAN IP, 外网访问则填写PUBLIC IP, 修改完成后保存. 重启mongoDB服务 `sudo service mongod restart`.

## 创建mongoDB用户

执行`mongo`进入命令行模式, 进入admin数据库 `use admin`, 使用`db.createUser`创建一个用户, 代码如下:

```
db.createUser(
  {
    user: "username",
    pwd: "password",
    roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
  }
)
```

执行代码, 就创建了一个用户, 打开MongoHub使用该用户连接既可. 该用户可以访问所有数据库, 关于roles请查看: [https://docs.mongodb.com/v2.6/reference/built-in-roles/#userAdminAnyDatabase](https://docs.mongodb.com/v2.6/reference/built-in-roles/#userAdminAnyDatabase)