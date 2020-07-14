---
author: wuzhou
comments: false
date: 2019-06-01 16:44:00+00:00
layout: post
slug: macos-ios-memory
title: MacOS, iOS 内存的基本概念和分析工具
categories:
- programming
tags:
- memory
---

# 一些基本概念

## Virtual Memory（虚拟内存）
虚拟内存指的是一个程序程序运行时，使用的内存空间。
虚拟内存的大小一般使用 VSS（Virtual Set Size）表示。它的大小一般这样计算：
`VSS = RSS + LSS + SwSS`
RSS 的全称为：Resident Set Size，表示当前程序进程实际使用的内存大小。
LSS 的全称为："Lazy" Set Size，表示系统同意给程序进程分配的，但是还没分配完成的内存大小。
SwSS 的全称为："Swap" Set Size，指交换内存的大小，与 MacOS 不同，iOS 没有交换内存（移动设备物理存储空间相对有限）。

## Page (内存页)
内存页是内存管理的基本单位。

# 内存分析工具

## 查看进程的虚拟内存大小
可以使用命令 `vmmap -interleaved $$`。

## 查看内存页大小
可以使用命令 `pagesize`。

## Memory Pressure（内存压力）分析
分析内存的压力首先要知道两个概念：

1. vm_page_free_count，当前系统空闲的内存页个数。
2. vm_page_free_target，系统至少需要的空闲内存页个数。

当 vm_page_free_count < vm_page_free_target 时，则表示当前系统的内存处于有压力的状态。
可以使用命令 `sysctl -a vm | grep page_free`，来查看值两个数值，从而来判断系统的内存压力的情况。

# Jetsam
Jetsam 是 MacOS 和 iOS 共有的一套内存管理机制，但是由于 iOS 没有交换内存，所以也就导致 Jetsam 在这两个平台上在面对内存压力需要释放内存时行为的不同。在 iOS 上，一旦触发了内存低事件（low RAM events），系统
会尽可能多的释放应用占用的内存，不管应用是否在运行中，这解释了为什么 iOS 的应用程序在运行时由于系统内存不足而被系统终止的情况。而在 MacOS 中，Jetsam 在面对低内存时的处理就要相对温和，它是会优先释放掉内存状态为  idle exit 的进程。

# 常见内存错误

### EXC_BAD_ACCESS exception, and the process receives a segmentation fault (SIGSEGV, Signal #11).
访问了还未被系统分配的内存，这也是在 iOS 应用崩溃中经常看到的错误类型。

# 参考文献
[No pressure, Mon!Handling low memory conditions in iOS and Mavericks](http://newosxbook.com/articles/MemoryPressure.html)