---
layout: post
title: "代码生成的 UIColor 与在 IB 中填写的颜色 RGB 最终颜色不一致的问题"
date: 2014-05-26 14:58
comments: true
categories: ['ios', 'IB', 'Xcode', 'UIKit', 'UIColor', 'Interface Builder']
---

iOS 应用界面的编写，按程序员的习惯可以分为两派：代码党和 `IB` 党。本人是倾向于 `Interface Builder` 的，毕竟能少写点就少写点，能用 `IB` 实现坚决不用代码。

但是在实际使用中，却发现一些问题，这个问题早有发现，我以为只是色差的原因，但是最近发现不那么简单了。

如下图，美术在 `PhotoShop` 中标了蓝色为 `#3689e5`，换成 `RGB` 为 54，137，229，于是在 `IB` 的颜色中填入此值，但是看到的颜色有差异。

![image0](/images/IB/QQ20140510-1.png)

<!--more-->

![image1](/images/IB/QQ20140510-2.png)

这点小差异其实一般人都不会太关心，但后来有一部分功能我不想用 `IB` 了，于是代码写了个界面，就发现颜色差异不能忍了，代码的颜色是对的，`IB` 的颜色偏深。

自己写了个 Demo ，截图如下：（这里差别还不是很大，因为我是在 Mac Pro Retian 自己的屏上输入的颜色，当你在外接显示器上用 `IB` 时，颜色差异十分明显）

![image2](/images/IB/s0.png)

其中上半部分是用 Color Inspector 输入的值：

![image3](/images/IB/s1.png)

可以对比下第 2 张图，与上面这张图，第 2 张图是在外接显示器上输入的，颜色差异之大，不可能是由 228 与 229 造成的！

我一再强调了显示器的不同，对，其实看到这里大家已经猜到，这种不同就来自显示器。点开 `RGB Sliders` 左边的按钮（我之前都不知道它能点的……），可以看到多种显示器配置文件

![image4](/images/IB/color-picker.png)

选择 `Generic RGB`，然后三个值会变掉，不管它，你自己再改成 54 137 229。这时它在这个显示器上显示的颜色可能与你要的不对，但是运行时，在模拟器上显示的就与代码无差别了！

![image5](/images/IB/s3.png)

可以用 Mac 自带的 *数码测色计* 来验证，注意要选 **显示原生值**！

个人的理解，每个显示器在制造时就会有色偏，为了纠正这种色偏，在出厂时会调好一个配置文件，指定显示器在显示 240 这个颜色时（打个比方），你应该显示 248 才能让人的眼睛看着是 240，即显示器上显示的颜色是在原生值上纠偏过的，而*数码测色计*取的的显示器上的原生值，即 248，显示器根本不知道 240 的存在。这就是为什么用取色的方式总有色差了。

附 [StackOverflow](http://stackoverflow.com/questions/7488378/weird-colors-in-xcode-interface-builder) 上的链接 <http://stackoverflow.com/questions/7488378/weird-colors-in-xcode-interface-builder>