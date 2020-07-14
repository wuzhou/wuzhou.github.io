---
author: wuzhou
comments: false
title: My First Page
layout: post
slug: summary-of-3d-rotation
title: 关于3D旋转的一点总结和思考
categories:
- 计算机图形
tags:
- 计算机图形
---

<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>

## 2D 旋转

2D旋转是3D旋转的一个特例，就是在 xy 平面上绕着 z 轴逆时针旋转。

旋转矩阵可以表示为：$$\begin{pmatrix} cos(\theta) & -sin(\theta) \\ sin(\theta) & cos(\theta) \end{pmatrix}$$

其 3D 形式就是：$$\begin{pmatrix} cos(\theta) & -sin(\theta) & 0\\ sin(\theta) & cos(\theta) & 0 \\ 0 & 0 & 1\end{pmatrix} $$

如果旋转点在原点，直接运用旋转矩阵进行旋转即可。如果旋转点不在原点，则首先将旋转点移动到原点，进行旋转后在移动到原来的位置。

## 3D 旋转

### 欧拉变换
使用矩阵表示欧拉变换时会遇到“万向锁问题”。“万向锁问题”并不是旋转在物理上遇到的问题，而是在使用矩阵表示欧拉旋转变换时遇到的旋转矩阵自由度丢失的问题。

原因是：欧拉旋转变换，是以自身坐标轴为基础进行变换的，先旋转的轴，会带动还没有旋转的进行选择。当旋转第一个轴时，三个轴之间的相对位置是不变的，这时候不会造成自由度丢失，而旋转第二个轴时则有可能会把第三个轴旋转和第一个轴相同的方向上去，造成了自由度的丢失。

### 以任意向量为轴进行旋转
向量 v 以任意轴 e 进行旋转角度 o，可以把 v 分解为平行与 e 的 v1 和垂直于 e 的 v2。那么 v 以 e 轴旋转后的向量就等于 v1 和 v2 旋转之后的向量的和。由于 v1 和 e 平行，所以保持不变。v2 则需要在与 e 垂直的平面上以 e 为轴旋转角度 o。这个平面可由 v1 归一化后的向量 b 和 v1 与 e 叉乘得到的向量 c 决定，那么旋转后的 v2 就等于 $\lVert v2\rVert cos(o) b +  \lVert v2\rVert sin(o) c$。这个方法叫做 Rodrigues 旋转公式。

### 使用四元数表示旋转
四元组表示旋转在原理上来说就是构造了一种类似于复数的形式来表示旋转。它可以避免使用矩阵的欧拉变换带来的“万向锁”问题，同时可以使旋转的插值更加容易。
