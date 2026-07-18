---
title: Function Limits and Continuity
tags:
  - Calculus
  - Limit
  - Continuity
categories:
  - Mathematics
date: 2026-07-18 13:59:09
katex: true
---

# 等价无穷小

| 结构 | 优先处理 |
|---|---|
| $0/0$，无明显相消 | 等价无穷小、因式分解、洛必达 |
| $0/0$，主项相消 | 泰勒展开到首个非零项 |
| 含变上限积分 | 积分中值或洛必达去积分号 |
| $\infty-\infty$ | 通分、有理化、提取主导项 |
| 振荡因子乘衰减因子 | 取绝对值后夹逼 |
| $1^\infty,0^0,\infty^0$ | 先取对数 |

## $0/0$：比较最低非零次数

写成

$$
P(x)=a_m(x-x_0)^m+o((x-x_0)^m),
$$

$$
Q(x)=b_n(x-x_0)^n+o((x-x_0)^n).
$$

- $m>n$：极限为 $0$；
- $m=n$：极限为 $a_m/b_n$；
- $m<n$：通常发散，需判断左右符号。

“次数对齐”不是让分子、分母形式相同，而是两边都展开到首个不相消的项。

常用等价无穷小：

$$
\sin x\sim x,\quad e^x-1\sim x,\quad
\ln(1+x)\sim x,\quad 1-\cos x\sim\frac{x^2}{2}.
$$

等价替换只直接用于乘积和商。遇到 $\sin x-x$、$e^x-1-x$ 等加减相消，改用泰勒展开。

## 含积分号

若被积函数连续，变上限积分可求导：

$$
\left(\int_a^x f(t)\,\mathrm dt\right)'=f(x).
$$

形成 $0/0$ 型时，可用洛必达去积分号。若分母就是区间长度，直接记住

$$
\lim_{x\to a}\frac1{x-a}\int_a^x f(t)\,\mathrm dt=f(a).
$$

先确认是未定式；“含积分”本身不是使用洛必达的理由。

## $\infty-\infty$

目标是暴露大项相消后的下一阶项：

- 分式通分；
- 根式乘共轭式；
- 同阶项提取主导量；
- 指数、幂式先取对数或作渐近展开。

例如

$$
\sqrt{x^2+x}-x
=\frac{x}{\sqrt{x^2+x}+x}
\to\frac12.
$$

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
- $A\ne0$ 时保号：取 $\varepsilon=|A|/2$；
- 证明分母远离零：同样取 $\varepsilon=|A|/2$。

$A=0$ 时不能只由极限推出符号。

> [!example] 题目
> 已知 $f'(x)\le0$，且
> $$
> \lim_{x\to+\infty}f(x)=0.
> $$
> 证明：$f(x)\ge0$。

$f'(x)\le0$ 说明 $f$ 单调递减。对任意 $t>x$，$f(x)\ge f(t)$；令 $t\to+\infty$，得到 $f(x)\ge f(t)\geqslant_{0}$。

# 夹逼

优先把目标压成绝对值：

$$
|f(x)|\le g(x),\qquad g(x)\to0
\Rightarrow f(x)\to0.
$$

振荡项用 $|\sin u|,|\cos u|\le1$；积分可用“区间长度 $\times$ 被积函数上界”估计。

# 连续与间断

连续需同时满足：$f(x_0)$ 有定义、左右极限相等、极限等于 $f(x_0)$。

| 单侧极限情况        | 间断类型 | 归类  |
| ------------- | ---- | --- |
| 有限且相等，但不等于函数值 | 可去   | 第一类 |
| 有限但不相等        | 跳跃   | 第一类 |
| 至少一侧趋于无穷      | 无穷   | 第二类 |
| 至少一侧振荡无极限     | 振荡   | 第二类 |

分段函数在分界点必须分别计算左极限、右极限和函数值。
