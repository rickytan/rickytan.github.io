---
layout: post
title: "我的翻墙代理"
date: 2014-07-14 22:14
comments: true
categories: ['GFW', 'Proxy', '翻墙', 'VPS']
---

不知道确切的日期，反正现在中国大陆访问 **[Google](https://www.google.com)** 的所有站点都直接返回 [404](http://zh.wikipedia.org/zh/HTTP_404) 了。这背后多少百度阴谋论又成为人们口中的谈资，但谈资归谈资，笑笑也就过去了，对于一个码农和伪科研者来说，没有 **Google** 是非常影响工作效率的。无论是学术 Paper 还是技术文档，从百度返回的屎一样的结果都会让人崩溃！

互联网是人类的互联网，人活着就不能放弃追求自由的权利！

# [ZenMate](https://chrome.google.com/webstore/detail/zenmate/fdcgdnkidjaadafnichfpabhfomcebme)
之前用过好长一段时间的 **[Chrome](https://www.google.com/chrome)** 插件，一键翻墙，无需过多配置，免费且连接稳定。后来开始收费了，于是弃用。再后来（也就是现在）又免费了，但大陆已经无法使用了。

当然了，大陆外也许还能用，用它来加密你的网络访问，哪怕是 `HTTP` 的站点。

# [红杏](https://chrome.google.com/webstore/detail/%E7%BA%A2%E6%9D%8F/heehjpdocpefckjobfgnfdbhoebhphkf)
取名于成语“红杏出墙”，是非常有本土特色的一个翻墙插件，刚出来时可以翻任何网站，稳定快速，免配置。<!--more-->后来开始分免费和`VIP`，免费用户可以上 [facebook](https://www.facebook.com)，[google](https://www.google.com)，和 [twitter](https://twitter.com)。

做为一个还有一定 `Web` 基础的码农，本人有时候也会通过改改插件的代码以访问以上三个之外的网站。也许后来他们监控到大量非 `VIP` 用户在翻其他网站，于是将代码打乱了。其实打不打乱区别不大

> If it runs, it can be cracked!

只是麻烦一点点而已。

随着伟大的防火墙越来越历害，*红杏* 也开始出现不稳定了，*Chrome Web Store* 上骂的人一大堆，一些人刚付了一年的费用结果就不能用了。事实上 *红杏* 的工作人员也是一直在维护，多服务器切换，出现问题后很快就会好的，只是最近挂的次数太多了。然后，最近，免费用户不能再用了……

# [ShadowSocks](http://shadowsocks.org/)
在朋友的推荐下，前不久开始用 [ShadowSocks](http://shadowsocks.org/)。也许是中国官方见识到了它的威力，于是连 *Shadowsocks* 的官方网站都被墙了。它的代理原理与 `go-agent` 类似，也是分 `server` 和 `client`。`server`监听代理请求，`client`则监听本地的 `SOCKS` 请求，`client`收到请求后，以可选的加密方式通过某种私有协议转发到 `server` 端，`server` 再发起网页请求到互联网。当然了，这个 `server` 是在墙外的，而这个私有协议就起到了穿墙的作用。

同时，如何你不想自己买 `VPS` 在墙外架设服务器的话，<https://www.shadowsocks.net/> 网站有一些热心网友为了人类的自由，分享了自己的 `server`，配置信息通过 `BASE64` 编码，并生成了二维码。读取方式也简单：

1. 用手机拍摄或把二维码图片链接地址贴到此[网页](http://zxing.org/w/decode.jspx)（推荐）上读取，得到原始信息：

		ss://YWVzLTI1Ni1jZmI6eG94b0AyMDguMTEwLjgzLjI0MjoxMjMzNA
    
2. 去掉`ss://`，复制后面的字符串到<http://tool.chinaz.com/Tools/base64.aspx> 下面那个框中，点 “BASE64解密”，得到：

		aes-256-cfb:xoxo@208.110.83.242:12334
		
3. 以上信息说明，采用`aes-256-cfb`加密方式，地址为：208.110.83.242，端口：12334，密码为：xoxo


## 优点
**Shadowsocks** 的优点是很明显的。首先，它采用私有协议，不那么容易被墙。其次，它有多种语言实现，安装都异常简单。

- Node 用户：

		npm install -g shadowsocks
- Python 用户

		pip install shadowsocks
		
- Ruby 用户

		gem install shadowsocks
		
等等。

最后，它拥有全平台支持的官户端，非程序员也可以较方便的使用

- PC 及 Mac 用户请下载[客户端](http://sourceforge.net/projects/shadowsocksgui/files/dist/)
- iOS 用户安装 [Shadowsocks](https://itunes.apple.com/cn/app/shadowsocks/id665729974?mt=8)
- 对于 Android 用户，[Google Play](https://play.google.com/store/apps/details?id=com.github.shadowsocks) 上有对应的版本，只是不能下载了，国内可以用 *碗豆夹* 试试，中文名叫 **影梭**

安装好客户端后，将之前获取的网友分享的服务器配置填入就可以了

![Shadowsocks](/images/shadowsocks-config.png)

代理服务器填 `sock5 127.0.0.1:1080`，取决于您的客户端配置。

##缺点
唯一的缺点是，需要在本地启动一个服务。对于这一点，见仁见智，当然你可以让它开机启动，但我觉得总差那么点意思。

## 我的代理服务器
### `VPS`
用过一段时间的 **Shadowsocks** ，网友共享的服务器用的人一多，也容易挂，于是就一直想自己搭一个。刚好也听说美国有很多 $15 一年的屌丝 `VPS` 出售，其性能做为个人使用的代理是绰绰有余的。经过对比之后，在 **[VPSDime](https://vpsdime.com/)** 买了一年的 `VPS`，配置如下：

- 双核共享CPU
- 512M 运行内存
- 30G 磁盘空间
- 1TB 网络带宽
- 1个独 IP

另外，还有 $8 每年的配置，不过好像卖完了。

我安装了 `Ubuntu 14.04` 从官方源升级软件达到了惊人的 30M/s！不过从国内 ssh 登录非常慢

### `HTTPS`代理
`Chrome` 浏览器支持最新的 `HTTPS` 代理（Safari 也支持），顾名思义，就是在传统的 `HTTP` 代理上加了安全协议。搭建代理服务器也是比较简单，可以直接用 `NodeJS` 安装 `spdyproxy` 来实现：

	npm install -g spdyproxy
	
很好用，也很方便。

	spdyproxy -k <您的私钥> -c <您的证书> -p <代理端口> -U <用户名，可选> -P <密码，可选>
	
但是，这个是在前台运行的，退出就没了，可以安装 `forever` 模块来让它在后台运行：

{% codeblock lang:bash%}
npm install -g forever
forever start `which spdyproxy` -k path/to/key.pem -c path/to/cert.pem -p ...
{% endcodeblock %}

当然也可以直接用 `nohup` 命令让它后台运行，`forever` 的好处是，当它挂了会自动重启！

    nohup spdyproxy -k path/to/key.pem -c path/to/cert.pem -p ...
	
然后，就可以在你的“网络偏好”中设置“Web安全代理”了。

![image](/images/https-proxy.png)

另外，一个值得权衡的问题是：不加密码（-U -P 参数），那么你的服务器容易被别人扫描发现；加密码，每次输入又很麻烦（有些系统下，每次网页请求都要输入密码，而不是第一次请求）。于是本人 **Fork** 了 `spdyproxy` 的 **Github** 源代码（<https://github.com/igrigorik/node-spdyproxy>），并添加了白名单支持：

    spdyproxy -k path/to/key.pem -c path/to/cert.pem -p 44300 -U username -P password -W whitelist.json

`whitelist.json` 中是一个 `JSON` 数组，每个元素是不需要密码的 IP 地址：

{% codeblock lang:json whitelist.json%}
[
'1.2.3.4',
'3.3.3.3',
'60.180.180.180'
]
{% endcodeblock%}

这样，添加自己常用的 IP 为白名单，自己访问不用密码，其他人访问需要密码，一定程度上防止别人盗用！

安装方式：

    git clone https://github.com/rickytan/node-spdyproxy
    npm install -g node-spdyproxy/package.json

### 自动代理配置

以上工作并没有完，让 **Chrome** 访问所有网站都通过代理不是明智的选择，于是在 `VPS` 上安装 `nginx` 或 `apache` 做为文件服务器，编写一个 `PAC` 文件，选择性使用代理：

{% codeblock lang:javascript proxy.pac %}
// 翻墙
var proxy_list = [
    'google.com',
    'facebook.com',
    ...
];

// 广告
var black_list = [
	"pubstat.sandai.net",
	"mcfg.sandai.net",
	"biz5.sandai.net",
	"float.sandai.net",
	"cl.kankan.xunlei.com",
	"211.94.190.80",
	"mtips.xunlei.com",
	"211.94.190.80",
	"mtips.xunlei.com",
	"adsresult.joywell.com.cn",
	"advstat.xunlei.com",
	"wy.xunlei.com",
	"kkpgv.xunlei.com",
	"statis.kankan.xunlei.com",
	"server1.adpolestar.net",
	"mpv.sandai.net",
	"vid.atm.youku.com",
	"valo.atm.youku.com",
	"valf.atm.youku.com",
	"walp.atm.youku.com",
	"static.atm.youku.com",
	"vid.atm.youku.com",
	"valo.atm.youku.com",
	"valf.atm.youku.com",
	"walp.atm.youku.com",
	"static.atm.youku.com",
	"*.p2v.tudou.com*",
	"at-img1.tdimg.com",
	"at-img2.tdimg.com",
	"at-img3.tdimg.com",
	"adplay.tudou.com",
	"adcontrol.tudou.com",
	"stat.tudou.com",
	"v2.stat.ku6.com",
	"v3.stat.ku6.com",
	"v0.stat.ku6.com",
	"v1.stat.ku6.com",
	"st.vq.ku6.cn",
	"stat2.888.ku6.com",
	"pq.stat.ku6.com",
	"mcfg.sandai.net",
	"biz5.sandai.net",
	"server1.adpolestar.net",
	"advstat.xunlei.com",
	"mpv.sandai.net",
	"images.sohu.com",
	"dcads.sina.com.cn",
	"pubstat.sandai.net",
	"float.sandai.net",
	"recommend.xunlei.com",
	"cl.kankan.xunlei.com",
	"googlesyndication.com",
	"pagead2.googlesyndication.com",
	"doubleclick.net",
	"union.baidu.com",
	"googleads.g.doubleclick.net",
	"cpro.baidu.com",
	"biz5.sandai.net",
	"*ad*.00000000*"
];

var https = "HTTPS xxx.xxx.xxx.xxx:xxx; DIRECT";
var direct = "DIRECT";
var deny = "PROXY 127.0.0.1;";

function FindProxyForURL(url, host) {
	for (var i=0;i<black_list.length;++i) {
		var domain = black_list[i];
		if (shExpMatch(host, domain)) {
			return deny;
		}
	};

	for (var i=0;i<proxy_list.length;++i) {
		var domain = proxy_list[i];
		if (domain.indexOf('.') > 0)
			domain = '.' + domain;
		if (host === proxy_list[i] ||
		    dnsDomainIs(host, domain)) {
			return https;
		}
	};
	return direct;
}
{% endcodeblock %}

*更详细的配置，请见：<https://github.com/igrigorik/node-spdyproxy>*

### 更好的方式
用自己的 `VPS` 为文件服务器是可行的，但是不方便。如之前所说，国内 ssh 访问非常慢，如果你要修改 `proxy.pac` 文件比较麻烦。同时，**Chrome** 启动时加载一个国外的配置文件会导致启动变慢。于是就想到了用 **[AVOSCloud](https://cn.avoscloud.com)** 的云代码功能，可以托管一些静态文件。

1. 登录 **AVOSCloud**，新建应用
2. 打开应用设置，添加 “Web主机” 的二级域名：`your-subdomain.avosapps.com`
3. 打开 “云代码” 面板，下载项目框架，Web主机版
4. 解压，在 `public` 目录下放置你的 proxy.pac 文件
5. 将项目建立 **git repo**，并 push 到 **[Github](https://github.com)**，或 **[Bitbucket](https://bitbucket.org)**（推荐，可私用）
6. 在 **AVOSCloud** 应用下的 “云代码” 面板下，填写**Repo**地址，并复制 `Deploy Key` 到你的 **Repo** 中
7. 在 “部署” 下，点“部署”
8. 在 **Mac** 中设置自动代理配置地址为：`http://your-subdomain.avosapps.com/proxy.pac`

以后想添加新的网站到翻墙列表，打开你的 **Repo**，在线编辑之后，登录 **AVOSCloud** 再点一次“部署”，重启你的 **Chrome** 即可，不用 ssh 登录到你的 `VPS` 了。同时，由于 **AVOSCloud** 在国内，访问速度较快，启动时间也没有影响。

## 总结
越是限制，越是激发人的潜能与创造力。以上，我们就用每年 $15 的费用建立起了一个私人的翻墙服务。感谢各个为人类的自由作出贡献的无私开发者，感谢 **[AVOSCloud](https://cn.avoscloud.com)** 提供的免费托管服务。
