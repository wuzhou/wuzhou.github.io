---
author: wuzhou
comments: false
date: 2014-06-29 12:32:09+00:00
layout: post
slug: swift-programming-language-note-1
title: Swift 编程语言学习笔记（一）
wordpress_id: 201
categories:
- 编程
tags:
- iOS
---

在 WWDC2014 上苹果公司推出了 OS X 和 iOS 的新开发语言 Swift。又要开始一个学习新编程语言的旅程了。但是现在 Swift 和 XCode 6 都还是处于测试阶段，要能转化为生产力还需要一段时间，所以学习的时间还是有的。而且我认为学习一门编程语言的最好方法还是要在实际项目中运用，所以这次笔记也都是一些显浅的理解和大致的概括。


# 1. 常量与变量


在我学过编程语言中，Swift 大概是最强调使用常量的一种编程语言。官方文档给出的建议是只有在值会发生变化时才需要使用变量，而使用常量会使你的代码更安全和清晰。结合我自己的编程经历来看，这样的设计是很有道理的，因为通常我在使用变量时，只是把它当作一个存放值的容器，而在很多情况下这些值都是不会变的。


# 2. 结构体（Structures）和枚举类型（Enumerations）


可以说结构体和枚举类型在 Swift 中是具体有很高地位的，因为在这两种类型不但可以存储值，还可以定义自己的方法，这就为编写程序提供了很多的可能性。特别是结构体和类（Class）在 Swfit 中已经非常相似了，但是有一个重要的不同是：在使用结构体的时候传的是值（Copy）而类则是引用（Reference）。


# 3. 可选类型（Optional）


可选类型应该是大家目前讨论的热点。我从官方的文档来看，设计和使用可选类型的思路大概是这样的：首先，在 Swift 中在定义一个变量或常量时必须保证它是非空的，但有的时候这些值很有可能为空，比如调用一个 Objective-C 的方法，返回值很有可是空，这是你就需要声明这个变量有可为空，也就是可选类型。


# 4. 和 Objective-C 的异同


在 Swift 可找到一些与 Objective-C 相类似的概念，这不过这些概念有了新的存在形式。下面是一个他们之间相关概念的对应列表：

Objective-C        Swift

id                        AnyObject

pointers              optionals

category             extension

block                  closures


# 5. 编写可以被 Objective-C 程序使用的 Swift 代码


只需要在相应的方法、属性和类的前面加上`@objc`标签即可。
