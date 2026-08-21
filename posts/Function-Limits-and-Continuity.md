---
title: Function Limits And Continuity
tags:
  - Limit
  - Continuity
categories:
  - Calculus
date: 2026-07-18 13:59:09
katex: true
---
# 等价无穷小
## $0/0$

使用泰勒展开，写成

$$
P(x)=a_m(x-x_0)^m+o((x-x_0)^m),
$$

$$
Q(x)=b_n(x-x_0)^n+o((x-x_0)^n).
$$

分母只保留次数最小的项作为主项，分子展开式只保留次数小于等于主项次数的项。

### 定积分与重积分

例如$\displaystyle\lim_{ t \to 0 } \frac{F(t,x,y)}{t^{n}}$型，其中$n$为固定的常数，$F(t,x,y)$是一个以$t$为上限的、被积函数为$f(x,y)$的变上限积分。

也是不断地用洛必达法则。

> [!important] 变限积分求导
> 1. 若
> $$
> F(x)=\int_{\alpha(x)}^{\beta(x)}f(t,x)\,\mathrm dt,
> $$
> 则莱布尼茨公式为
> $$
> F'(x)=f(\beta(x),x)\beta'(x)-f(\alpha(x),x)\alpha'(x)
> +\int_{\alpha(x)}^{\beta(x)}\frac{\partial f}{\partial x}(t,x)\,\mathrm dt.
> $$
> 前两项来自积分上下限的变化，积分项来自被积函数中参数 $x$ 的变化。
>
> 2. 若：
> $$
> G(t)=\int_0^t\int_x^t f(x,y)\,\mathrm dy\,\mathrm dx.
> $$
> 积分区域是 $0\le x\le y\le t$，交换积分次序得
> $$
> G(t)=\int_0^t\int_0^y f(x,y)\,\mathrm dx\,\mathrm dy.
> $$
> 因而
> $$
> G'(t)=\int_0^t f(x,t)\,\mathrm dx,
> $$
> 再求一次导数：
> $$
> G''(t)=f(t,t)+\int_0^t\frac{\partial f}{\partial y}(x,t)\,\mathrm dx.
> $$


## $\infty-\infty$


若幂式的底数和指数同时变化，可插入一个中间项, 使得第一项只改变指数或底数，第二项只改变底数或指数, 然后再对每一项提公因式。例如：

$$
a_1^{b_1}-a_2^{b_2}
=\left(a_1^{b_1}-a_1^{b_2}\right)
+\left(a_1^{b_2}-a_2^{b_2}\right)=a_{1}^{b_{1}}(1-e^{(b_{2}-b_{1})\ln a_{1}})-a_{1}^{b_{2}}(1-e^{b_{2}\ln(1+(\frac{a_{1}-a_{2}}{a_{1}}))}).
$$


插入哪一个中间项并不固定。选择标准是：拆分后每一项只保留一种变化，并且能直接使用通分、共轭、中值定理或渐近展开。


> [!example] 底数与指数同时变化
> 设 $a>0$，求
>
> $$
> \lim_{x\to+\infty}
> \left[(x+a)^{1+\frac1x}-x^{1+\frac1{x+a}}\right].
> $$
>
> 令
>
> $$
> \alpha=1+\frac1x,
> \qquad
> \beta=1+\frac1{x+a}.
> $$
>
> 插入 $(x+a)^\beta$：
>
> $$
> \begin{align}
> (x+a)^\alpha-x^\beta &=\left[(x+a)^\alpha-(x+a)^\beta\right]+\left[(x+a)^\beta-x^\beta\right] \\
> &= (x+a)^{\alpha}(1-e^{(\beta-\alpha)\ln(x+a)})+(x+a)^{\beta}(1-e^{\beta \ln(1-\frac{a}{x+a})}) \\
> \end{align}
> $$
>
> 又$$\lim_{x\to+\infty}(x+a)^{\frac1x}=\exp\left\{ \lim_{ x \to \infty } \left(\frac{1}{x} \right)\ln(x+a) \right\}=0$$同理$$\lim_{ x \to \infty } (x+a)^{ \frac{1}{x+a} }=0$$
>
> 所以原式即为：
>
> $$
> \begin{align}
> &(x+a)\left( 1-e^{  -\frac{a}{x(x+a)} \ln(x+a)} \right)+(x+a)\left( 1-e^{\left( 1+ \frac{1}{x+a} \right) \left( -\frac{a}{x+a} \right)} \right) \\
> =&(x+a)(\frac{a}{x(x+a)} \ln(x+a))+(x+a)\left( \left( 1+ \frac{1}{x+a} \right) \left( \frac{a}{x+a} \right) \right) \\
> =& \frac{a}{x}\ln(x+a)+ a\left( 1+\frac{1}{x+a} \right) \\
> =& 0 + a \\
> =& a
> \end{align}
> $$

# 极限

$$
f(x)\to A
\iff
\forall\varepsilon>0,\ \exists\delta>0,
\ 0<|x-x_0|<\delta
\Rightarrow |f(x)-A|<\varepsilon.
$$

证明局部性质时，把 $\varepsilon$ 取成合适的固定值：

- 局部有界：取 $\varepsilon=1$；
- $A\ne0$ 时[[#保号性]]：取 $\varepsilon$为某个小于 $|A|$ 的正数即可；

# 保号性

**重要前提：$\displaystyle\lim_{ x \to x_{0} }f(x)=A$存在！**

先讨论$A\neq 0$时。保号性即存在$x_{0}$的去心邻域，若$x$在去心邻域里，则$f(x)$与$A$同号。

令 $\displaystyle\varepsilon= \frac{|A|}{3}$；则存在$\delta>0$, 使得$x \in (x_{0}-\delta, x_{0}+\delta)$，有$$0<|f(x)-A|< \frac{|A|}{3}$$
即$$\begin{cases}
0<\frac{2}{3}A<f(x)< \frac{4}{3}A,A>0; \\
\frac{4}{3}A<f(x)< \frac{2}{3}A<0,A<0
\end{cases}$$

得证。

所以我们还能更进一步地说：**保号性即存在$x_{0}$的去心邻域，若$x$在去心邻域里，在$A>0$时，则$f(x)$的值可以大于$[0, A)$的任意值；$A<0$时，则$f(x)$的值可以小于$(A, 0]$的任意值**

$A=0$ 时不能只由极限推出符号。还必须要有其他信息。

> [!example] 题目
> 已知 $f'(x)\le0$，且
> $$
> \lim_{x\to+\infty}f(x)=0.
> $$
> 证明：$f(x)\ge0$。

$f'(x)\le0$ 说明 $f$ 单调递减。对任意 $t>x$，$f(x)\ge f(t)$；令 $t\to+\infty$，得到 $f(x)\ge f(t)\geqslant{0}$。

# 夹逼

优先把目标压成绝对值：

$$
|f(x)|\le g(x),\qquad g(x)\to0
\Rightarrow f(x)\to0.
$$

振荡项用 $|\sin u|,|\cos u|\le1$；积分可用“区间长度 $\times$ 被积函数上界”估计。

# 连续

连续需同时满足：$f(x_0)$ 有定义、左右极限相等、极限等于 $f(x_0)$。

# 间断

| 单侧极限情况        | 间断类型 | 归类  |
| ------------- | ---- | --- |
| 有限且相等，但不等于函数值 | 可去   | 第一类 |
| 有限但不相等        | 跳跃   | 第一类 |
| 至少一侧趋于无穷      | 无穷   | 第二类 |
| 至少一侧振荡无极限     | 振荡   | 第二类 |

分段函数在分界点必须分别计算左极限、右极限和函数值。
