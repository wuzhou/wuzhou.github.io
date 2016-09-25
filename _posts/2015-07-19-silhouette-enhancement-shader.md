---
author: wuzhou
comments: false
date: 2015-07-19 15:09:00+00:00
layout: post
slug: silhoutte-enhancement-shader
title: 轮廓增强效果着色器
categories:
- 计算机图形
tags:
- 计算机图形
---

轮廓增强效果是在游戏中很常见的一种效果，比如：在游戏中选择一个单位，这个单位的模型的轮廓就会出现像描边一样的效果。最近在阅读 cg 编程方面的文章时发现了这种效果的一种实现方法，所以就把它记录了下来。

# 原理
实现轮廓增强效果首先要能够识别 3D 模型的边缘，本文中的实现并没去准确地判断模型的边缘，而是去找出模型视觉上的边缘，这也刚好是我们想要的效果。视觉边缘可以利用模型表面的法线（normal vector）和视线方向向量的关系去识别，当这个两个向量正交（orthogonal）时，那么我们可以认为这个点是处在视觉边缘的位置。在找出边缘点之后只需要对这些点进行增强处理即可。

# 实现
考虑实现这个着色器，第一个需要考虑的问题是「应该在那个坐标系下计算这两向量之间的夹角？」，这个问题的答案应该是在任何坐标系均可，只要这两个向量在同一坐标系下即可，考虑到计算的方便和易理解性，我们选择在世界坐标系下进行计算。

剩下的问题就是需要考虑是在「顶点着色器」还是在「块着色器」里实现增强效果，这个问题也没有确定的答案。在「顶点作色器」中处理速度更快，原因是顶点的数量相对较少，但是带来的问题就是：由于顶点的数量少，它们的法线方向的变化会比较突然（不平滑），所以增强的效果可能会差一点。因此，在性能不是着重考虑的因素时，我们选择在「块着色器」中实现增强效果。

# 细节
在实现的细节上我们可以使用向量的内积来计算两个向量之间的夹角，当内积越接近与0就说明这个点越靠近视觉的边缘区域。在 cg 编程中可以这样表示：`dot(viewDirection, normalDirection)`。

具体到增强效果的实现是通过调节颜色的透明度来实现，在程序中是这样的表示的：`min(1.0, _Color.a  / abs(dot(viewDirection, normalDirection)))` ，边缘点法线与视线方向向量的内积趋近于0，那么`_Color.a  / abs(dot(viewDirection, normalDirection))` 的值将远大于1，这时颜色的透明度则会取1，也就是不透明，这样就实现了边缘的增强效果。

[完整的 shader 代码](https://github.com/wuzhou/LearnCgPrograming/blob/master/Assets/Shaders/Cg%20silhouette%20enhancement.shader)

[参考资料](https://en.wikibooks.org/wiki/Cg_Programming/Unity/Silhouette_Enhancement)
