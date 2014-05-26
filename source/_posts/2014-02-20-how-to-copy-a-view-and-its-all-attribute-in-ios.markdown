---
layout: post
title: "iOS 下如何复制一个视图（包括它的所有属性）"
date: 2014-02-20 02:43
comments: true
categories: ['iOS', '技术']
---

在某些效果中，我们需要在当前视图上创建一个新的、一样的视图，并且叠在老的上面，然后让新的视图移动、缩放，或什么的，这样做出来的动画看起来比较流畅。对于简单的界面，我们可能按原有的样子生成一个新的就好，有没有更好更通用的办法呢？

# NSCoding

`UIView`无法通过`copy`方法来创建一个副本，这个想必大家都已经试过了。事实上，`UIView`是服从<NSCoding>协议的，这样它才能从`Xib`中反序列化出来，而我们最常用的一个序列化类就是`NSKeyedArchiver`。
<!--more-->
所以创建一个视图的副本可以这样：

{% codeblock lang:objc %}
UIView *view = ...;
UIView *copy_view = [NSKeyedUnarchiver unarchiveObjectWithData:[NSKeyedArchiver archivedDataWithRootObject:view]];

{% endcodeblock %}

这样，视图中所有能通过`Xib`设置的属性（底色、字体、Frame等）全部复制到新的视图上了。