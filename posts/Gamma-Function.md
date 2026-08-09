---
title: Gamma Function
tags:
  - Integral
categories:
  - Calculus
date: 2026-08-08 17:53:48
katex: true
---
# $\Gamma$ 函数

$$\Gamma(\alpha)=\int_{0}^{+\infty} x^{\alpha-1}e^{-x} \, \mathrm{d}{x} $$

它的性质十分有趣。

# $\Gamma (\alpha)$ 值

考虑递推式

$$\begin{align}
\Gamma(\alpha)&= -x^{\alpha-1}e^{-x} |_{0}^{+\infty} + (\alpha -1) \int_{0}^{+\infty} x^{\alpha-2}e^{-x} \, \mathrm{d}{x}  \\
&= (\alpha -1) \Gamma(\alpha-1), \alpha>1
\end{align}  $$

且 $\Gamma(1)=1$ 。

所以对于正整数$n$，可得$$\Gamma(n)=(n-1)!,\quad n\geqslant {1}$$