---
title: Improper integral
tags:
  - Integral
categories:
  - Calculus
date: 2026-07-26 13:22:43
katex: true
---

# 反常积分定义

反常积分与普通定积分的区别只有两种：

1. 积分区间伸向无穷；
2. 被积函数在某个端点或内部点趋于无穷。

瑕点都必须改写成极限。例如

$$
\int_a^{+\infty}f(x)\,dx
=\lim_{R\to+\infty}\int_a^R f(x)\,dx,
$$

$$
\int_a^b f(x)\,dx
=\lim_{\varepsilon\to0^+}
\int_{a+\varepsilon}^b f(x)\,dx
$$

右侧极限为有限数时，反常积分收敛；极限为无穷或不存在时，反常积分发散。这里判断的是**截断积分的极限**，不是 $\lim f(x)$。

若 $c\in(a,b)$ 是瑕点，必须拆成

$$
\int_a^c f(x)\,dx+\int_c^b f(x)\,dx,
$$

并要求两个单侧积分分别收敛。

> [!danger] 不能用正负抵消代替收敛
> $\displaystyle\int_{-1}^1\frac{dx}{x}$ 的两个单侧积分均发散，因此反常积分发散；对称计算得到的 $0$ 只是柯西主值。

能通过求原函数然后取极限计算得出定积分者，计算出值直接判断是否收敛即可。本文主要讨论无法轻易算出具体定积分值的情况。

# 等价比较

设 $f,g\ge0$，并且在同一瑕点附近

$$
\lim\frac{f(x)}{g(x)}=C,\qquad 0<C<+\infty.
$$

则充分靠近瑕点时$f(x) \sim Cg(x)$

积分后仍保持该性质。所以 $\int f$ 与 $\int g$ 同敛散。

**该比较要求函数在瑕点附近同号；振荡积分不能直接这样处理。**

## $g(x)$选取

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
> 因为 $-\dfrac12>-1$ 且 $-\dfrac32<-1$，两部分均收敛。



# 基准幂积分

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

## 临界幂次与对数

$\alpha=-1$ 时，幂积分变成对数并发散。若还有对数因子，不能把它丢掉：

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

**收敛是绝对收敛的必要条件。**
## Dirichlet 判别

Dirichlet 判别处理“振荡项有抵消，但振幅逐渐减小”的情形。

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

> [! success] Proof
> 对它分部积分
> $$
> \int_A^{+\infty} h(x)\phi(x)\,dx
=H(R)\phi(R)-\int_A^{+\infty} H(x)\phi'(x)\,dx.
> $$
> 第一项趋于 $0$；$H$ 有界，而单调性使 $\int_A^{+\infty}|\phi'(x)|dx<+\infty$，故第二项收敛。

> [!example] $\displaystyle\int_0^{+\infty}\frac{\sin x}{x^p}\,dx$
> - $x\to0^+$：$\dfrac{\sin x}{x^p}\sim x^{1-p}$，要求 $p<2$；
> - $x\to+\infty$：Dirichlet 判别要求 $p>0$，绝对收敛要求 $p>1$。
>
> 所以：
> - $0<p\le1$：条件收敛；
> - $1<p<2$：绝对收敛；
> - 其余情形发散。

