---
author: wuzhou
comments: false
date: 2014-10-10 14:53:48+00:00
layout: post
slug: unity-3d-orthographic-camera-notes
title: Unity 3D 正交（Orthographic）摄像机尺寸学习笔记
wordpress_id: 228
categories:
- 编程
tags:
- Unity 3D
---

在 Unity 3D 中可以把摄像机设置为正交。正交摄像机与 Unity 3D 中普通摄像机相比没有透视效果（近大远小），所以正交相机一般可以用于 2D 游戏开发或者是 3D 游戏的 UI 开发。

在 2D 游戏开发中，有时会遇到根据屏幕的分辨率对游戏的背景进行自适应缩放的需求，这就需要对正交摄像机的尺寸也就是显示范围有一定的了解。

在正交相机中唯一与显示范围相关的属性只有一个，那就是 Size，单位为 Unity 单位。这个属性的值代表了摄像机在纵向上一半的显示范围。举例来说，如果把 Size 设置为5，那就意味着这个摄像机在纵向可以显示10个 Uinty 单位。摄像机的 Size 是不会随着屏幕的分辨率而变化。而摄像机横向的显示范围则是会发生变化的，Uinty 3D 通过屏幕的宽度除以高度获 Camera.aspect，再通过纵向的显示范围乘以 Camera.aspect 获得摄像机横向的显示范围，这就保证了摄像机的显示范围可以覆盖整个屏幕。

在通常情况下，使用 Unity 单位来对游戏中的对象（例如：Rigidbody，Collider）的大小进行调节就够了，但是要调节一些图片素材的大小时就需要考虑到像素（Pixel）和 Unity 单位之间的转换。那么问题就来了，像素和 Unity 单位之间的转换关系是怎么的呢？准确的说，他们两者之间的关系是不固定的，这取决与在导入素材时"Pixels To Units"这一属性的值，在默认情况下这个属性的数值为100，也就是说100像素等于1个 Unity 单位。

了解这些知识之后，再根据屏幕的分辨率来做游戏背景的自适应就会很容易了。首先，使用摄像机的 Size 和 aspect 计算出以 Unity 单位为单位的屏幕长宽。然后，根据"Pixels To Units"的值算出以像素为单位的长宽。同理，对于背景素材，首先通过 gameObject.render.bounds.size 获取以 Unity 单位为单位的素材大小，然后转换为像素大小。知道了背景和素材的像素大小后就可以很容易地得出他们之间的比例，最后通过调节背景的 transform.localScale 就可以达到背景自适应的目的了。
