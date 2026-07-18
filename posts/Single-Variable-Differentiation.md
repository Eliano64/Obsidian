---
title: Single Variable Differentiation
tags:
  - Calculus
  - Derivative
  - Limit
  - Continuity
categories:
  - Mathematics
date: 2026-07-18 13:59:09
katex: true
---
# 导数定义

$$
f'(x_0)=\lim_{x\to x_0}
\frac{f(x)-f(x_0)}{x-x_0}.
$$

一点导数存在的充要条件：

$$
f'_-(x_0)=f'_+(x_0),
$$

其中两者都是差商的单侧极限。**注意这不等于$f'(x)$ 的左右极限。反例见[[#^e7d1e0|衰减振荡函数]]**

判断顺序：

1. 先查 $f(x_0)$ 与左右极限；不连续则必不可导。
2. 连续后再算左右差商。
3. 两侧差商极限有限且相等，才可导。

连续只是必要条件，但不连续一定不可导。


> [!danger] $f'(x_0)$ 与 $f'(x)$ 不要混淆 
>
> $f'(x_0)$ 是一个**数**；$f'(x)$ 是**函数**。对 $x\ne x_0$ 求出的公式在 $x_0$ 无定义，不代表 $f'(x_0)$ 不存在。
> 
> 同样地，$\displaystyle\lim_{ x \to x_{0}^{\cdot} }f'(x)\neq f'_{\cdot}(x), \cdot = + \text{ or } -$，见[[#^e7d1e0|衰减振荡函数]]
> 
> 所以求分类函数的分界点的导数必须单独用定义计算。即计算$f'_{-}(x_{0})$和$f'_{+}(x_{0})$而不能直接使用$f'(x)$的$x_{0}$左右两侧极限。

# 导函数的间断

导函数不一定连续，但具有 Darboux 性质，即不能跳过中间值。因此**导函数只能连续或存在振荡间断点**。

**若导函数在一点附近有有限极限，该极限必须等于该点导数。反之不成立，见[[#衰减振荡函数]]**

有第一类间断点和无穷间断点的函数不可能是某个函数的导函数，也就没有原函数。

# 衰减振荡函数

常使用以下函数构造反例。

$$
f(x)=
\begin{cases}
Ax^m\sin\left(\dfrac B{x^n}\right),&x\ne0,\\
0,&x=0,
\end{cases}
\qquad A\ne0.
$$

由 $|\sin u|\le1$：

- $m>0$ 时$f(x)$在 $0$ 连续且$f(0)=0$；
- $m>1$ 时 $f'(0)=0$。

对 $x\ne0$，

$$
f'(x)=Amx^{m-1}\sin\left(\frac B{x^n}\right)
-ABnx^{m-n-1}\cos\left(\frac B{x^n}\right).
$$

所以根据指数 $m-n-1$ 判断导函数在 0 附近的存在性：

- $m-n-1>0$：导函数连续，$\displaystyle\lim_{ x \to0 }f'(x)$存在；
- $m-n-1\leqslant0$：导函数不连续，$\displaystyle\lim_{ x \to 0 }f'(x)$不存在但$f'(0)=0$，即存在$f'_{+}(0)=0$与$f'_{-}(0)=0$； ^e7d1e0

由于$m>1$ 时 $f'(0)=0$。常与与多项式 $q(x)$ 相加，使得$f'(0)+q'(0)=q'(0)$，同时让一些$q'(x)$一些显然满足的性质失效。

> [!info] 有界振荡函数$\displaystyle\sin\left( \frac{1}{x} \right)$
> 实际上衰减振荡函数是考虑使用了$x^{m}$这个衰减因子使得有界振荡函数能在 0 处收敛为 0。衰减振荡函数之所以那样有趣，很大一部分来自于它的有界振荡函数父亲。
> 关于有界振荡函数的性质，也有一些很经典的反例，例如：
> 我们知道，有界函数乘无穷小量等于0。那么:
> $f(x)$在 0 的去心邻域连续，且$\displaystyle\lim_{ x \to 0 }f(x)\cdot \sin\left( \frac{1}{x} \right)=0$, 我们可以说$\displaystyle\lim_{ x \to 0 }f(x)=0$吗？
> 考虑是否存在反例。$\sin \frac{1}{x}$有无穷零点，一个很显然的构造思路就是，在零点处$f(x)$为一个固定不变的值$A$，而非零点处$f(x)$趋于0。
> > 当$\sin \frac{1}{x}$为0时，$f(x)$为$A$，当$\sin \frac{1}{x}$非0时，$f(x)$趋于0.
> 
> 一种构造即是：
> $$
> f(x)= \frac{x}{|x|+|\sin \frac{1}{x}|}
> $$
# 洛必达的限制

在 $0/0$ 或 $\infty/\infty$ 型下，满足洛必达条件时，

$$
\lim\frac{f'}{g'}=L
\Rightarrow
\lim\frac fg=L.
$$

**反推不成立**。题设若只给出 $\lim f/g$，不能据此断言 $\lim f'/g'$ 存在。见[[#^24468e|例题]]。

# 例题

> [!example] 题目
> $f$ 在 $(-1,1)$ 可导，且
> $$
> \lim_{x\to0}\frac{f(x)}{x^2}=1.
> $$
> 判断下列结论：
> 
> A. $\displaystyle\exists\lim_{x\to0}\frac{f'(x)}x$
> 
> B. $\displaystyle\not\exists\lim_{x\to0}\frac{f'(x)}x$
> 
> C. $f'(0)=0,\ f''(0)=2$
> 
> D. $f(0)$ 为 $f(x)$ 的极小值

由可导 $\Rightarrow$ 连续及已知极限，得 $f(0)=0$。再由导数定义，

$$
f'(0)=\lim_{x\to0}\frac{f(x)}x
=\lim_{x\to0}x\frac{f(x)}{x^2}=0.
$$

当 $x$ 充分接近 $0$ 且 $x\ne0$ 时，$f(x)/x^2>0$，故 $f(x)>f(0)=0$，D 正确。

A、B 都不能由题设单独确定：$f(x)=x^2$ 时 A 成立、B 不成立；下面的反例中 A 不成立、B 成立。C 也不一定成立。利用[[#衰减振荡函数]]构造

$$
f(x)=x^2+x^4\sin\frac1{x^2}\quad(x\ne0), \qquad f(0)=0,
$$

则

$$
\frac{f'(x)}x
=2+4x^2\sin\frac1{x^2}-2\cos\frac1{x^2},
$$

极限不存在。

这也说明 $f(x)/x^2\to1$ 不能反推出 $f'(x)/x$ 的极限；同样不能据此断言 $f''(0)=2$。 ^24468e
