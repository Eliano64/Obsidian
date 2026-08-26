---
title: Improper Integral
tags:
  - Integral
categories:
  - Calculus
date: 2026-07-26 13:22:43
katex: true
---

# 反常积分定义

反常积分的积分区间有两种情况：

1. 积分区间无穷大；
2. 被积函数在某个端点或内部点趋于无穷。

**瑕点：被积函数在该点附近无界。**

实质是看黎曼积分在该不满足黎曼积分定义点（瑕点）处是否存在极限。

**瑕点都必须先改写成极限。** 

例如

$$
\int_a^{+\infty}f(x)\,dx
=\lim_{R\to+\infty}\int_a^R f(x)\,dx,
$$

$$
\int_a^b f(x)\,dx
=\lim_{\varepsilon\to0^+}
\int_{a+\varepsilon}^b f(x)\,dx
$$

右侧极限为有限数时，反常积分收敛；极限为无穷或不存在时，反常积分发散。

> [!tip] 收敛的 Cauchy 描述
> 见[[../drafts/Infinite-Series#收敛的定义与 Cauchy 准则|Cauchy准则]]
> 
> $\int_a^{+\infty}f(x)\,dx$要收敛，即足够靠后以后，不管积分区间多么大，这一整段的积分都必须任意小。等价于：$$\forall\varepsilon>0,\ \exists N,\ \forall n>m\ge N:\ \left|\int_{m}^{n} f(x) \, \mathrm{d}x\right|<\varepsilon$$
> 在瑕点 c 处收敛的情况同理。

若 $c\in(a,b)$ 是瑕点，必须拆成

$$
\int_a^c f(x)\,dx+\int_c^b f(x)\,dx,
$$

并要求两个单侧积分分别收敛。

> [!danger] 不能用正负抵消代替收敛
> $\displaystyle\int_{-1}^1\frac{dx}{x}$ 的两个单侧积分均发散，因此反常积分发散；对称计算得到的 $0$ 只是柯西主值。

能通过求原函数然后取极限与夹逼法计算得出定积分者，计算出值直接判断是否收敛即可。本文主要讨论无法轻易算出具体定积分值的情况。

# 幂函数的积分

有限点附近：

$$
\int_\varepsilon^\delta x^\alpha\,dx
=
\begin{cases}
\dfrac{\delta^{\alpha+1}-\varepsilon^{\alpha+1}}{\alpha+1},
&\alpha\ne-1,\\[6pt]
\ln\dfrac{\delta}{\varepsilon},
&\alpha=-1.
\end{cases}
$$

令 $\varepsilon\to0^+$ 可得

$$
\int_0^\delta x^\alpha\,dx
\text{ 收敛}\iff\alpha>-1.
$$

无穷远处同理：

$$
\int_A^{+\infty}x^\alpha\,dx
\text{ 收敛}\iff\alpha<-1.
$$

因此

| 位置        | 收敛条件        |
| --------- | ----------- |
| 有限点 $a$   | $\alpha>-1$ |
| $+\infty$ | $\alpha<-1$ |
# 比较审敛
## 等价比较

设 $f,g$在同一瑕点附近同号，且

$$
\lim\frac{f(x)}{g(x)}=C,\qquad 0<C<+\infty.
$$

根据保号性，充分靠近瑕点时$f(x) \sim Cg(x)$

设$a$为正常端点，$b$为瑕点，则 $\int_{a}^{b} f(x)\, \mathrm{d}x=\int_{a}^{b-\epsilon} f(x) \, \mathrm{d}x+C\int_{b-\epsilon}^{b} g(x) \, \mathrm{d}x$,$\epsilon$为充分小的正数。

所以$\int_{}^{} f(x) \, \mathrm{d}x$与$\int_{}^{} g(x) \, \mathrm{d}x$同敛散。

**该比较要求函数在瑕点附近同号。**

## 不等价比较

其他条件同上，如果$f(x)\leqslant g(x)$，且$\int_{}^{} g(x) \mathrm{d}x$收敛，则$\int_{}^{} f(x) \, \mathrm{d}x$收敛；如果$f(x) \geqslant g(x)$，且$\int_{}^{} g(x) \mathrm{d}x$发散，则$\int_{}^{} f(x) \, \mathrm{d}x$发散。

> [!success] Proof
> %%TODO%%

## $g(x)$选取

一般选择幂函数。

有限点使用等价无穷小：

$$
\sin x\sim x,\qquad
1-\cos x\sim\frac{x^2}{2},\qquad
\ln(1+x)\sim x,\qquad
e^x-1\sim x.
$$

$$
\frac{1-\cos x}{x^p}
\sim\frac12x^{2-p}
\Longrightarrow
\int_0^1\frac{1-\cos x}{x^p}\,dx
\text{ 收敛}\iff p<3.
$$

无穷远处取最高阶项；根式必要时先有理化。增长速度：

$$
\ln x\ll x^\alpha\ll a^x
\qquad(\alpha>0,\ a>1).
$$

$$
\frac{x^m}{(1+x)^n}\sim x^{m-n}
\Longrightarrow
\int_1^{+\infty}\frac{x^m}{(1+x)^n}\,dx
\text{ 收敛}\iff m-n<-1.
$$



> [!example] $\displaystyle\int_0^{+\infty}\frac{dx}{\sqrt{x}(1+x)}$
> 瑕点为 $0$ 与 $+\infty$：
> $$
> \frac{\dfrac1{\sqrt{x}(1+x)}}{x^{-1/2}}
> =\frac1{1+x}\to1
> \qquad(x\to0^+),
> $$
> $$
> \frac{\dfrac1{\sqrt{x}(1+x)}}{x^{-3/2}}
> =\frac{x}{1+x}\to1
> \qquad(x\to+\infty).
> $$
> 因为 $\int_{0}^{\epsilon}x^{ -\frac{1}{2} }  \, \mathrm{d}x$ 和 $\int_{A}^{+\infty}x^{-\frac{3}{2}}  \, \mathrm{d}x$均收敛，故原反常积分收敛。

# 含$\ln x$与幂函数的反常积分

1. 若为整式则分部积分。
2. 若为分式，一般令$\ln x = t$，则有 $\mathrm{d}t=\frac{1}{x}\mathrm{d}x$ 就可以提出$\frac{1}{t}$项。

如

$$
\int_0^\delta
\frac{dx}{x\left(\ln\dfrac1x\right)^q}.
$$

令 $u=\ln\dfrac1x$，则 $dx/x=-du$，原积分化为

$$
\int^{+\infty}\frac{du}{u^q},
$$

故

$$
\int_0^\delta
\frac{dx}{x\left(\ln\dfrac1x\right)^q}
\text{ 收敛}\iff q>1.
$$

无穷远处的 $\displaystyle\int_A^{+\infty}\frac{dx}{x(\ln x)^q}$ 结论相同。

再看               

$$
\int_0^\delta
\frac{dx}{x^q\left(\ln x\right)}.
$$

不妨取 $0<\delta<1$。令

$$
u=\ln x,\qquad x=e^{u},\qquad dx=e^{u}du,
$$

当 $x\to0^+$ 时，$u\to-\infty$；当 $x=\delta$ 时，$u=\ln\delta$。因此

$$
\int_0^\delta
\frac{dx}{x^q\ln x}
=
\int_{-\infty}^{\ln\delta}
\frac{e^{(1-q)u}}{u}\,du.
$$

- $q<1$ 时，$1-q>0$。当 $u\to-\infty$ 时，$e^{(1-q)u}$ 指数衰减，积分收敛；
- $q=1$ 时，积分化为 $\displaystyle\int_{-\infty}^{\ln\delta}\frac{du}{u}$，发散；
- $q>1$ 时，$1-q<0$。当 $u\to-\infty$ 时，$e^{(1-q)u}/u$ 的绝对值趋于无穷，积分发散。

因此

$$
\int_0^\delta
\frac{dx}{x^q\ln x}
\text{ 收敛}\iff q<1.
$$

无穷远处取 $A>1$，仍令 $u=\ln x$：

$$
\int_A^{+\infty}
\frac{dx}{x^q\ln x}
=
\int_{\ln A}^{+\infty}
\frac{e^{(1-q)u}}{u}\,du.
$$

- $q>1$ 时，$1-q<0$，指数因子随 $u\to+\infty$ 衰减，积分收敛；
- $q=1$ 时，积分化为 $\displaystyle\int_{\ln A}^{+\infty}\frac{du}{u}$，发散；
- $q<1$ 时，$e^{(1-q)u}/u$ 趋于无穷，积分发散。

因此

$$
\int_A^{+\infty}
\frac{dx}{x^q\ln x}
\text{ 收敛}\iff q>1.
$$

两端的条件正好相反：

| 瑕点 | 收敛条件 |
| --- | --- |
| $x\to0^+$ | $q<1$ |
| $x\to+\infty$ | $q>1$ |

临界情形 $q=1$ 在两端都会化为 $\int du/u$，所以一次方的对数因子不足以使积分收敛。

# 振荡积分

## 绝对收敛


$$
\int |f(x)|\,dx<+\infty
\Longrightarrow
\int f(x)\,dx\text{ 绝对收敛}.
$$

> [! success] Proof
> $$\int_{}^{} f(x)\, \mathrm{d}{x}= \int_{}^{} f(x)+|f(x)|-|f(x)|\, \mathrm{d}{x}$$
> 而
> $$0 \leqslant \int_{}^{} f(x)+|f(x)|\, \mathrm{d}{x} \leqslant 2\int |f(x)|\,\mathrm{d}{x}$$  

**绝对收敛一定收敛**
## Dirichlet 判别

> [! info] 引理
若
>
> $$
> H(x)=\int_A^x h(t)\,dt
> $$
> 
> 有界，且 $\phi(x)$ 单调趋于 $0$，则
> $$
> \int_A^{+\infty}h(x)\phi(x)\,dx
> $$
> 收敛。

即要求 $\displaystyle\lim_{ x \to \infty }h(x)\phi(x)=0$ 以及 $| h(x)\phi(x)|$ 单减， 类似于[[../drafts/Infinite-Series#Leibniz 判别法 (充分条件)|交错级数的 Leibniz 判别]]。

> [! success] Proof
> 对它分部积分
> $$
> \int_A^{+\infty} h(x)\phi(x)\,dx
=H(R)\phi(R)-\int_A^{+\infty} H(x)\phi'(x)\,dx.
> $$
> 第一项趋于 $0$；$H$ 有界，而单调性使 $\int_A^{+\infty}|\phi'(x)|dx<+\infty$，于是根据[[#不等价比较]]得出第二项绝对收敛。故第二项收敛。

> [!example] $\displaystyle\int_0^{+\infty}\frac{\sin x}{x^p}\,dx$
> - $x\to0^+$：$\dfrac{\sin x}{x^p}\sim x^{1-p}$，要求 $p<2$；
> - $x\to+\infty$：Dirichlet 判别要求 $p>0$，绝对收敛要求 $p>1$。
>
> 所以：
> - $0<p\le1$：条件收敛；
> - $1<p<2$：绝对收敛；
> - 其余情形发散。
