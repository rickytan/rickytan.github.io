---
layout: post
title: "蛐蛐儿 SDK 破解"
date: 2014-01-18 15:23
comments: true
categories: ['技术', '破解', '蛐蛐儿']
---

[蛐蛐儿](http://xququ.com) 是一套利用声音传输信息的解决方案，大家可能知道 `支付宝` 在早些时候就推出了当面付的功能，其实就是利用超声波将加密后的支付信息传输给附近的人。蛐蛐儿SDK可以在其官网上下载到，通过它的提供的Demo可以看出，它需要将一段任意字符串_S_上传它们的后台，然后生成一段16字节的_Token_，发送时只能发送此_Token_。同时，在接收方接收到_Token_后，需要访问它们的后台把原始字符串_S_下载下来。这个过程也正好解释了，为什么无论发送什么东西，声音长度是一致的，真正的数据并不是通过声音传的，而是通过网络下载的。

当然，其实大部分情况下这个过程已经够用了，因为开发者可以将自己需要传输的任何字符串编码为_Token_（这个字符串可以是`BASE64`编码，也可以是一个文件的下载链接），通过声音传输，然后再解码。

{% codeblock viewcontroller.m lang:objc %}
- (void)viewDidLoad
{
	[super viewDidLoad];
	[[XQuquerService defaultService] setDelegate:self];
	[[XQuquerService defaultService] start];
	
	NSString *str = ...;
	[[XQuquerService defaultService] uploadData:str];
	...
}

#prama mark XQuquer Delegate
- (void)didSendDataToken
{

}

- (void)didReceiveDataToken:(NSString *)dataToken
{
	...
}
{% endcodeblock %}

但是如何我要传输的字符少于16位呢？这个过程似乎有点过了，于是我将_Token_改为任意一个我想要传输的字符串，发现没有声音了。原来这个_Token_并不是一个随机的值。原来我以为，SDK的后台就一张表，记录着_Token_和原始值的对应关系，而_Token_本身是随机的，或者就像短链接一样，将太长的内容变短而已。

_Token_的生成引起了我的兴趣，于是我记下了一些能发送的_Token_，如：

    020240d3d0d42a48

改动任何字符均会导致没有声音，所以猜测这个_Token_本身带有验证机制。于是后来就跟踪到汇编代码想看看它的验证过程，这样我好自己生成_Token_。跟踪过程我就不细说了，反正中间出现过一段字符串：**0003242ddd4082a4**，我对照了一下，没有什么规律，然后又出现了**0003242dddwuyifan**，对比下SDK 的作者，正是 **wu yifan**，而这里他将后6位换成自己名字拼音做什么呢？

对于这两段 _Token_ ：

    A = 020240d3d0d42a48
    B = 0003242ddd4082a4
    
它们等长，同时 _A_ 中出现过的字符 _B_ 中也同样出现，并且词频是一样的！
在然后的几次单指令跟踪时，发现变量名中出过过 `dkey` 和 `ekey`，应该是 `decode` 和 `encode` ，检查内存，发现它们是 `int[16]` 的数组，这才发现 _A_ 到 _B_ 就是一个简单映射：

{% codeblock lang:c %}
const int dkey[] = {};

char A[16] = "020240d3d0d42a48";
char B[17] = {0};
for (int i=0; i<16; ++i) {
	B[i] = A[dkey[i]];
}
{% endcodeblock %}

这个问题解决了，那作者自己的名字拼音是怎么回事呢？还有那段被替换的后6位？在后来的跟踪中，发现代码有调用 `MD5` ，参数是替换成名字拼音的字符！

    MD5("0003242dddwuyifan") = "4082a4f05d45574077b88a53965a7293"
    
这货的前6位，不就是 _B_ 的后6位吗？到此， _Token_ 的验证过程已经非常清楚了：

    Token = "...";              // 原始 Token
    Token' = decode(Token);     // 用 dkey 解密的 Token
    Prefix = Token'[0, 10];     // 前 10 位
    Subfix = Token'[10, 6];     // 后 6 位
    
    if (Subfix == MD5(Prefix + "wuyifan")[0, 6])
        return TRUE;
    return FALSE;
    
所以，我要自己生成 _Token_ 那就是它的反过程了！

    MyValue = "...";            // 必须 10 位长
    Subfix = MD5(MyValue + "wuyifan")[0, 6];
    Token = encode(MyValue + Subfix);
    
    return Token;
    
用代码测试了下，有些生成的 _Token_ 可以发送，有些不行。原来它声音发送的数据还有一点要求，`MyValue` 必须是 16 进制的字符 0~f，并且第1位是0~7，总的来说，一次只能通过声音传5字节不到的数据。

	出于保护的目的，dkey ekey 就不公布出来了，感兴趣的可以私下联系。另外，如果`蛐蛐儿`的开发者认为此
	文章不合适也请联系本人删除。
