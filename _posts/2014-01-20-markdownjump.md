---
layout: post_m
title:  "实现markdown的页内跳转"
date:   2014-01-20 15:07:07
categories: Markdown
description:  
src :  http://blog.chinaunix.net/uid-20147410-id-3706093.html
---


默认情况下，标准的markdown语法并没有业内跳转的功能, 可以使用html标签来解决。

##markdown页内跳转实现

1. 先定义一个锚(id)

		<span id="jump">Hello World</span>


2. 然后使用markdown的语法:

		[XXXX](#jump)

##使用Ultisnips实现代码片段

在markdown.snippets中增加下面片段，能够实现 jump 自动输入跳转代码片段。

    snippet jump "建立跳转"
    <span id="${1:jump}"> </span>
    [$2](#$1)
    endsnippet