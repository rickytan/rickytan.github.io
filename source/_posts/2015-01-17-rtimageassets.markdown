---
layout: post
title: "自动生成低清图的 Xcode 插件"
date: 2015-01-17 00:02
comments: true
categories: ['xcode', 'ios', 'rtimageassets']
---

## 前言
随着 iPhone 6 Plus 等的发布，在普通消费者眼中，又要少两个肾，在 **iOS** 开发者眼中，又有两款新屏幕需要支持了。新的 iPhone 6 Plus 拥有一块 5.5 英寸的视网膜屏，分辨率为 `1080x1920`，**PPI** 为 401，在程序中表现为 **3x**，即，对于一张代码中指定的 `100x100` 的图片，实际需要 `300*300` 的图片填充才不会产生模糊。为此开发者们及美术们又要伤不少脑筋了，开发者需要一遍又一遍地向美术解释，每张图需要切三份，每份应该怎么样。有时候少切了，或者不满足要求，程序员还得自己动手缩放。

程序员总是懒的，于是我做了一个 `Xcode` 插件来完成此事：[RTImageAssets](https://github.com/rickytan/RTImageAssets)
<!--more-->

> ##简介

> 本项目是一个 Xcode 插件，用来生成 @3x 的图片资源对应的 @2x 和 @1x 版本，只要拖拽高清图到 @3x 的位置上，然后按 Ctrl+Shift+A 即可自动生成两张低清的补全空位。当然你也可以从 @2x 的图生成 @3x 版本，如果你对图片质量要求不高的话。

> ##特性

> - 只会填补空位，如果你已经设置好了自己的 @2x 图，则不会生成；
> - 自动重命名，保持项目干净（把 N.imageset 下的图片名字改为 N.png N@2x.png N@3x.png 等）；
> - 使用简单，不用再麻烦美术同学缩放了；

> **注意：**本插件从 @3x 到 @2x 的缩放保证图片在屏幕上显示的物理尺寸一样，而不是与屏幕比例一样，缩放系数是 1.5，而不是 1242 / 640 = 1.94。

更多信息，请移步：<https://github.com/rickytan/RTImageAssets>

## 安装

你可以从源码构建安装（推荐）：

	git clone https://github.com/rickytan/RTImageAssets.git
	
或者运行如下命令安装 `Xcode` 插件管理器（官网：<http://alcatraz.io/>）：

	curl -fsSL https://raw.github.com/supermarin/Alcatraz/master/Scripts/install.sh | sh
	
搜索：`RTImageAssets`

本插件还在不断完善之中，并在不久后会加入自动生成所有 **Icon** 的功能，欢迎各位测试吐槽！如有任何问题，请到 <https://github.com/rickytan/RTImageAssets/issues> 创建 **Issue**
