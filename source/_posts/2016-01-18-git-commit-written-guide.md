---
layout: post
author: 碎碎酱
title: Commit Message 编写指南
permalink: git-commit-written-guide
date: 2016-01-18
thumbnail: /img/git-logo.jpg
category: software
tags:
- git
lede: "我们在每次提交代码时，都需要编写Commit Message，否则是不运行提交的。编写Commit Message需要遵循一定的范式，内容应该清晰明了，指明本次提交的目的，便于日后追踪问题。"
---


参考文档：
[《Commit message 和 Change log 编写指南》][2]

我们在每次提交代码时，都需要编写Commit Message，否则是不运行提交的。 `git commit -m "hello world"`, 编写Commit Message需要遵循一定的范式，内容应该清晰明了，指明本次提交的目的，便于日后追踪问题。

普通的范式：
![2016-01-18-git-commit-written-guide-01](/img/2016-01-18-git-commit-written-guide-01.png)

良好的范式：
![2016-01-18-git-commit-written-guide-02](/img/2016-01-18-git-commit-written-guide-02.png)

## Commit Message 格式

一般来说，Commit Message 应包含三部分内容：Header、Body、Footer

```
<type>(<scope>): <subject>
// 空一行
<body>
// 空一行
<footer>
```

### Header

Header部分应只包含一行，包括三个字段：`type`、`scope`和`subject`

1. type
    `type`用于说明Commit的类型，包含一下7种类型

    ```
    feat：新功能（feature）
    fix：修补bug
    docs：文档（documentation）
    style： 格式（不影响代码运行的变动）
    refactor：重构（即不是新增功能，也不是修改bug的代码变动）
    test：增加测试
    chore：构建过程或辅助工具的变动
    ```

2. scope

    `scope`用于说明本次Commit所影响的范围，比如`controller`、`user`或者`README`，视项目的不同而不同

3. subject

    `subject`是本次Commit目的的简短描述，一般不要超过50个字符
    
    ```
    以动词开头，使用第一人称现在时，比如change，而不是changed或changes
    第一个字母小写
    结尾不加句号（.）
    ```
    
### Body

`Body`是对本地提交的一个详细描述，下面是一个示例

```
More detailed explanatory text, if necessary.  Wrap it to 
about 72 characters or so. 

Further paragraphs come after blank lines.

- Bullet points are okay, too
- Use a hanging indent
```

### Footer

`Footer`只用于两种情况

#### 1. 不兼容改动

如果当前代码与上一个版本不兼容，则 Footer 部分以`BREAKING CHANGE`开头，后面是对变动的描述、以及变动理由和迁移方法。

#### 2. 关闭Issue

如果当前Commit是针对某个Issue的提交，那么久可以在Footer中关闭这个Issue：`Closes #234`

## Commitizen

[Commitizen][3]是一个编写合格Commit Message的实用工具

安装：`npm install -g commitizen`
添加项目支持：`commitizen init cz-conventional-changelog --save --save-exact`

以后，凡事需要`git commit`命令的地方，都用`git cz`来代替，会自动的出现选项，用来生成符合格式的Commit Message

![2016-01-18-git-commit-written-guide-03](/img/2016-01-18-git-commit-written-guide-03.png)


  [1]: http://www.suisuijiang.com
  [2]: http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html
  [3]: https://github.com/commitizen/cz-cli
  [4]: https://github.com/commitizen/cz-cli/raw/master/meta/screenshots/add-commit.png