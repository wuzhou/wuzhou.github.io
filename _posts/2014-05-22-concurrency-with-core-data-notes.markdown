---
author: wuzhou
comments: false
date: 2014-05-22 09:25:49+00:00
layout: post
slug: concurrency-with-core-data-notes
title: Concurrency with Core Data 笔记
wordpress_id: 184
categories:
- 编程
tags:
- iOS
---

Core Date 的并发也就是指在多线程中使用 Core Data。

**方法一：使用事件通知**

官方文档推荐的方法是在每一个线程中都单独创建一个 managed object context, 同时在不同线程中的 managed object context 共享同一个persistent store coordinator(NSPersistentStoreCoordinator).

首先创建NSManagedObjectModel 对象和 NSPersistentStoreCoordinator 对像。NSPersistentStoreCoordinator 对像可以由 initWithManagedObjectModel 完成初始化。

然后不同的线程可以创建不同的 NSManagedObjectContext 对象用于对 Core Data 进行操作。

使用 NSNotificationCenter 对 NSManagedObjectContextDidSaveNotification 时间进行监听并处理。在通知中的 userInfo 中包含有其他线程对数据的操作内容。同时还可以使用mergeChangesFromContextDidSaveNotification 方法合并更新。

**方法二：在创建 Context 的时候使用 initWithConcurrencyType 方法。**

ConcurrencyType 的类型主要有三种：

1. Confinement. 每个线程独立有独立的 Context，主要是为了兼容以前的设计。

2. Private queue. 这种 Context 会创建和管理自己的 queue，不会阻塞主线程。

3. Mian queue. 这种 Context 在主线程中运行，所以与 UI 直接相关的 Context 可以使用这种类型。

同时每个 Context 可以设置父 Context (parent context)。父 Context 会接收到子 Context 的变化，宾且和并这种变化。在 Context 中使用 performBlock: 和 performBlockAndWait: 方法对数据进行操作。performBlock: 方法会在它自己的线程中立即执行代码并返回的结果。performBlockAndWait: 方法同样是在自己的线程中执行代码，只不过不是立即执行和返回。

参考：

1. [说说iOS的多线程Core Data](http://blog.leezhong.com/ios/2013/06/16/ios-multi-thread-core-data.html)

2. [NSManagedObjectContext Class Reference](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/CoreDataFramework/Classes/NSManagedObjectContext_Class/NSManagedObjectContext.html#//apple_ref/occ/cl/NSManagedObjectContext) 

3. [The Concurrent Core Data Stack](http://floriankugler.com/blog/2013/4/2/the-concurrent-core-data-stack)

4. [How do I create a child NSManagedObjectContext?](http://stackoverflow.com/questions/12271464/how-do-i-create-a-child-nsmanagedobjectcontext/12272022#)
