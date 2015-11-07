---
layout: post
title: "是时候使用 PDF 资源了"
date: 2015-11-06 17:56
comments: true
categories: ['Xcode', 'iOS', 'Development']
---

一年前我做了这个插件 [RTImageAssets](https://github.com/rickytan/RTImageAssets)，为了解决自己在开发过
程中的一个痛点，这个痛点貌似也恰好是许多其他 iOS 开发者的痛点，那就是要不断地向美术解释如何正确切图。

于是一时间也获得了不少反馈，但这始终只是个临时解决办法。事实上从 **Xcode 6** 开始就已经支持使用 PDF
矢量图了。现在 iOS 设备越来越多，谁知道以后需要支持什么样的屏幕，而矢量图正是解决这一问题的良策。<!--more-->另外，很多同行反应从 @3x 的图生成的 @2x 的图点用的磁盘空间还大些，这其实是色彩空间和 PNG
压缩算法决定的，我也没有时间深入研究这个，开发这个插件的目的只是让它有 @2x 的图以保证运行时图片大小跟
期望的一致，事实上 **Xcode** 在打包时会自动优化 PNG 图片。所以终极解决方案，还是从现在开始，全部使用矢量图吧！

如果对 PNG 图片压缩有兴趣，相关信息可以在 <https://tinypng.com/> 和 <https://imageoptim.com/> 上找到。

使用矢量图的方法跟普通 PNG 一样，按下图设置好后，原来的三个槽会变成一个，重新把图片拖进去就好：

![image0](/images/pdf-asset/1.png)

一定要选为 *Vector* ，之前碰到过将 **PDF** 图当 **PNG** 用的人……

这样一来，无需再担心屏幕的分辨率问题，随着 **Xcode** 的更新，它会自动加入所需的分辨率。

你不需要担心移动设置上使用矢量图的性能问题，因为事实上在打包的时候 **Xcode** 将它们变成 PNG 了：

![image1](/images/pdf-asset/2.png)

> _最低支持到 **iOS 6** 时，会使用原始 PNG 图，可以看到以上的命名方式，**iOS 7** 以上已经将所有图片打包为 `Assets.car` 了_

关于矢量图的生成问题，如果美术会使用 [Sketch](http://www.sketchcn.com/) ，那么它对生成  PDF 等十分友好，如果 TA 不会用 Sketch，你是时候招个新的了。



