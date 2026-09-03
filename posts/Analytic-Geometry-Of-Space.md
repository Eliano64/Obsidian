---
title: Analytic Geometry Of Space
tags:
  - Integral
categories:
  - Calculus
date: 2026-08-24 11:24:41
katex: true
---
# 点与直线

点$P$到过点$P_{0}$且方向为$\vec{\alpha}$直线的距离：以$\vec{PP_{0}},\vec{\alpha}$组成的平行四边形，底长为$|\vec{\alpha}|$，高即为距离。

# 点与面

$$d=\frac{|Ax_{0}+By_{0}+Cz_{0}+D|}{\sqrt{ A^{2}+B^{2}+C^{2}}}$$

# 直线与直线

设两条空间直线为

$$L_1:\ \mathbf r=\mathbf p+s\mathbf a, \qquad L_2:\ \mathbf r=\mathbf q+t\mathbf b, $$

其中 $\mathbf a,\mathbf b$ 是方向向量。

直线是两个平面相交的产物；两条相交直线唯一确定一个平面。

## 异面直线的距离

若这两条直线平行, 则退化到点到直线的距离.

若不平行，令

$$ \mathbf n=\mathbf a\times\mathbf b. $$

过 $L_1$ 作与 $L_2$ 平行的平面 $\Pi$。该平面法向量即为 $\mathbf{n}$ 确定，故

$$\Pi:\ (\mathbf r-\mathbf p)\cdot\mathbf n=0.$$

此时$L_2\parallel\Pi$。所以两直线的距离等于 $L_2$ 上点到 $\Pi$ 的距离。于是代入点到平面的距离公式得到

$$d= \frac{ \left|(\mathbf q-\mathbf p)\cdot(\mathbf a\times\mathbf b)\right| }{ \left\|\mathbf a\times\mathbf b\right\| }. $$

## 求公垂线

仍取

$$ \mathbf n=\mathbf a\times\mathbf b. $$

以 $L_1$ 和方向 $\mathbf n$ 构造平面 $\Pi_1$，以 $L_2$ 和方向 $\mathbf n$ 构造平面 $\Pi_2$。两个平面的方程为

$$
\Pi_1:\ (\mathbf r-\mathbf p)\cdot(\mathbf a\times\mathbf n)=0,
\qquad
\Pi_2:\ (\mathbf r-\mathbf q)\cdot(\mathbf b\times\mathbf n)=0.
$$

[html-card height=680 title=辅助平面求公垂线](../assets/spatial-lines-distance.html)

两平面的交线 $m=\Pi_1\cap\Pi_2$ 的方向为 $\mathbf n$。它与 $L_1$ 同在 $\Pi_1$ 内、与 $L_2$ 同在 $\Pi_2$ 内，故分别与两条直线相交并垂直，即为公垂线。

# 直线与平面

求直线在平面上的投影，即求过此直线且与该平面垂直的第二个平面与该平面的交线。

将过直线$l:\displaystyle\begin{cases}A_{1}x+B_{1}y+C_{1}z+D_{1}=0; \\  A_{2}x+B_{2}y+C_{2}z+D_{2}=0\end{cases}$的平面$\alpha$表示为$A_{1}xA_{1}x+B_{1}y+C_{1}z+D_{1}+\lambda(A_{2}x+B_{2}y+C_{2}z+D_{2})$时，**尤其注意这个平面簇没有包含平面$\alpha_{2}:A_{2}x+B_{2}y+C_{2}z+D_{2}=0$! 为了不遗漏，最好先带入$\alpha_{2}$看看是否能满足题设条件。**

# 直线与曲面

根据一条含参$(x_{1},y_{1},z_{1})$的母线和准线求曲面方程：

1. 母线上存在一点$A(x,y,z)$
2. $(x_{1},y_{1},z_{1})$在准线上，即 $F(x_{1},y_{1},z_{1})=0$
3. 联立12，消去$x_{1},y_{1},z_{1}$ 于是得到曲面方程。


