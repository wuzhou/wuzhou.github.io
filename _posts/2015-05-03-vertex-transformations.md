---
author: wuzhou
comments: false
date: 2015-05-03 17:23:00+00:00
layout: post
slug: vertex-transformations
title: 定点着色器中的顶点转换
categories:
- 计算机图形
tags:
- 计算机图形
---


游戏引擎中的顶点着色器（vertex shader）的主要任务之一就是：把模型的顶点从原始坐标系（模型的原始坐标系一般是指其3D建模工具中的坐标）转换到屏幕坐标系。这样的转换过程叫做顶点转换。

顶点转换可以分为4个步骤：

1. Modeling Transformation
2. Viewing Transformation
3. Projection Transformation
4. Cropping Transformtion

在 [Cg Programming/Vertex Transformations](http://en.wikibooks.org/wiki/Cg_Programming/Vertex_Transformations) 这篇文章中用了静物摄影的过程来类比这4个转换，这个类比对理解这些转换很有帮助。这个类比是这样的：

Modeling Transformation 是要把模型的坐标系从建模工具的坐标系转换到游戏引擎中的世界坐标系，这就相当于在静物摄影中把物体摆放到摄影场景中。

在游戏引擎中往往有摄像机（camera）这个概念，摄像机的位置会影响模型在游戏中看起来会是什么样子，所以还需把模型的坐标系从游戏世界坐标系转化为摄像机的坐标系，这个过程就叫做 Viewing Transformation，对应于静物摄影就相当于你怎么去摆放您的摄像机。

在3D游戏引擎中物体是会呈现出近大远小的视觉感受的，这种感受的形成就是依靠 Projection Transformation 来实现的，在静物摄影中这就相当于调节相机的焦距。

最后游戏中的物体还要被显示在屏幕上，这就需要把坐标系坐标转换到屏幕坐标系上，这就是 Cropping Transformtion。在静物摄影中就相当于在冲洗出照片后裁剪出符合需求大小的照片的过程。

在顶点着色器的编程中需要考的是前3个转换，最后一个转换是由游戏引擎自动完成的，从而通过编程去不能去改变。需要注意的一点：`Viewing Transformation` 和 `Viewport Transformation` 并不是一回事。

# 顶点转换矩阵
了解了顶点转换的过程之后，就需要了解转换的具体过程了。通常顶点转换是通过矩阵运算来完成的。使用矩阵运算的好处就是通过一次运算即可对顶点坐标三个维度的值进行转换。顶点转换矩阵是一个 4X4 矩阵:

![顶点转换矩阵](/assets/150503_matrix_1.png)

而原始的坐标则用一个 4X1 的矩阵来表示：

![原始坐标矩阵](/assets/2015-05-03_matirx_2.png)

那么转换的计算过程就是这样

![顶点转换](/assets/2015-05-03_transform.png)

我们可以看到转换的结果刚好是一个线性变换加上一个平移的结果，这也是顶点转换要用矩阵运算的原因。
