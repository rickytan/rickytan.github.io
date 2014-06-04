---
layout: post
title: "PHP 中读 XML 的一个坑"
date: 2014-06-05 01:47
comments: true
categories: ['php', 'xml', '微信']
---

## 问题
在做一个微信的 `PHP` 后台时需要读微信服务器 `POST` 过来的 `XML`，然后保存到 `Memcache` 中。

{% codeblock index.php lang:php %}
<?php
public index() {
	$postData = $GLOBALS["HTTP_RAW_POST_DATA"];
	$postObj = simplexml_load_string($postStr, 'SimpleXMLElement', LIBXML_NOCDATA);
	$this->cache->memcached->save("openid_".$something, $postObj->FromUserName);
	
	...
}
?>
{% endcodeblock %}

怎么也出不来结果，然后发现运行到 `save` 时有异常<!--more-->，是 `Memcache` 抛出的，说无法串行化。通过 `var_dump` 后发现 `$postObj->FromUserName` 是个 `SimepleXMLElement` 类型，而用 `var_dump` `$postObj` 时发现是 `string`！！

## 解决方法
解决方法有多种:

1. 强制转换为 `string`
	
{% codeblock lang:php %}
<?php
		$this->cache->memcached->save("openid_".$something, (string)$postObj->FromUserName);
?>
{% endcodeblock %}

2. 强制转换为 `array`

{% codeblock lang:php %}
<?php
		$postData = $GLOBALS["HTTP_RAW_POST_DATA"];;
		$postObj = (array)simplexml_load_string($postStr, 'SimpleXMLElement', LIBXML_NOCDATA);
		$this->cache->memcached->save("openid_".$something, $postObj["FromUserName"]);
?>
{% endcodeblock %}
	
3. 转为 `stdClass`

{% codeblock lang:php %}
<?php
		$postObj = simplexml_load_string($postStr, 'SimpleXMLElement', LIBXML_NOCDATA);
		$postObj = json_decode(json_encode($postObj));
?>
{% endcodeblock %}
	
这个大坑耗去了快两小时！ 