根据一条给定母线和旋转轴求曲面方程：

1. 定直线上存在一点$A_{0}(x_{0},y_{0},z_{0})$
2. 在A点旋转后得到的曲线上再取一点$A(x,y,z)$。
3. $\vec{A_{0}A} \bot \vec{a}$ $\vec{a}$为旋转轴的一个方向向量
4. $A_{0}$到旋转轴的距离等于$A$到旋转轴的距离
5. 联立134，消去$x_{1},y_{1},z_{1}$ 于是得到曲面方程。

# 曲面的法向量

若空间曲面为$z=f(x,y)$ 形式，要写为$z-f(x,y)=0$的标准曲面形式（这样写可以保证法向量方向与$z$轴正方向夹角不超过$\frac{\pi}{2}$）$F(x,y,z)=0$。然后才能研究它在某点的法向量。

$x=f(y,z),y=f(z,x)$的情况同理。


# 方向导数

> [!example] 方向导数的存在性
> 设
> $$
> f(x,y)=\begin{cases}
> 1,&y=x^2, x\ne0,\\
> 0,&\text{其他}.
> \end{cases}
> $$
> 判断 $f$ 在 $(0,0)$ 处沿任意方向的方向导数是否存在。
>
> **求方向导数时，先固定方向 $\theta$，再令 $t\to0^+$。**
>
> 以原点为圆心、$t$ 为半径作圆。圆与抛物线交于 $A_t,B_t$；当 $t$ 改变时，$A_t,B_t$ 的方向角也随之改变。因此，抛物线上趋近原点的这些点并不位于同一固定方向上:
>
> 固定方向 $\mathbf l=(\cos\theta_{0},\sin\theta_{0})$。该方向上的点为
> $$
> (x,y)=(t\cos\theta_{0},t\sin\theta_{0}),\qquad t>0.
> $$
> 若它落在 $y=x^2$ 上，则
> $$
> t\sin\theta_{0}=t^2\cos^2\theta_{0}.
> $$
> 解得的两个交点都与$t$有关。因此存在 $\delta>0$，使得 $0<t<\delta$ 时$\theta=\theta_{0}$不再与抛物线相交，于是
> $$
> f(t\cos\theta,t\sin\theta)=0.
> $$
> 故
> $$
> \left.\frac{\partial f}{\partial l}\right|_{(0,0)}
> =\lim_{t\to0^+}\frac{f(t\cos\theta,t\sin\theta)-f(0,0)}{t}=0.
> $$
> 任意方向的方向导数都存在，且均为 $0$。

# 曲线的投影

空间曲线向某坐标面的投影，是沿垂直于该坐标面的方向，把曲线上的每一点落到坐标面上。

若曲线用参数方程

$$
\mathbf r(t)=(x(t),y(t),z(t))
$$

表示，投影时直接把垂直于目标坐标面的分量置为 $0$：

| 投影面 | 投影曲线的参数方程 |
| --- | --- |
| $xOy$ | $(x(t),y(t),0)$ |
| $xOz$ | $(x(t),0,z(t))$ |
| $yOz$ | $(0,y(t),z(t))$ |

若曲线由两个曲面的交线给出：

$$
\begin{cases}
F(x,y,z)=0,\\
G(x,y,z)=0,
\end{cases}
$$

例如，向 $xOy$ 面投影时，根据方程组用$x,y$去表示$z$，这样就消去了 $z$，得到

$$
H(x,y)=0.
$$

$H(x,y)=0$ 在空间中表示一个母线平行于 $z$ 轴的柱面。它与 $xOy$ 面相交，才得到实际的投影曲线：

$$
\begin{cases}
H(x,y)=0,\\
z=0.
\end{cases}
$$

向 $xOz$ 面投影时消去 $y$ 并补上 $y=0$；向 $yOz$ 面投影时消去 $x$ 并补上 $x=0$。