---
layout: post
title: "iZJU中浙大黄页的实现"
date: 2013-06-16 11:33
comments: true
categories: ['技术', 'iZJU'] 
---

## 简介

用过[iZJU iOS](https://itunes.apple.com/cn/app/izju/id573810521?mt=8)版的同学应该会注意到，在2.1.1版本之后新增了“浙大黄页”的功能。

![iZJU](/images/screenshot20130616.png)

这个功能是原先“紧急电话”的加强版，细心的同学会知道，所有的电话号码数据来自“浙大电话黄页”网站（<http://zjutel.zju.edu.cn>），如下图：

![浙大黄页](/images/QQ20130616-1.png)

## 数据获取

网站管理员我不熟，他也不会主动给我数据，获取这个唯一的办法就是抓取了。但是网站数据没有一次列全，自己点开哪个就能看到哪个，如何一次性抓到所有数据？人工去点开吧，嗯，一开始我是这么想的，可是后来发现能展开的节点太多了，自己点太蛋疼了。于是就想到了让浏览器自动点。

于是乎花了些时间写了个Chrome 小插件，如果还有节点，点开那个节点链接，否则无动作。
主要代码如下，通过一张小gif 图来判断是否可以展开：

{% codeblock index.user.js lang:js %}
function open() {
    var img = $("font").find("img");
    img.each(function(k,v){
        if (v.src.indexOf("w.gif") != -1) {
            var p1 = $(this).parents("tr").children("td").length;
            var p2 = $(this).parents("tr").next().children("td").length;
            if (p1 >= p2) {
                var c = this.parentNode.attributes[0];
                var urlS = c.childNodes[0].textContent;
                var idx = urlS.indexOf("?", 0)
                var url = urlS.substr(idx, urlS.length - idx - 2);
                var a = document.createElement("a");
                a.href = url;
                document.body.appendChild(a);
                a.click();
                return false;
            }
        }
    });
}
window.setTimeout(open, 300);
{% endcodeblock %}

打包为.crx 后**[安装](/download/zjutel.crx)**，在Chrome访问<http://zjutel.zju.edu.cn/tree.php>，插件就开始工作了，这时你可以坐下来喝喝咖啡，等它停止。

停止之后，将整个网页下载下来，可以用`Vim`等文本编辑器用搜索替换的方式去除HTML标签等不必要的信息，最终整理成如下所示文本文件（tree.txt）：

{% codeblock %}
!+ 浙江大学
!!+ 党委正 副书记(秘书)
!!! 党委书记秘书 : 88981739
!!! 邹晓东副书记联系秘书 : 88981457
!!! 郑强副书记联系秘书 : 88981175
!!! 任少波副书记联系秘书 : 88981062
!!! 周谷平 : 
!!+ 校长 副校长(秘书)
!!! 杨卫校长秘书 : 88981109
!!! 来茂德副校长联系秘书 : 88981217
......
{% endcodeblock %}

以上文本，既包含了节点信息，又有节点间的层次关系。下面将说明如何处理它让它成为结构化的数据。

## 数据处理

观察上述文本的格式，以“＋”号结尾的都是有子部门的大部门，而没有“＋”的则是有电话信息的叶节点，同时“！”号的个数
表明了它所处的层级。
为了方便处理，抽象出两个数据模型：Node，Number。Node 指所有部门节点，而Number是叶节点的电话信息。可以想见只有叶节点才能有电话信息，但叶节点不一定有电话信息。

建立数据表如下：

{% codeblock lang:sql %}

USE zjutel;

CREATE TABLE IF NOT EXISTS `Node` (
    `id`          INT auto_increment NOT NULL COMMENT '',
    `pid`         INT NOT NULL DEFAULT 0 COMMENT '父节点id',
    `title`       varchar(128) NOT NULL COMMENT '节点标题',
    `has_child`   bool NOT NULL DEFAULT FALSE COMMENT '是否有子节点',
    `is_leaf`     bool NOT NULL DEFAULT FALSE COMMENT '是否为叶节点，叶节点一定有电话号码',
    `create_date` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
    `update_date` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY(`id`),
	KEY (`title`)
)default charset=UTF8 engine=InnoDB,AUTO_INCREMENT=1;

CREATE TABLE IF NOT EXISTS `Number` (
    `id`          INT auto_increment NOT NULL,
    `nid`         INT NOT NULL,
    `number`      VARCHAR(20) NOT NULL DEFAULT '',
    PRIMARY KEY(`id`),
    FOREIGN KEY(`nid`) REFERENCES `Node`(`id`) ON DELETE CASCADE,
	KEY (`nid`),
	KEY (`number`)
)default charset=UTF8 engine=InnoDB AUTO_INCREMENT=1;

{% endcodeblock %}

然后简单地写了个PHP脚本来处理，并存入数据库。更具体的信息，请访问Github Repo: <https://github.com/rickytan/YellowPage>，所有源代码都是开放的！zjutel.db 是处理好之后的 `Sqlite` 数据库，可以直接在手机等中使用。

此电话信息的更多利用价值，有待大家一起挖掘！


