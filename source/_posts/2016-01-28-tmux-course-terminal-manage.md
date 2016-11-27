---
layout: post
author: 碎碎酱
title: tmux教程：使用tmux、tmuxinator管理你的终端窗口
date: 2016-01-28
thumbnail: /img/tmux.png
category: software
tags:
- tmux
lede: "tmux是什么？tmux是一个优秀的终端复用软件，类似GNU Screen，但来自于OpenBSD，采用BSD授权。使用它最直观的好处就是，通过一个终端登录远程主机并运行tmux后，在其中可以开启多个控制台而无需再“浪费”多余的终端来连接这台远程主机。"
---

[tmux][2]是什么？tmux是一个优秀的终端复用软件，类似GNU Screen，但来自于OpenBSD，采用BSD授权。使用它最直观的好处就是，通过一个终端登录远程主机并运行tmux后，在其中可以开启多个控制台而无需再“浪费”多余的终端来连接这台远程主机。

当然，仅仅是用来切分窗口的话，tmux并不亮眼，考虑下你是否遇到过以下情形：

1. 在公司时，开了一堆vim、log和debug窗口，下班回家后，灵感突发，SSH连上公司机器，然后重新打开各种窗口。。。囧~ 瞬间就没心情了。
2. 在服务器上调试程序，开了一堆窗口，有事需要出去一下，回来的时候发现SSH都超时了，`broken pipe`。。。FUCK，如果你使用tmux的话，就不会有这个问题。

## tmux概念

tmux是典型的C/S架构，包含如下概念：

1. Session：会话，tmux使用会话管理不同任务，你可以创建用于work的会话、或者用于play的会话，随时可以切换不同的会话。
2. Window：窗口，tmux会话可以包含多个窗口，每个窗口都是一个完整的终端。
3. Pane：窗格，窗口可以切分出任意数量、任意大小的窗格，每个窗格均是一个shell终端。

## tmux安装

### 在*linux*中安装：

下载[tmux-2.1.tar.gz][3]，解压并进入tmux目录，执行`./configure && make`构建程序，再执行`sudo make install`安装应用。

### 在*ubuntu*中安装：

在终端输入`sudo apt-get install tmux`

### 在*MAC*中安装：

首先安装Homebrew，已安装过可以跳过此步。

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

安装tmux：`brew install tmux`

## tmux使用

在终端输入`tumx`，即可进入tmux环境。

### tmux快捷键

tmux为了使快捷键不与系统快捷键冲突，需要使用快捷键前缀来组合快捷键，默认的快捷键前缀是C-b(Ctrl+B一起按)，你可以把这个快捷键理解成vim的命令模式，按下C-b后进入命令模式，再按下不同的按键执行对应的功能。

因为C-b不太容易按，可以选择修改为自己顺手的按键，在`~/.tmux.conf`中添加下列内容，将其修改为C-a：

```
unbind C-b
set -g prefix C-a
```

### 常用快捷键

使用C-b + ?可以查看所有可用命令以及相应的快捷键。

常用的命令有：

```
prefix Space            next-layout
prefix "                split-window
prefix %                split-window -h
prefix (                switch-client -p
prefix )                switch-client -n
prefix [num:0-9]        select-window -t :=[num:0-9]
prefix p                previous-window
prefix n                next-window
prefix c                new-window
prefix :                command-prompt
prefix C-Up             resize-pane -U
prefix C-Down           resize-pane -D
prefix C-Left           resize-pane -L
prefix C-Right          resize-pane -R
prefix {                swap-pane -U
prefix }                swap-pane -D
```

## tmux实用工具：Tmuxinator

假如你需要经常创建tmux会话，它包含三个窗格：vim、debug和shell，并且每个窗格需要不同的大小。又或者，你又临时投入另一项工作，需要打开多个终端。你可能会想，如果仅需一条命令就可以做的该多好，好的，tmuxinator出场了。

[Tmuxinator][4] 是一个 Ruby 的 gem 包，可用于创建 Tmux 的会话。它的工作方式是先在配置文件中定义会话中的细节，然后用 1 条命令创建出这些会话。下面就让我们看看如何安装 Tmuxinator 以及如何添加配置来为指定项目开启一个会话。可以通过运行如下命令安装 Tmuxinator 的 gem 包。

```
gem install tmuxinator
```

安装好之后，就可以在终端中使用`tmuxinator`或者缩写`mux`来启动Tmuxinator了。

首先让我们创建一个工程：

```
mux new project0
```

tmuxinator会在`~/.tmuxinator/`目录中创建`project0.yml`文件，编辑`project0.yml`，写入下列内容(**重点提醒：书写时要使用空格缩进，且严格按照图示缩进对其，否则会报语法错误！**)：

```
# ~/.tmuxinator/project0.yml

name: project0
root: ~/

windows:
  - editor:
      layout: main-vertical
      panes:
        - vim
        - top
  - ls: ls
  - netconfig: ifconfig
```

启动刚刚创建的项目：

```
mux start project0
```

你会看到，我们的tmux自动的启动了一个tmux会话，它包含三个窗口，其中第一个窗口包含两个窗格，每个窗体/窗格均执行了指定任务。

## 在Tmuxinator中配置layout

我们可能会需要指定窗格的排班规则，tmuxinator支持设置tmux中的5种默认layout样式：

    even-horizontal
             Panes are spread out evenly from left to right across the window.

    even-vertical
             Panes are spread evenly from top to bottom.

     main-horizontal
             A large (main) pane is shown at the top of the window and the
             remaining panes are spread from left to right in the leftover
             space at the bottom.  Use the main-pane-height window option to
             specify the height of the top pane.

     main-vertical
             Similar to main-horizontal but the large pane is placed on the
             left and the others spread from top to bottom along the right.
             See the main-pane-width window option.

     tiled   Panes are spread out as evenly as possible over the window in
             both rows and columns.

但是默认的5种样式不能让我们自由的设定每个窗格的大小，所以我们要使用自定义的layout，在tmux中可以使用C-b + 方向键调整窗格的大小。tmuxinator中可以直接使用tmux中的layout值，在终端输入`tmux list-windows`可以查看每个窗口的layout值，我们只需将这个值写入tmuxinator项目配置配置中的layout即可。

  [1]: http://www.suisuijiang.com
  [2]: https://tmux.github.io/
  [3]: https://github.com/tmux/tmux/releases/download/2.1/tmux-2.1.tar.gz
  [4]: https://github.com/tmuxinator/tmuxinator