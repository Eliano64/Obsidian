---
title: Differential Equations
tags:
  - Differentiation
categories:
  - Calculus
date: 2026-08-18 00:06:44
katex: true
---
# 一阶常微分方程

> [!cite]
> 我们可能并不能找到所有一阶常微分方程的通解通法，但我们可以揣测出题人是如何构造的。

> 较为平凡的分离变量不做介绍。

## 齐次式

令$u=\frac{y}{x}$,将$\frac{\mathrm{d}y}{\mathrm{d}x}$改写为$u+x\frac{\mathrm{d}u}{\mathrm{d}x}$即可

### $\frac{\mathrm{d}y}{\mathrm{d}x}=F(\frac{a_{1}x+b_{1}y+c_{1}}{a_{2}x+b_{2}y+c_{2}})$但$c_{1},c_{2}$不同时为0


解线性方程组$$\begin{cases}
a_{1}x+b_{1}y+c_{1}=0 \\
a_{2}x+b_{2}y+c_{2} =0
\end{cases}$$

1. 方程组无解或有无穷多解，即方程最后化为$a_{1}=a_{2}=a,b_{1}=b_{2}=b$。此时令$u=ax+by$,分离常数。
2. 方程组有唯一解$(x_{0},y_{0})$。令$x=x_{0}+u,y=y_{0}+v$,则$F\left( \frac{a_{1}x+b_{1}y+c_{1}}{a_{2}x+b_{2}y+c_{2}} \right)=F\left( \frac{a_{1}x_{0}+b_{1}y_{0}+c_{1}+a_{1}u+b_{1}v}{a_{2}x_{0}+b_{2}y_{0}+c_{2}+a_{2}u+b_{2}v} \right)=F\left( \frac{a_{1}u+b_{1}u}{a_{2}u+b_{2}v} \right)$这样便化为了齐次式


## $\frac{\mathrm{d}y}{\mathrm{d}x}=\frac{y}{F(x,y)}$，非齐次式

分母简单，倒转计算$\frac{\mathrm{d}x}{\mathrm{d}y}$

## $\frac{\mathrm{d}y}{\mathrm{d}x}= \frac{\phi(y)}{\phi'(y)f(x)}$, 非齐次式

令$\phi(y)=u$,则化为$$\frac{\mathrm{d}u}{\mathrm{d}x}-\frac{1}{f(x)}u=0$$
## $f'(x)=f(\phi(x))$ 且$\phi^{-{1}}=\phi$型，即括号里的不一样

$f(x)$一般有二阶导。对原式求导：

$$\begin{cases}
f'(x)=f(\phi(x)); \\ f''(x)=f'(\phi(x))\cdot\phi'(x).
\end{cases}$$


将$f'(x)$写成$f'(\phi(x))$的形式，于是得到$$\begin{cases}
f'(\phi(x))=f(\phi(\phi(x)))=f(x); \\ f''(x)=f'(\phi(x))\cdot\phi'(x).
\end{cases}$$
这样就可以将括号里的内容化为一样了，最终得到$$f''(x)=f(x)\cdot\phi'(x)$$

# 二阶非常系数线性微分方程

> [!cite]
> 授人以鱼不如授人以渔。

先考虑齐次方程。形式可化为$y''(x)+P(x)y'(x)+Q(x)y(x)=0$的微分方程即为二阶非常系数齐次线性微分方程。

**令$t=\phi(x)$，若该换元使得二阶非常系数线性微分方程$y''(x)+P(x)y'(x)+Q(x)y(x)=0$化为二阶常系数线性微分方程，则一定有$|\phi(x)|=\sqrt{ CQ(x) }$。** $C$为某个常数。

说明：

令$t=\phi(x)$，$y(x)=u(t)$则原式化为：（为了方便消去$P(x)$和$Q(x)$，于是除了$u(t)$，系数均保留$x$）

$$u''(t)[\phi'(x)]^2+u'(t)\phi''(x)+P(x)u'(t)\phi'(x)+Q(x)u(t)=0$$
整理得：

$$u''(t)+\frac{\phi''(x)+P(x)\phi'(x)}{[\phi'(x)]^2}u'(t)+\frac{Q(x)}{[\phi'(x)]^2}u(t)=0$$


出题人出的二阶非常系数线性微分方程一定可以化为二阶常系数微分方程。例如欧拉方程。

于是可以直接看$\frac{Q(x)}{[\phi'(x)]^2}$项。可以直接令$\frac{Q(x)}{[\phi'(x)]^2} =C\implies |\phi'(x)|=\sqrt{C Q(x) }$ 。$C$为任意常数，肯定是怎么方便怎么来。于是解出$\phi(x)$。

当然$\phi(x)$一定使得$u'(t)$前更复杂的系数为常数，否则这题没法做了嘛。
