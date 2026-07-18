---
title: Higher Order Derivatives
tags:
  - Calculus
  - Derivative
categories:
  - Mathematics
date: 2026-07-18 13:59:09
katex: true
---


# $F(q(x))$

当 $q(x)$ 是多项式，且函数形如

$$
e^{q(x)},\quad \frac1{1+q(x)},\quad
\ln(1+q(x)),\quad \sin q(x),\quad \cos q(x),
$$

先对外层变量 $u=q(x)$ 泰勒展开：

$$
e^u=\sum_{k\ge0}\frac{u^k}{k!},\qquad
\frac1{1+u}=\sum_{k\ge0}(-1)^ku^k.
$$

若 $q(x)$ 的最低次数为 $r$，求 $n$ 阶导时只需保留 $kr\le n$ 的 $q(x)^k$。

对数函数可先求导：

$$
[\ln(1+x)]'=\frac1{1+x}=1-x+x^2-\cdots,
$$

故

$$
[\ln(1+x)]^{(n)}\big|_{x=0}=(-1)^{n-1}(n-1)!.
$$

三角函数可用

$$
\sin x=\frac{e^{ix}-e^{-ix}}{2i},\qquad
\cos x=\frac{e^{ix}+e^{-ix}}2.
$$

只求零点导数时直接用奇偶次幂；处理 $e^{ax}\sin bx$、$e^{ax}\cos bx$ 时再用复指数。

# $g(x)q(x)$：莱布尼茨公式截断

$$
(gq)^{(n)}=sum_{k=0}^{n}\binom nk g^{(n-k)}q^{(k)}.
$$

若 $q$ 是 $m$ 次多项式，则 $q^{(k)}=0\ (k>m)$：

$$
(gq)^{(n)}=
\sum_{k=0}^{\min(m,n)}\binom nk g^{(n-k)}q^{(k)}.
$$

例如

$$
(x^2e^x)^{(n)}
=e^x\left[x^2+2nx+n(n-1)\right].
$$

# $F(x,n) = \prod_{i=1}^{n}g(x,i)$

$$\begin{align}
F'(x,n)&=(\pi_{i=1}^{n-1}g(x,i))_{x}'g(x,n)+(\pi_{i=1}^{n-1}g(x,i))g'(x,n) \\
&=F'(x,n-1)g(x,n)+F(x,n-1)g_{x}'(x,n)
\end{align}$$

找递推公式化简通项公式即可。