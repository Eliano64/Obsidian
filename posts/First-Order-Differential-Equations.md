---
title: First Order Differential Equations
tags:
  - Derivative
categories:
  - Calculus
date: 2026-08-18 00:06:44
katex: true
---
> [!cite]
> 我们可能并不能找到所有一阶常微分方程的通解通法，但我们可以揣测出题人是如何构造的。

> 较为平凡的分离变量不做介绍。

# 齐次式

令$u=\frac{y}{x}$,将$\frac{\mathrm{d}y}{\mathrm{d}x}$改写为$u+x\frac{\mathrm{d}u}{\mathrm{d}x}$即可

# $\frac{\mathrm{d}y}{\mathrm{d}x}=\frac{y}{F(x,y)}$，非齐次式

分母简单，倒转计算$\frac{\mathrm{d}x}{\mathrm{d}y}$

# $\frac{\mathrm{d}y}{\mathrm{d}x}= \frac{\phi(y)}{\phi'(y)f(x)}$, 非齐次式

令$\phi(y)=u$,则化为$$\frac{\mathrm{d}u}{\mathrm{d}x}-\frac{1}{f(x)}u=0$$
# $f'(x)=f(\phi(x))$ 且$\phi^{-{1}}=\phi$型，即括号里的不一样

$f(x)$一般有二阶导。对原式求导：

$$\begin{cases}
f'(x)=f(\phi(x)); \\ f''(x)=f'(\phi(x))\cdot\phi'(x).
\end{cases}$$


将$f'(x)$写成$f'(\phi(x))$的形式，于是得到$$\begin{cases}
f'(\phi(x))=f(\phi(\phi(x)))=f(x); \\ f''(x)=f'(\phi(x))\cdot\phi'(x).
\end{cases}$$
这样就可以将括号里的内容化为一样了，最终得到$$f''(x)=f(x)\cdot\phi'(x)$$