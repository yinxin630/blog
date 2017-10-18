---
layout: post
author: 碎碎酱
title: HTML标签语义化
permalink: html-element-semantic
date: 2017-09-12
thumbnail: /img/html-logo.png
category: html
tags:
- html
lede: HTML标签语义化参考手册
---

本文不包含已经不被推荐使用的标签

`a` 超链接  
`abbr` 表示缩写, 并提供详细描述  
`address` 为 article 或 body 提供联系信息, 表示一个和联系信息无关的任意的地址时, 应该使用 p  
`area` 在图片上定义一个区域, 效果类似 a, 只能搭配 map 使用  
`article` 表示文档、页面、应用或网站中的独立结构，是一种可独立分配的或可复用的结构. 作者信息可通过 address 提供, 发布日期和时间可通过 time 提供  
`aside` 表示一个和其余页面内容几乎无关的部分，被认为是独立于该内容的一部分并且可以被单独的拆分出来而不会使整体受影响。其通常表现为侧边栏或者嵌入内容。  
`audio` 音频

`b` 表示相对于普通文本字体上有区别，但不表示任何特殊的强调或者关联  
`base` 表示一个文档中包含的所有相对URL的基本URL, 只能存在一个  
`bdi` 文本可能会以不同方向显示  
`bdo` 修改文本方向(从左往右, 从右往左)  
`blockquote` 块级引用, 表示引用的内容  
`body` html主体  
`br` 换行  
`button` 按钮  

`canvas` 绘图  
`caption` 表格标题  
`cite` 作品引用  
`code` 代码  
`col` 表格列, 通常位于 colgroup 内  
`colgroup` 表格列组

`data` 将指定内容和机器可读的值联系在一起  
`datalist` 包含一组 option 元素, 作为表单k控件可选值  
`dd` 作为 dl 子元素, 表示一个术语的描述  
`del` 表示从文档中删除的文本, 通常显示删除线  
`details` 表示一种用户可以从其获取附加信息的小部件  
`dfn` 表示一个术语的定义  
`dialog` 对话框  
`div` 通用块级容器  
`dl` 是一个包含术语定义以及描述的列表，通常用于展示词汇表或者元数据  
`dt` 作为 dl 子元素, 用于在一个定义列表中声明一个术语

`em` 表示内容需要被着重阅读  
`embed` 表示一个外部应用, 插件

`fieldset` 对表单控制元素分组  
`figcaption` 作为 figure 子元素, 是与其相关联的内容的说明/标题  
`figure` 代表一段独立内容, 内容可以是在主文中引用的图片，插图，表格，代码段等等  
`footer` 章节或者根节点的页脚  
`form` 表单

`h1` - `h6` 标题  
`head` 文档元信息  
`header` 表示一组引导性的帮助，可能包含标题元素，也可以包含其他元素，像logo、分节头部、搜索表单等  
`hr` 水平线, 表示段落级元素之间的主题转换  
`html` 根元素

`i` 表现因某些原因需要区分普通文本的一系列文本。例如技术术语、外文短语或是小说中人物的思想活动等，它的内容通常以斜体显示  
`iframe` 框架  
`img` 图像  
`input` 表单控件  
`ins` 表示插入到文档的文本

`kbd` 表示用户输入, 按键

`label` 项目标题, 通常与 input 关联  
`legend` fieldset 子元素, 用于表示 fieldset 的内容的标题  
`li` 列表项  
`link` 链接内容, 通常为css文件

`main` 文档的主体内容  
`map` 定义图像映射, 一个可点击的链接区域  
`mark` 突出显示的文字  
`menu` 菜单, 大部分浏览器不兼容  
`menuitem` 菜单项  
`meta` 元数据  
`meter` 表示已知范围的值

`nav` 描绘一个含有多个超链接的区域，这个区域包含转到其他页面，或者页面内部其他部分的链接列表  
`noscript` 不支持脚本的浏览器显示内容

`object` 表示一个外部资源, 这个资源可能是一张图片，一个嵌入的浏览上下文，亦或是一个插件所使用的资源  
`ol` 有序列表  
`optgroup` select 元素中的一组选项  
`option` 选项列表中的一项  
`output` 表示一个用户的操作或者计算的结果

`p` 文本段落  
`param` 定义 object 元素的参数  
`picture` 用来为 img 元素提供多个 source  
`pre` 预定义格式文本  
`progress` 进度

`q` 短引用, 行内引用

`rp` 用于为那些不能使用 ruby 元素展示 ruby 注解的浏览器，提供随后的圆括号  
`rt` 用于描述东亚字符的发音  
`rtc` 包含文字的语义注解  
`ruby` 用来展示东亚文字注音或字符注释

`s` 使用删除线渲染文本, 表示不再相关，或者不再准确的事情. 当表示文档编辑时, 提倡使用 del 和 ins  
`samp` 用于标识计算机程序输出  
`script` 脚本  
`section` 表示文档中的一个区域, 比如, 内容中的一个专题组, 一般来说会有包含一个标题  
`select` 表单控件  
`slot` web 组件的一部分, 代表一个占位符  
`small` 小一号字体  
`source` 为媒体内容提供源  
`span` 通用行内容器  
`strong` 重要文本, 粗体  
`style` 样式  
`sub` 定义了一个文本区域，出于排版的原因，与主要的文本相比，应该展示得更低并且更小  
`summary` details 元素的摘要，标题或图例  
`sup` 定义了一个文本区域，出于排版的原因，与主要的文本相比，应该展示得更高并且更小

`table` 表格  
`tbody` 表格内容行, 可出现多个  
`td` 表格元素  
`template` 是一种用于保存客户端内容的机制, 文档加载时不渲染, 可以在运行时使用 JavaScript 实例化  
`textarea` 多行文本控件  
`tfoot` 表格页脚  
`th` 表格标题元素  
`thead` 表格标题行  
`time` 日期或时间  
`title` 文档标题  
`tr` 表格行  
`track` audio 和 video 的子元素, 指定字幕

`u` 文本显示下划线  
`ul` 无序列表

`var` 表示变量的名称，或者由用户提供的值  
`video` 视频

`wbr` 标记文本中一个位置, 浏览器可以选择在这里换行