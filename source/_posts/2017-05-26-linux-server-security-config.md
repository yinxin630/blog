---
layout: post
author: 碎碎酱
title: linux服务器安全性配置
permalink: linux-server-security-config
date: 2017-05-26
thumbnail: /img/linux-logo.jpg
category: linux
tags:
- linux
lede: ""
---

以下配置在 Ubuntu 16.04 中可以使用, 其它系统需要根据实际情况调整.

## 安全配置

### 用户
* 创建新用户并设置用户目录和所属组 `useradd -d [目录] -m -s /bin/bash -G sudo,adm,root [用户名]`
* 设置用户密码 `passwd [用户名]`

### SSH
编辑 SSH 配置文件, `sudo vim /etc/ssh/sshd_config`
* 修改端口 `port [新端口]`
* 禁止root用户登录 (请先确认新创建的用户可以正常登录) `PermitRootLogin no`
* 禁止空密码登录 `PermitEmptyPasswords no`
* 限制密码错误重试次数 `MaxAuthTries 3`
* 限制仅指定用户可登陆 `AllowUsers [用户名]`

### ufw 防火墙

[How To Setup a Firewall with UFW on an Ubuntu and Debian Cloud Server](https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server)

```
sudo ufw allow from [ip] to any port [端口] 仅特定ip可以访问此端口
sudo ufw allow 80 允许80端口访问
sudo ufw allow 443 允许443端口访问
sudo ufw disable 禁用ufw
sudo ufw enable 启用ufw
```

### logwatch

* 安装logwatch `apt-get install logwatch`
* 编辑任务, 发送log到指定邮箱 `vim /etc/cron.daily/00logwatch`
    ```
    #!/bin/bash
    # Check if removed-but-not-purged
    test -x /usr/share/logwatch/scripts/logwatch.pl || exit 0
    # execute
    /usr/sbin/logwatch --output mail --mailto [邮箱] --detail high
    # Note: It's possible to force the recipient in above command
    # Just pass --mailto address@a.com instead of --output mail
    ```

### mongodb

* 修改监听以便远程访问 `bind_ip = 0.0.0.0`
* 修改默认端口 `port = [端口]`
* 开启权限验证 `auth = true`

权限验证需要 admin 数据库存在用户时才会真正启用, 所以首先创建一个用户
```bash
# 进入 mongo shell
mongo
# 进入 admin 数据库
use admin
# 创建 root 用户
db.createUser(
   {
     user: "[用户名]",
     pwd: "[密码]",
     roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
   }
)
# 创建拥有指定数据库权限的用户
use [数据库名]
db.createUser(
   {
     user: "[用户名]",
     pwd: "[密码]",
     roles: [ { role: "readWrite", db: "[数据库名]" } ]
   }
)
```

## 常用配置

### nginx
```
# 本地目录
server {
    listen [端口];
    server_name [域名];
    root [目录];
    index index.html;
}
# 启用 https, 并将http请求转发到https
server {
    listen 80;
    server_name [域名];
    return 301 https://[域名]$request_uri;
}
server {
    listen 443 ssl;
    root [目录];
    server_name [域名];
    ssl on;
    ssl_certificate [crt证书];
    ssl_certificate_key [key证书];
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;
}
# 反向代理
server {
    listen [端口];
    server_name [域名];
    location / {
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header   Host             $http_host;
        proxy_set_header   X-NginX-Proxy    true;
        proxy_set_header   Connection "";
        proxy_http_version 1.1;
        proxy_pass         [代理地址]
    }
}
```

### sendmail

安装 sendmail
```
sudo apt-get install sendmail  
sudo apt-get install sendmail-cf
```

如果 mail 命令不存在, 安装 mailutils
```
sudo apt-get install mailutils
```

查看是否启动
```
ps aux | grep sendmail
```

编辑配置文件, `sudo vim /etc/mail/sendmail.mc`
修改监听地址, `DAEMON_OPTIONS('Family=inet,  Name=MTA-v4, Port=smtp, Addr=0.0.0.0')dnl</span>`

如果提示 hostname 不合格, 修改它
```
查看 hostname `hostname`
修改 hostname `hostname [新值]`
```

发送邮件
```
mail [邮箱地址]
主题
内容
Ctrl-D结束
```

### vim
```
syntax on  
set showmatch  
set nu  
set tabstop=4  
set noexpandtab  
setlocal noswapfile  
set bufhidden=hide  
set history=100  
set hlsearch  
set nobackup  
set ruler  
set rulerformat=%20(%2*%<%f%=\ %m%r\ %3l\ %c\ %p%%%)  
set selection=exclusive  
set selectmode=mouse,key  
set formatoptions=tcrqn  
set autoindent  
autocmd BufReadPost *  
  \ if line("'\"") > 0 && line("'\"") <= line("$") |  
  \   exe "normal! g`\"" |  
  \ endif  
set showcmd  
colorscheme desert   
syntax enable   
set tags=tags;   
set backspace=indent,eol,start  
let g:winManagerWindowLayout='FileExplorer|TagList'   
nmap wm :WMToggle<cr>  
let Tlist_Show_One_File=1   
let Tlist_Exit_OnlyWindow=1  
let g:miniBufExplMapCTabSwitchBufs=1   
let g:miniBufExplorerMoreThanOne=2  
nnoremap <silent> <F3> :Grep<CR>  
let Grep_Default_Options='-r --exclude-dir=.svn'  
set cscopequickfix=s-,c-,d-,i-,t-,e-  
set nocompatible              " be iMproved, required  
filetype off                  " required  
set rtp+=~/.vim/bundle/Vundle.vim  
call vundle#begin()  
Bundle 'gmarik/Vundle.vim'  
Bundle 'a.vim'  
Bundle 'taglist.vim'  
Bundle 'winmanager'  
Bundle 'minibufexpl.vim'  
Bundle 'grep.vim'  
Bundle 'wolfpython/cscope_map.vim'  
call vundle#end()            " required  
filetype plugin indent on    " required
```

安装插件: [插件工具](https://github.com/VundleVim/Vundle.vim)

## 常用命令

* `netstat -nlp` 查看端口监听
* `sshd -t` sshd 测试模式
* `cut -d: -f1 /etc/passwd` 查看用户列表
* `history [-20]` 查看zsh历史[offset, 倒序]
* `sudo service ssh restart | reload | stop` ssh服务
* `crontab -e` 编辑定时任务