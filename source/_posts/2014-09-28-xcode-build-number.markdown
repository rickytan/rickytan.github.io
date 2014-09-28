---
layout: post
title: "XCode 中如何自动增长 Build 号"
date: 2014-09-28 19:46
comments: true
categories: ["XCode", "iOS", "技术"]
---

## iTunes Connect 新步骤
随着 iOS8 的推出，苹果 itunes connect 中上传新应用多了一个新步骤：测试。

几个月前听闻 Apple 收购 [TestFlight](https://testflightapp.com)，但一直没有看到任何变化，如今 TestFlight 已经集成到新应用的发布
、审核过程中了。打开 [iTunesConnect](https://itunesconnect.apple.com)，可以看到在 `Versions` 旁边新
增了一个 `Prerelease` 标签页。原先通过 `XCode` 上传的应用会直接进入苹果的审核队列中，现在它会先出现
在这里，这里其实就是一个 `TestFlight` 的功能，你可以在 ***Users and Roles*** 中增加测试人员的 *Apple ID*，
然后在 `Prerelease` 下的 `Internal Testers` 中可以看到。每上传一个新的 ***Build***，苹果会自动发邮件给
测试人员，测试人号点开邮件中的链接，在 `Safari` 中打开，然后 `Safari` 会打开设备上的 `Test Flight`，
开始自动下载应用进行测试。

这样的好处很明显，正在进行测试的版本与正式上线的版本是完全一样的，因为原来我们要做测试可能测试版与正式
版使用了不同的签名证书，然后每次加新人又要添加设备 `UDID` 到 ***Provisioning Profile*** 等等。

测试通过之后，回到 `iTunesConnect` 中的 `Versions`，下拉找到 `Build`，选择刚刚测试好的 ***Build*** 号，
点 `Save`，然后就可以进入正常的审核队列了。

> ***注意：***测试功能只有使用苹果开发者网站最近生成的 ***Provisioning Profile*** 才能开启，而且会自
动加入，不要去 ***Entitlement*** 中勾选了。

## Build 号
在 `iOS` 应用的 `Info.plist` 中有两个版本号，即 ***Version*** 和 ***Build***，对应键值为 `CFBundleShortVersionString` 和 `CFBundleVersion`。
一个是应用程序本身发布时的版本号，一个是表示这个发布版的二进制代码是第几次编译得到的。在 ***iOS8*** 
之前由于苹果并没有约束这两者的关系，本人一般就把两个填一样的，如 `2.1.4` 等，也一直没出过问题，但是
最近由于苹果加入了测试功能，所有上传的 ***Build*** 都会保存，然后第一个因为某原因审核不过，再上传第
二个就出现了 ***Build*** 号冲突的问题。于是想到这才是 ***Build*** 的正确使用方法，它本应该是个整数，
在 ***Version*** 保持不变，***Build*** 应该是要随着编译次数增长的！

## 自动增长

那么如何让 ***Build*** 号自动增加呢？很早前搜到过一段代码，现在可以用上了：

1. 在 `XCode` 中选中项目，打开 `Build Phases`，如下图：

	![RunScript](/images/run-script.png)

2. 点 “+” 增加一个过程，选择“Run Script”，移到“Link Binary With Libraries”之后，并贴入以下代码：

	{% codeblock lang:bash %}
#!/bin/bash
buildNumber=$(/usr/libexec/PlistBuddy -c "Print CFBundleVersion" "$INFOPLIST_FILE")
buildNumber=$(($buildNumber + 1))
/usr/libexec/PlistBuddy -c "Set :CFBundleVersion $buildNumber" "$INFOPLIST_FILE"
	{% endcodeblock %}

