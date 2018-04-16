---
layout: post
title: "Markdown Learn"
description: Basic learning markdown
date: 2014-08-18 01:07:36 +0800
tags: [blog,markdown] 
img: post-1.jpg
---
A First Level Header
====================
Some of these words *are emphasized*.
Some of these words _are emphasized also_.
Use two asterisks for **strong emphasis**.
Or, if you prefer, __use two underscores instead__.

A Second Level Header
--------------------

Now is the time for all good men to come to the aid of their country,
This is just a regular paragraph. 

The quick brown fox jumped over the lazy dog's back.
###Header 3

>This is a blockquote.
>
>This is the second paragraph in the blockquote.
>
>This is an H2 in a blockquote

Some of these words *are emphasized*.
Some of these words _are emphasized also_.
Use two asterisks for **strong emphasis**.
Or, if you prefer, __use two underscores instead__.
###Lists
####无序列表
无序列表使用星号，加号和减号作为列表项目的标记
#####星号
* Candy.
* Gum.
* Booze.
#####加号
+ Candy.
+ Gum.
+ Booze.
#####减号
- Candy.
- Gum.
- Booze.
#####混合使用
* Candy.
+ Gum.
- Booze.
由上面的测试可以显示，三者之间是完全等价的（**符号和内容之间要有一个空格**）
####有序列表
有序列表使用数字接着一个英文句点作为项目标记：
1. Red
2. Green
3. Bule
**句点和内容之间要有一个空格**
####项目之间的空行问题
* A list item.

	With multiple paragraphs. 

* Another item in the list.
<!--more-->

###链接
Markdown有两种链接的语法：行内和参考两种形式，都是使用角括号来把文字转成链接。
行内形式是直接在后面用括号直接接上链接：
This is an [example link](http://example.com/).
也可以选择给链接加上title属性：
This is an [titled example link](http://example.com/ "With a Title which inditicates as a tooltip").

参考形式的链接让你可以为链接定一个名称，之后你可以在文件的其他地方定义该链接的内容：
I get 10 times more traffic from [Google][1] than from
[Yahoo][2] or [MSN][3].

[1]: http://google.com/ "Google"
[2]: http://search.yahoo.com/ "Yahoo Search"
[3]: http://search.msn.com/ "MSN Search"


title 属性是选择性的，链接名称可以用字母、数字和空格，但是不分大小写：

I start my morning with a cup of coffee and
[The New York Times][NY Times].

[ny times]: http://www.nytimes.com/

####图片
图片的语法和链接很像。
行内形式（title是可选择的）：

![first picture](/blog/assets/img/markdownlearn/keyboardcat.gif "Cat")

参考形式：
![second picture](keyboardcat.gif)

![my picture][id]

[id]: keyboardcat.gif "Cat cat"

######注意点
>链接的路径，最好设置成/images/blogname/picturefilename.jpg,绝对路径
>
>参考形式的冒号和路径之间要有个空格

####代码
在一般段落文字中，可以使用反引号 \` 来标记代码区段，区段的`&`、`<`、`>`都会被自动转换成HTML实体，反引号可以很容易在代码区段中插入HTML代码：

I strongly recommend against using any `<blink>` tags.

I wish SmartyPants used named entities like `&mdash;`
instead of decimal-encoded entites like `&#8212;`.

如果要建立一个已经格式化好的代码区块，只要每行都缩进 4 个空格或是一个 tab 就可以了，而 &、< 和 > 也一样会自动转成 HTML 实体。

If you want your page to validate under XHTML 1.0 Strict,
you've got to put paragraph tags in your blockquotes:

	<blockquote>
	<p>For example.</p>
	</blockquote>


