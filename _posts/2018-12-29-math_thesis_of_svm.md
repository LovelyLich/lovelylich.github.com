---
layout:     post
title:      "数学角度看支持向量机"
date:       2018-12-29 12:00:00
author:     "Yi"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 机器学习
---

## 点到超平面的距离计算
SVM是基于超平面来分类，核心目标是寻找某个超平面，被分类的所有点，到该超平面的距离最大。
## 欧式空间中两点之间的距离
在欧几里得空间中，点 **p** =$(p_1, p_2,\ldots, p_n)$和 **q** =$(q_1, q_2,\ldots,q_n)$之间的欧氏距离为:

$$
\begin{aligned}
d({\bf{p}},{\bf{q}})&=\sqrt{(p_1-q_1)^2+(p_2-q_2)^2+\cdots+(p_n-q_n)^2}\\
                  &=\sqrt{\sum_{i=1}^n(p_i-q_i)^2}
\end{aligned}
$$

由该定理可知，欧式空间中的点，都对应着一个从原点出发，终止于该点的向量，而该向量的长度即为该点到原点到距离：

$$
\lVert{\bf{p}}\rVert=\sqrt{p_1^2+p_2^2+\cdots+p_n^2}
$$

## 欧式空间中点到超平面的距离
平面的一般方程式为：

$$
Ax+By+Cz+D=0
$$

其中n = (A, B, C)是平面的法向量，D是将平面平移到坐标原点所需距离（所以D=0时，平面过原点）
所以，对于计算任意点**P**=(x<sub>1</sub>, y<sub>1</sub>,z<sub>1</sub>)到该超平面的距离，其实是计算该点投影到法向量(A, B, C)上的值:

$$
d=\frac {\lvert Ax_1+By_1+Cz_1+D \rvert} {\sqrt{A^2+B^2+C^2}}
$$

## 支持向量机的训练目标
所以，支持向量机是为了寻找某个超平面 Ax+By+Cz+D=0，使得所有点到该平面的距离最大。

### 凸集
引用自维基百科:
>在点集拓扑学与欧几里得空间中，凸集（convex set）是一个点集合，其中每两点之间的直线点都落在该点集合中

凸集和非凸集合的几何表现：

|凸集|非凸集|
|:-------------------------:|:-------------------------:|
|![post_convex_img.png](/img/in-post/post_convex_img.png)|![post_non_convex.png](/img/in-post/post_non_convex.png)|

## 凸函数定义
引用自维基百科定义:
>凸函数是一个定义在某个向量空间的凸子集C（区间）上的实值函数，如果在其定义域C 上的任意两点x,y，以及$t \in [0,1]$，有:
$f(tx+(1-t)y) \leq tf(x)+(1-t)f(y)$
也就是说，一个函数是凸的当且仅当其上境图（在函数图像上方的点集）为一个凸集。

具体地，凸函数如下图所示：

![post_convex_function.png](/img/in-post/post_convex_function.png)

定义比较抽象，但如果设$t=\frac 1 2$，则能推导出我们熟悉的凸函数定义:

$$
f(\frac {x+y} 2) \leq \frac {f(x)+f(y)} 2
$$

## 凸优化
引用自维基百科:
>令$\mathcal{X}\subset \mathbb{R^n}$为一凸集，且$f:\mathcal {X}\to \mathbb {R}$为一凸函数。凸优化就是要找出一点$x^{\ast}\in {\mathcal {X}}$，使得每一$x\in {\mathcal {X}}$满足$f(x^{\ast })\leq f(x)$称为全局最优值，或全域最佳解。
>或者可以表示为下面的标准型：
>min f(x)
>subject to $g_{i}(x) \leq 0, i=1,2,\ldots,m$
>其中$f,g_{1}\ldots g_{m}:\mathbb {R} ^{n}\rightarrow \mathbb {R}$为凸函数

### 无约束优化
### 等式约束优化
### 不等式约束优化
### SVM 中的约束优化
SVM的求解最佳超平面问题，最终化为求解该约束优化问题：

$$
\begin{aligned}
& &&\min_{w,b}\frac{1}{2}{\left \| w \right \|}^2\\
&\text{s.t.} &&y_i(w^Tx_i+b)\geqslant1, i=1,2,\cdots,m
\end{aligned}
$$

所以, 该问题是凸二次规划问题

这是针对通常凸二次规划的解法，我们也可以使用拉格朗日乘子解决最小值
### 拉格朗日乘子


