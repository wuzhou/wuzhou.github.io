---
author: wuzhou
comments: false
date: 2011-09-02 05:18:29+00:00
layout: post
slug: '%e5%9c%a8flex3%e7%9a%84module%e4%b8%ad%e4%bd%bf%e7%94%a8remoteobject%e9%9c%80%e8%a6%81%e6%b3%a8%e6%84%8f%e7%9a%84%e9%97%ae%e9%a2%98'
title: 在FLEX3的Module中使用RemoteObject需要注意的问题
wordpress_id: 15
categories:
- 编程
tags:
- Flex
- Java
---

在使用Flex的Module时，在多次载入某个Module的情况下，RomoteObject对象的方法中使用或返回的一些AS对象与Java对象绑定的信息会丢失，这个就会产生对象为null的情况。为了避免这种情况就必须在Module文件中再次注册绑定信息。


<blockquote>import flash.net.registerClassAlias;
registerClassAlias("className", className);</blockquote>


registerClassAlias方法的第一个参数为Java端对象的类名（有包名的话，也要包含在内），第二个参数为AS对象名称。

参考：[http://bbs.9ria.com/thread-39964-1-1.html](http://bbs.9ria.com/thread-39964-1-1.html)



