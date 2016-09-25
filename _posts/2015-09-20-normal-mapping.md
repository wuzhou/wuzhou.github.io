---
author: wuzhou
comments: false
date: 2015-09-20 10:49:00+00:00
layout: post
slug: normal-mapping
title: 理解 Normal Mapping 及其改进
categories:
- 计算机图形
tags:
- 计算机图形
---

# Normal Mapping

在现实世界中，很多物体的表面都不是光滑的，比如由砖砌成的墙面。以这种墙面为例，从大体上来看他是一个平面，但是他的表面却是有各种的凸起凹陷，那么这种不平整怎么在3D游戏中表现出来呢？最直观的想法就是在建模的时候就把这些凸起凹陷作为模型的一部分放在模型中，这种方法需要要更多地几何图形来表现这些不平整的地方，这就造成了两大问题：

1. 建模更困难成本更高了，因为需要把更多地细节放到模型中。
2. 更多的几何图形就意味着更多的顶点，那么渲染这样的模型也会对硬件造成更大的负担。

还有一种方法就是通过改变模型表面的法线的方向来改变光照效果，来模拟出凸起和凹陷的效果，也就是 Normal Mapping 方法。

# 实现

## Normal Map

在不考虑 Normal Mapping 情况下，顶点的法线信息是有内置变量提供给开发者使用，如在 Unity 3D 中的 `float3 normal : NORMAL`，这个法线信息是根据模型的形状生成的，如果模型表面是平滑的，这些法线信息就没办法用来模拟出表面粗糙的效果，所以我们还需要其他的介质来存模拟粗糙效果的法线信息，这个用来存储法线信息的介质就是 Normal Map。

Normal Map 一般可以用灰度（gray-scale）图片来表示，平滑的部分用特定的灰度来表示，凸起的部分用更高的灰度，凹陷的部分用较低的灰度。在 Unity 3D 可以用贴图自动生成相应地 Normal Map。在 Unity 生成的 Normal Map 中，法线向量的 x 轴坐标存储在 G (绿色) 通道中，而 y 轴坐标存储在 A (透明度)通道中， 而 z 轴的坐标则可以通过 x 和 y 轴的坐标计算得到。计算公式如下：

x = 2A - 1

y = 2G - 1

z = (1 - x^2 - y^2)^0.5

## 坐标转换

在 Unity 中 Normal Map 的法线向量信息的所用的坐标系并不是世界坐标系，而是顶点的表面坐标系。所谓表面坐标系（local surface coordinate）就是该坐标系的 x-y 平面是当前顶点的切平面，而 z 轴就是 x-y 平面的法线。x 轴的方向会由 Unity 提供：`float4 tangent : TANGENT` ，而法线也就是 z 轴就是 Unity 提供的该顶点的法线信息 `float3 normal : NORMAL`。由于 x, y, z 三个向量相互正交，那么 y 轴方向就可以由 x 和 z 外积得到。这样就可以构建出表面坐标系到世界坐标系的转换矩阵，即 t = [x, y, z]。

在得到转换矩阵之后，我们就可以使用它把在 Normal Map 中法线向量转换到世界坐标系，这样在光照处理的时候就可以使用转换好的法线向量，以到达我们需要的模拟粗糙表面的效果。

# 改进

Normal Mapping 在当视点的方向和模型表面垂直的时候显示效果很好，但是当视点方向和模型表面不垂直时效果就会变差，要改善这种情况方法之一就是采用 Parallax（视差） Mapping。

## Parallax Mapping

Parallax Mapping 的基本思想是：在从 Normal Map 获取像素点的法线信息时对 UV 坐标进行一定的偏移，已达到视差的效果。

比如，正常的 Normal Mapping 在获取法线信息时是这样的：

```
float4 encodedNormal = tex2D(_BumpMap,
              	 			 _BumpMap_ST.xy * input.tex.xy + _BumpMap_ST.zw);
```

而进行 Parallax Mapping 在获取法线信息时则需要加上一定的偏移量：

```
float4 encodedNormal = tex2D(_BumpMap,
               				 _BumpMap_ST.xy * (input.tex.xy + texCoordOffsets)
                             + _BumpMap_ST.zw);
```

所以进行 Parallax Mapping 的关键就是如何确定这个偏移量，下面就来讲讲这个偏移量的确定。

## 视差偏移量的确定

首先我们需要这个模拟凸起的点的相对于模型表面的高度，这些高度信息也需要用一种介质保存起来，这个介质就是 Height Map。这个高度信息是在模型表面像素点的法线方向上的。

然后我们需要利用这个高度信息对 UV 坐标进行偏移。偏移的量是由在这个点上的视点向量来决定的，也就是这个高度 h 在视点方向上的分量。那么

offsetX = h * (Vx / Vz)

offsetY = h * (Vy / Vz)

然后把这个偏移量从像素点的表面坐标系转换到 UV 坐标系就是我们所需要的偏移量了。

# 参考资料

* [Lighting of Bumpy Surfaces](https://en.wikibooks.org/wiki/Cg_Programming/Unity/Lighting_of_Bumpy_Surfaces)
* [Projection of Bumpy Surfaces](https://en.wikibooks.org/wiki/Cg_Programming/Unity/Projection_of_Bumpy_Surfaces)
