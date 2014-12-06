---
author: wuzhou
comments: true
date: 2012-02-26 07:00:07+00:00
layout: post
slug: windows_emacs_php-mode_config
title: Windows环境下配置Emacs的php-mode
wordpress_id: 65
categories:
- 编程
tags:
- Emacs
---

配置Emacs的各种模式（插件）需要在Emacs的配置文件.emacs中配置相关信息，Windows下配置这个文件的困难在于.emacs文件很难找的，网上的很多资料都不是Windows平台的，Windows在Geek的世界里总是显得有些非主流。有很多所谓的在Windows下配置的方法也都不太靠谱。后来看到一篇英文文章写了这方面的问题，按照上面的方法实验后成功。

在Windows平台下，需要为Emacs配置一个名为HOME用户环境变量，Emacs将会在这个变量的路径下创建相关的配置文件。所以，只要配置好了这个环境变量，就可顺利的找到.emacs配置文件了。

打开Emacs文件开始对.emacs文件进行配置。用Emacs打开这个文件后添加如下带码：

    
    (add-to-list 'load-path "c:/home")
    (require 'php-mode)


第1句的函数add-to-list是用来添加php-mode的el文件所存放的目录，有了这一句作为基础，第2句的require函数就可顺利的引用到php-mode的文件了，这样配置就完成了。
