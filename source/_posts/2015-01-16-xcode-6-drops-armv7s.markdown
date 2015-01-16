---
layout: post
title: "Xcode 6 默认不再构建 armv7s 指令集的代码"
date: 2015-01-16 11:21
comments: true
categories: ['xcode', 'ios']
---

> 原文 [Xcode 6 drops armv7s](http://www.cocoanetics.com/2014/10/xcode-6-drops-armv7s/)

最新的 `Xcode` 默认已经不再为 `armv7s` 构建可执行代码了，这到底是计划之中还是一时疏忽？

当前最新版 `Xcode 6` 已经将 `${ARCHS_STANDARD}` 定义为 `armv7` 和 `arm64` 了，同时当你更新 `Xcode` 时它会不断提醒你删除你自己的定义，让 `Xcode` 帮你做决定。而在你不堪骚扰按它的做之后，你会发现你的项目不再构建 `armv7s` 的代码了。

`armv7s` 的指令集运行于 Apple A6 （iPhone 5）和 Apple A6X （iPad 4）CPU 上。接下来的 Apple A7 （iPhone 5S，iPad Air，iPad Mini Retina）已经使用了 64 位的 `arm64` 架构。
<!--more-->

当苹果在 `Xcode` 中加入 `armv7s` 的支持后，它让许多使用了第三方的二进制库，如 Google Analytics，的开发者困惑了很久。为了构建应用，链接器需要所有链接的外部库包含相同的架构。开发者们不得不从他们的应用中移除 `armv7s` 的支持，而第三方库需要更新他们的二进制代码以添加新架构的支持。

这并不是一个真的问题，因为尽管少了一些优化，但 A6 芯片还是可以很好地运行 `armv7` 的代码。只是你升级了 `Xcode` 之后，原来好好的代码突然出现链接错误会让你很不爽而已。

![Image0](/images/Screen-Shot-2014-10-10-at-10.42.32.png)

修复方法也很简单。作为一个应用开发者，你可以简单地按照 `Xcode` 的建议删掉你自己覆写的编译架构设置。如果设置是以**粗体**显示的，你可以按下 `CMD + delete` 让它回到项目推荐设置。

作为一个组件的开发商，你需要采用其他途径。你想要开发者自行决定是否支持 `armv7s`，那么你应该在你所有的静态库及 **Framework** 中加入 `armv7s` 的支持。

![Image1](/images/Screen-Shot-2014-10-10-at-10.49.50.png)

开发者的链接器会自动选择所需的架构片断，用以生成应用。如果你设置了 **Build Active Architecture Only** 你会发现只有特定的架构包含进去了。**Debug** 编译时，这个设置默认为 **YES**，它只会生成当前活动的设备或模拟器的架构代码以加快编译速度。**Release** 编译时默认为 **NO**，所有指定的架构都会生成。

在静态库的生成日志中，每一行是一个架构的编译命令，最后是一行将所有架构代码合并，生成一个大的二进制文件。

![Image2](/images/Screen-Shot-2014-10-10-at-10.53.48.png)

另一种检验库文件中包含的架构片断的方法是使用 `file` 命令：

```
file libBarCodeKit.a 
libBarCodeKit.a: Mach-O universal binary with 3 architectures
libBarCodeKit.a (for architecture armv7):	current ar archive random library
libBarCodeKit.a (for architecture arm64):	current ar archive random library
libBarCodeKit.a (for architecture armv7s):	current ar archive random library
```

这个库包含所有的移动架构的代码。如果你要生成一个静态的 **Framework** 或者是通用的静态库做为二进制包发布你甚至还要包含模拟器的架构，如下所示：

```
file DTRichTextEditor 
DTRichTextEditor: Mach-O universal binary with 5 architectures
DTRichTextEditor (for architecture armv7):	current ar archive random library
DTRichTextEditor (for architecture armv7s):	current ar archive random library
DTRichTextEditor (for architecture arm64):	current ar archive random library
DTRichTextEditor (for architecture i386):	current ar archive random library
DTRichTextEditor (for architecture x86_64):	current ar archive random library
```

总而言之，苹果再一次将我们推向指定的方向：停止支持 `armv7s`。这个架构目前已经被两代新产品的更强大的 64 位 CPU 取代。但是，作为组件开发商，我相信我们仍然需要加入 `armv7s` 的支持而将选择权交给应用开发者。