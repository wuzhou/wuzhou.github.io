---
author: wuzhou
comments: false
date: 2016-10-30 14:35:00+00:00
layout: post
slug: ios-certificates
title: 理解 iOS 开发中使用的证书
categories:
- programming
tags:
- iOS
---

在 iOS 开发中需要用到多种证书，在以前虽然经常用到，但没有关注这些证书是什么和为什么要使用这些证书，导致在遇到问题时总是要去查资料，不能够很快的解决问题，所以想把这些知识整理一下，记录下来。

简单来说，进行 iOS 开需要的两种类型的证书：Certificates 和 Provisioning，而根据不同的使用环境，这两种证书又有 Development 和 Distribution 两种版本。顾名思义 Development 版本用在开发环境下的，而 Distribution 版本则是用在发布环境下。下面再来说一说 Certificates 和 Provisioning 到底是什么。

# Certificates

为了确保应用来源的可靠性，Apple 使用了公私钥加密的方式。在向 Apple 申请 Certificate 时开发者首先钥需要在本地生成一对公私钥，这就是 Certificate Signing Request (CSR) 文件，私钥用来对应用进行签名，而公钥则用来对应用进行验证。把 CSR 文件上传之后，生成的 Certificate 文件就包含了相应的公钥，这个证书可以用来验证应用是不是拥有还证书私钥的开发者签名的，从而确保了安全性。

# Provisioning

程序进行打包发布的时候，Provisioning 文件会打包到应用文件中，在 Provisioning 中包括了应用 ID，验证应用可靠性的 Certificate 也就是公钥，以及该应用可在哪些设备上运行的信息。  

# 总结

这么就可以看出，Certificates 主要使用来验证开发者的，而 Provisioning 包含了 Certificate 的信息和应用 ID 所以它就可以验证该应用包是不是由正确的开发者开发的，以及应用包是不是对应用ID的包。

# 参考资料

[iOS 基础：证书介绍](http://blog.csdn.net/sampoowoo/article/details/40301461)
