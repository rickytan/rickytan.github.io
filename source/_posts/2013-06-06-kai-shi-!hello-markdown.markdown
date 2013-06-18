---
layout: post
title: "开始！Hello Markdown"
date: 2013-06-06 13:50
comments: true
categories: 技术
---
**新的开始**

在[biaobiaoqi](http://biaobiaoqi.me)的影响下，开始学着高端洋气一点，利用
[Github Pages](http://pages.github.com)和[Octopress](http://octopress.org)框架
搭起了自己的博客系统，但是此系统是纯静态页面组成的，写作用的是**Markdown**，自己并不熟悉。以这种方
式写页面还是第一次，真的不一样的体验，有点技术极客的感觉。万事开头难，就以此文作为**Markdown**写作
的开端吧！

本文以熟悉**Markdown**语法为目的，算是先练练手。

**标题**

{% codeblock %}
# 这是一级标题
## 这是二级标题
### 这是三级标题
......(依此类推
{% endcodeblock %}
# 这是一级标题
## 这是二级标题
### 这是三级标题

当然也有另外的写法：
{% codeblock %}
这是一级标题
======
这是二级标题
------
{% endcodeblock %}
这是一级标题
======
这是二级标题
------

**段落**

段落就是一些连续的文字，只要不出现空行（只有空格和Tab的行），Markdown就会认为是同一段，所以在你的文本编辑器中，一段文字太长，大可以回车另起一段。

**引用**

像邮件中一样，Markdown的引用以**>**开头

{% codeblock %}
> 这是一段引用文字。。。
> 你会发现，回车后**Vim**会自动在新行前加上**>**
{% endcodeblock %}

> 这是一段引用文字。。。
> 你会发现，回车后**Vim**会自动在新行前加上**>**

**列表**

{% codeblock %}
* 列表项
* 列表项
* 列表项
+ 列表项
+ 列表项
+ 列表项
- 列表项
- 列表项
- 列表项
{% endcodeblock %}

以上列表样式是一样的。

* 列表项
+ 列表项
- 列表项

有序号的列表，用数字+. 构成
{% codeblock %}
1. 这是1
2. 这是2
8. 这是3
{% endcodeblock %}

你会发现，数字不会影响Markdown最终的样式。

1. 这是1
2. 这是2
8. 这是3

**代码段**

简单地，一个Tab或缩进四个空格即可。
{% codeblock %}
	int main() {
		printf("Hello World!\n");
		return 0;
	}
{% endcodeblock %}

同样，**Vim**会自动在新行上加上四个空格！

	int main() {
		printf("Hello World!\n");
		return 0;
	}

另外，Octopress 有更强大的插件可以突现代码，如你看到的各种有序号开头的部分，可以使用 `codeblock` 和
`endcodeblock` 将代码包起来，在生成博客时，会自动加亮。
{% codeblock 这是一小段C程序 lang:c %}
\{\% codeblock 这是一小段C程序 lang:c \%\}
int main() {
	printf("Hello World!\n");
	return 0;
}
\{\% endcodeblock \%\}
{% endcodeblock %}

加上*lang:c*以告诉插件以何种语法高亮代码。

**链接**

{% codeblock %}
这里有一个[链接](http://www.google.com "Google")
<http://www.google.com>
{% endcodeblock %}

这里有一个[链接](http://www.google.com "Google")
<http://www.google.com>

**加粗**

{% codeblock %}
*加粗*
**加粗**
_加粗_
__加粗__
{% endcodeblock %}

*加粗*
**加粗**
_加粗_
__加粗__

**图片**

跟链接的写法类似，不过以!开头
{% codeblock %}
![图片替代文字](https://www.google.com/images/srpr/logo4w.png "Google")
{% endcodeblock %}

![图片替代文字](https://www.google.com/images/srpr/logo4w.png "Google")


