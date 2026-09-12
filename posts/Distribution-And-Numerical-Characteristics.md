---
title: Distribution And Numerical Characteristics
tags:
  - Distribution
categories:
  - Probability & Statistics
date: 2026-09-05 16:28:30
katex: true
---
# 一维随机变量及分布

## 一维离散型随机变量

离散型随机变量由分布律刻画：列出全部可能取值 $x_k$ 及其概率：

$$
P\{X=x_k\}=p_k,\qquad p_k\ge0,\qquad \sum_k p_k=1.
$$

### 常见一维离散型随机变量分布

以下记 $q=1-p$。

| 分布                | 含义与分布律                                                                                |
| ----------------- | ------------------------------------------------------------------------------------- |
| 二项分布 $B(n,p)$     | $n$ 次独立试验的成功次数；$P\{X=k\}=\binom nkp^kq^{n-k}$                                         |
| 几何分布 $G(p)$       | 首次成功所需试验次数；$P\{X=k\}=q^{k-1}p,\ k\ge1$                                                |
| 负二项分布 $NB(r,p)$   | 第 $r$ 次成功出现时的试验次数；$P\{X=k\}=\binom{k-1}{r-1}p^rq^{k-r},\ k\ge r$                      |
| 超几何分布 $H(N,M,n)$  | $N$ 个对象中有 $M$ 个目标，不放回抽取 $n$ 个；$P\{X=k\}=\dfrac{\binom Mk\binom{N-M}{n-k}}{\binom Nn}$ |
| 泊松分布 $P(\lambda)$ | 单位区间内稀有事件的发生次数；$P\{X=k\}=e^{-\lambda}\dfrac{\lambda^k}{k!}$                           |

> [!info] 二项分布的特殊性质
> 
> 二项分布的形式恰好是$(p+q)^{n}$的二项展开。即：
> $$\sum_{i=0}^{n}\binom nip^{i}q^{n-i}=(p+q)^{2}$$
> 对特定项求和则在这个公式的基础上进行构造。例如：
> >[!example] 奇数项和
> >$$\begin{align}
> S&=\sum_{i=0}^{n} \frac{1-(-1)^{i}}{2}\binom nip^{i}q^{n-i} \\
> &=\frac{1}{2}(p+q)^{n}-\frac{1}{2}(q-p)^{n}
> \end{align}$$

### 一维离散型随机变量分布的近似

| 原分布        | 近似分布         | 条件与参数                       |
| ---------- | ------------ | --------------------------- |
| $H(N,M,n)$ | $B(n,p)$     | 总体大、抽样比 $n/N$ 小，$p=M/N$     |
| $B(n,p)$   | $P(\lambda)$ | $n$ 大、$p$ 小，$\lambda=np$ 适中 |
| $B(n,p)$   | $N(np,npq)$  | $n$ 较大而 $p$ 不太大             |


## 一维连续型随机变量分布

### 分布函数与概率密度

分布函数定义：

$$
F_X(x)=P\{X\le x\}.
$$

函数 $F$ 是分布函数，当且仅当：

- $F$ 单调不减、右连续；
- $\displaystyle\lim_{x\to-\infty}F(x)=0$，$\displaystyle\lim_{x\to+\infty}F(x)=1$。

连续型随机变量若有概率密度 $f_X$，则满足

$$
f_X(x)\ge0,\qquad
\int_{-\infty}^{+\infty}f_X(x)\,\mathrm dx=1,\qquad
F_X(x)=\int_{-\infty}^{x}f_X(t)\,\mathrm dt.
$$

在 $F_X$ 可导处，$f_X(x)=F_X'(x)$。单点处密度值的改变不影响分布。

### 常见一维连续型随机变量分布

#### 均匀分布

设 $\displaystyle X\sim U(a,b)$，其中 $\displaystyle a<b$。密度与分布函数为：

$$
f_X(x)=
\begin{cases}
\frac{1}{b-a},&a<x<b,\\
0,&\text{其他},
\end{cases}
\qquad
F_X(x)=
\begin{cases}
0,&x\le a,\\
\frac{x-a}{b-a},&a<x<b,\\
1,&x\ge b.
\end{cases}
$$

#### 指数分布

设 $\displaystyle X\sim E(\lambda)$，其中 $\displaystyle \lambda>0$ 为率参数。密度与分布函数为：

$$
f_X(x)=
\begin{cases}
\lambda e^{-\lambda x},&x\ge0,\\
0,&x<0,
\end{cases}
\qquad
F_X(x)=
\begin{cases}
0,&x<0,\\
1-e^{-\lambda x},&x\ge0.
\end{cases}
$$

**无记忆性**：由 $\displaystyle P(X>x)=e^{-\lambda x}$（$\displaystyle x\ge0$），对 $\displaystyle s,t\ge0$ 有：

$$
P(X>s+t\mid X>s)
=\frac{P(X>s+t)}{P(X>s)}
=\frac{e^{-\lambda(s+t)}}{e^{-\lambda s}}
=e^{-\lambda t}
=P(X>t).
$$

#### 正态分布

$\displaystyle X\sim N(\mu,\sigma^2)$，$\displaystyle \mu\in\mathbb R$、$\displaystyle \sigma>0$，密度为：

$$
f_X(x)=\frac{1}{\sqrt{2\pi}\sigma}e^{-\frac{(x-\mu)^2}{2\sigma^2}},\qquad x\in\mathbb R.
$$

曲线关于 $\displaystyle \mu$ 对称；$\displaystyle \sigma$ 越大，曲线越宽、峰值越低。

性质：

以下 $\displaystyle \varphi,\Phi$ 分别为标准正态密度、分布函数。
1. 标准化与概率计算：

$$
Z=\frac{X-\mu}{\sigma}\sim N(0,1),\qquad F_X(x)=\Phi\left(\frac{x-\mu}{\sigma}\right).
$$

$$
P(a<X<b)=\Phi\left(\frac{b-\mu}{\sigma}\right)-\Phi\left(\frac{a-\mu}{\sigma}\right).
$$

2. 对称性：$\displaystyle \varphi(-z)=\varphi(z)$，$\displaystyle \Phi(0)=\frac12$
3. 积分对称性：

$$
\Phi(-z)=\int_{-\infty}^{-z}\varphi(t)\,\mathrm dt
=\int_z^{+\infty}\varphi(t)\,\mathrm dt=1-\Phi(z).
$$

所以当 $\displaystyle z\ge0$ 时，$\displaystyle P(\lvert Z\rvert<z)=2\Phi(z)-1$。

4. 线性变换：$\displaystyle a\ne0$ 时，$\displaystyle aX+b\sim N(a\mu+b,a^2\sigma^2)$。

## 一维随机变量$X$的函数的分布

### $X$ 为离散型

若 $X$ 为离散型，直接由分布律合并产生相同函数值的各项概率：

$$
P\{Y=y\}=\sum_{x:\,G(x)=y}P\{X=x\}.
$$

### $X$ 为连续型且$Y=G(X)$仍连续

若 $X$ 为连续型且$Y=G(X)$仍连续，从 $F_Y(y)=P\{G(X)\le y\}$ 出发，把事件改写成 $P\{X \in D_{y} \}$。

#### 概率积分变换

**若 $\displaystyle X$ 的分布函数 $\displaystyle F_X$ 连续**，则：

$$
U=F_X(X)\sim U(0,1).
$$


证明：令 $\displaystyle F_X(X)=U$，因此有

$$
P(U\le u)=P(F_X(X)\le u)
=P(X\le F^{-1}_{X}(u))=F_X(F_{X}^{-1}(u))=u.
$$

结合 $\displaystyle U$ 的取值范围为 $\displaystyle [0,1]$，这就是 $\displaystyle U(0,1)$ 的分布函数。


### $X$ 为连续型但$Y=G(X)$ 不连续

这里的“不连续”指 **$\displaystyle Y$ 的分布函数出现跳跃**：某个值 $\displaystyle c$ 满足 $\displaystyle P(Y=c)>0$。

此时 $\displaystyle Y$ 可能是离散型，也可能同时含有**概率点**和连续部分，成为混合型；不能只根据函数 $\displaystyle G$ 是否连续来判断。

>[!question]- 为什么连续型随机变量会变出概率点
>
> $\displaystyle P(X=x)=0$ 只说明每个单点的概率为零，不代表一整段取值的概率也为零。若 $\displaystyle G$ 把一段具有正概率的取值都映成同一个 $\displaystyle c$，这些概率就集中在 $\displaystyle Y=c$ 上：
>
> $$P(Y=c)=P(G(X)=c)=\int_{\{x:G(x)=c\}}f_X(x)\,\mathrm dx.$$
>
> 例如，将所有负数都变为零，原来 $\displaystyle X<0$ 这一整段的概率便成为 $\displaystyle Y=0$ 的概率。只要 $\displaystyle P(X<0)>0$，$\displaystyle Y$ 就不再是有普通概率密度的连续型随机变量。

从 $\displaystyle F_Y(y)=P(G(X)\le y)$ 出发，根据 $\displaystyle X$ 取值进行[[#$X$ 连续、$Y$ 离散时的 $Z=G(X,Y)$|全集分解]]，再分别计算分布函数，最后求和。

> [!example] 设 $\displaystyle X\sim N(0,1)$，$\displaystyle Y=\lfloor(X-1)^2\rfloor$，其中 $\displaystyle \lfloor u\rfloor$ 表示不超过 $\displaystyle u$ 的最大整数。求 $\displaystyle Y$ 的分布律。
>
> 全集分解：$\displaystyle Y$ 的可能取值为 $\displaystyle k=0,1,2,\ldots$，且：
>
> $$
> Y=k\iff k\le(X-1)^2<k+1
> \iff\sqrt{k}\le\lvert X-1\rvert<\sqrt{k+1}.
> $$
> 于是得到$X$的取值：
> $$
> (1-\sqrt{k+1},\,1-\sqrt{k}),\qquad
> (1+\sqrt{k},\,1+\sqrt{k+1}).
> $$
>
> 分别计算，再相加：记标准正态分布函数为 $\displaystyle \Phi$，得到完整分布律：
>
> $$
> \begin{aligned}
> P(Y=k)
> &=\Phi(1-\sqrt{k})-\Phi(1-\sqrt{k+1})\\
> &\quad+\Phi(1+\sqrt{k+1})-\Phi(1+\sqrt{k}),
> \qquad k=0,1,2,\ldots.
> \end{aligned}
> $$


> [!example] 设 $\displaystyle X\sim U(-1,2)$，
>
> $$
> Y=G(X)=
> \begin{cases}
> 0,&X\le0,\\
> X,&0<X<1,\\
> 1,&X\ge1.
> \end{cases}
> $$
>
> 所以：
>
> $$
> P(Y=0)=P(X\le0)=\frac13,\qquad
> P(Y=1)=P(X\ge1)=\frac13.
> $$
>
> $\displaystyle y<0$ 时事件不可能；$\displaystyle y\ge1$ 时事件必然发生。
> 
> 对于 $\displaystyle 0\le y<1$，$\displaystyle Y\le y$  $\displaystyle F_Y(y)=F_{X}(X<y)=\frac{y+1}{3}$。
> 
> 完整结果为：
>
> $$
> F_Y(y)=
> \begin{cases}
> 0,&y<0,\\
> \frac{y+1}{3},&0\le y<1,\\
> 1,&y\ge1.
> \end{cases}
> $$


# 二维随机变量及分布

## 二维离散型随机变量

设 $X$ 有 $m$ 个可能值 $x_1,\ldots,x_m$，$Y$ 有 $n$ 个可能值 $y_1,\ldots,y_n$。

联合分布律由任意 $m\times n$ 个格子组成：
$$
p_{ij}=P\{X=x_i,Y=y_j\},\qquad 1\le i\le m,\ 1\le j\le n.
$$
$$
p_{i\cdot}=\sum_{j=1}^{n}p_{ij},\qquad p_{\cdot j}=\sum_{i=1}^{m}p_{ij},\qquad \sum_{i=1}^{m}\sum_{j=1}^{n}p_{ij}=1.
$$

|              |      $Y=y_1$ |      $Y=y_2$ | $\cdots$ |       $Y=y_n$ | $P(X=x_{i})$ |
| ------------ | -----------: | -----------: | -------: | ------------: | -----------: |
| $X=x_1$      |     $p_{11}$ |     $p_{12}$ | $\cdots$ |      $p_{1n}$ | $p_{1\cdot}$ |
| $X=x_2$      |     $p_{21}$ |     $p_{22}$ | $\cdots$ |      $p_{2n}$ | $p_{2\cdot}$ |
| $\vdots$     |     $\vdots$ |     $\vdots$ | $\ddots$ |      $\vdots$ |     $\vdots$ |
| $X=x_m$      |     $p_{m1}$ |     $p_{m2}$ | $\cdots$ |      $p_{mn}$ | $p_{m\cdot}$ |
| $P(Y=y_{i})$ | $p_{\cdot1}$ | $p_{\cdot2}$ | $\cdots$ | $p_{\cdot n}$ |          $1$ |

条件分布为：

$$
P\{X=x_i\mid Y=y_j\}
=\frac{p_{ij}}{P\{Y=y_j\}}.
$$

$X,Y$ 独立，当且仅当每个格子都满足

$$
p_{ij}=P\{X=x_i\}P\{Y=y_j\}.
$$

## 二维连续型随机变量分布

联合分布函数：

$$
F_{X,Y}(x,y)=P\{X\le x,Y\le y\}.
$$
边缘分布函数为：
$$
F_X(x)=\lim_{y\to+\infty}F_{X,Y}(x,y),\qquad F_Y(y)=\lim_{x\to+\infty}F_{X,Y}(x,y).
$$

如果 $F_{X,Y}(x,y)$ 对 $x,y$ 连续，并且存在连续的二阶混合偏导数，则联合概率密度函数定义为：
$$
f_{X,Y}(x,y)=\frac{\partial^2F_{X,Y}(x,y)}{\partial x\,\partial y}.
$$
反过来，联合分布函数由联合密度在左下区域上的积分得到：
$$
F_{X,Y}(x,y)=\int_{-\infty}^{x}\int_{-\infty}^{y}f_{X,Y}(u,v)\,\mathrm dv\,\mathrm du.
$$
由分布函数的性质，密度满足：
$$
f_{X,Y}(x,y)\ge0,\qquad
\iint_{\mathbb R^2}f_{X,Y}(x,y)\,\mathrm dx\mathrm dy=1.
$$

使用$f_{X,Y}$求概率更为灵活，因为：

$$P\{x,y \in D\}=\iint_{D}f_{X,Y}(x,y)\,\mathrm dx\mathrm dy.$$

所以求例如 $\{Ax^{k}+By^{k}+C\leqslant {0}\}$ 这样的较复杂的事件的概率，用这种方式就可以很方便地计算出来。


边缘密度函数由联合密度积分得到：
$$
f_X(x)=\int_{-\infty}^{+\infty}f_{X,Y}(x,y)\,\mathrm dy,\qquad
f_Y(y)=\int_{-\infty}^{+\infty}f_{X,Y}(x,y)\,\mathrm dx.
$$
在有密度且分母为正时，条件密度为：
$$
f_{Y\mid X=x}(y)=\frac{f_{X,Y}(x,y)}{f_X(x)},\qquad
f_{X\mid Y=y}(x)=\frac{f_{X,Y}(x,y)}{f_Y(y)}.
$$

因此，条件分布函数由条件密度积分得到：
$$
F_{Y\mid X=x}(y)=\int_{-\infty}^{y}f_{Y\mid X=x}(v)\,\mathrm dv,
\qquad
F_{X\mid Y=y}(x)=\int_{-\infty}^{x}f_{X\mid Y=y}(u)\,\mathrm du.
$$

> [!example] 在线段上依次取两点，求第二点的分布
> 在线段 $\displaystyle [0,1]$ 上均匀选一点 $\displaystyle X$；给定 $\displaystyle X=x$ 后，再在 $\displaystyle [x,1]$ 上均匀选一点 $\displaystyle Y$。求 $\displaystyle Y$ 的密度与分布函数。
>
> 先固定第一点：$\displaystyle X\sim U(0,1)$，$\displaystyle Y\mid X=x\sim U(x,1)$。因此：
>
> $$
> f_{Y\mid X=x}(y)=
> \begin{cases}
> \frac{1}{1-x},&x<y<1,\\
> 0,&\text{其他},
> \end{cases}
> \qquad 0<x<1.
> $$
>
> 又因为$\displaystyle f_{X,Y}f(x,y)=f_{Y \mid X=x}(y)f_{X}(x), f_{Y}(y)=\int_{-\infty}^{+\infty} f_{X,Y}(x,y) \, \mathrm{d}x$ ：
>
> $$
> f_Y(y)=\int_{-\infty}^{+\infty} f_{Y\mid X=x}(y)f_X(x)\,\mathrm dx
> =\begin{cases} 
> \int_0^y\frac{1}{1-x}\,\mathrm dx,&0<y<1 \\
> 0,&others
> \end{cases}
> =\begin{cases}
> -\ln(1-y),&0<y<1 \\ \\
> 0, &others
> \end{cases}.
> $$
>
> 再积分得到 $F_Y(y)$。
> 
> 完整结果为：
>
> $$
> f_Y(y)=
> \begin{cases}
> -\ln(1-y),&0<y<1,\\
> 0,&\text{其他},
> \end{cases}
> \qquad
> F_Y(y)=
> \begin{cases}
> 0,&y\le0,\\
> y+(1-y)\ln(1-y),&0<y<1,\\
> 1,&y\ge1.
> \end{cases}
> $$

### 常见二维型随机变量分布

#### 二维均匀分布

若 $(X,Y)$ 在区域 $D$ 上服从二维均匀分布，$S(D)$为区域面积，则：

$$
f_{X,Y}(x,y)=
\begin{cases}
\dfrac1{S(D)},&(x,y)\in D,\\
0,&(x,y)\notin D.
\end{cases}
$$

如果区域是一个边均与坐标轴平行的矩形，那么易得此时$f_{X,Y}(x,y)=f_{X}(x)f_{Y}(y)$

#### 二维正态分布

记为

$$
(X,Y)\sim N(\mu_1,\mu_2;\sigma_1^2,\sigma_2^2;\rho),
$$

其中 $\mu_1,\mu_2$ 是均值，$\sigma_1^2,\sigma_2^2$ 是方差，$\rho$ 是线性相关系数。

性质：

1. $X\sim N(\mu_1,\sigma_1^2),\qquad Y\sim N(\mu_2,\sigma_2^2)$
2. 在二维正态分布中，独立与线性不相关等价
3. 任意线性组合仍服从正态分布：$k_1X+k_2Y\sim N\left(k_1\mu_1+k_2\mu_2,k_1^2\sigma_1^2+k_2^2\sigma_2^2+2k_1k_2\rho\sigma_1\sigma_2\right)$

## 二维随机变量$(X,Y)$的函数的分布

### $(X,Y)$ 均为离散型

离散型情形，把所有映到同一 $(u,v)$ 的点的联合概率相加。

### $X$ 连续、$Y$ 离散时的 $Z=G(X,Y)$

设 $X$ 连续，$Y$ 离散，研究 $Z=G(X,Y)$ 的分布。对事件 $\{G(X,Y)\le z\}$，按 $Y$ 的全部可能取值作全集分解：
$$
\{G(X,Y)\le z\}=\bigcup_j\bigl(\{G(X,y_j)\le z\}\cap\{Y=y_j\}\bigr),
$$
于是有：
$$
\begin{aligned}
F_Z(z)
&=P\{G(X,Y)\le z\}\\
&=\sum_jP\bigl(\{G(X,y_j)\le z\}\cap\{Y=y_j\}\bigr),
\end{aligned}
$$

如果某些项使 $G(X,y_j)$ 恒等于某个数 $c$，就在 $Z=c$ 处计入相应概率；其余项按连续变量函数的方法计算。

>[!example] 混合型分布
> X,Y独立。 $X$ 连续，$P\{Y=0\}=1/4$、$P\{Y=1\}=3/4$。求 $Z=YX$ 的分布。
> 
> 按上述方法对样本空间作全集分解：
> $$F_Z(z)=P\{X\le z,Y=1\}+P\{0\le z,Y=0\}=\frac34F_X(z)+\frac14P{\{z\ge0\}}.
> $$
>
> $P{\{z\ge0\}}$使 $F_Z$ 在 $0$ 处跳跃：当$z<0$时$,z\geqslant{0}$为$\varnothing$ 当$z\geqslant{0}$ 时, $z >0$ 为 $\Omega$ 。

### $Z=G(X,Y)$ 且$(X,Y)$ 均为连续型


$$
F_Z(z)=P\{G(X,Y)\le z\}
=\iint_{G(x,y)\le z}f_{X,Y}(x,y)\,\mathrm dx\mathrm dy.
$$

对 $z$ 求导即得概率密度函数。

求概率密度函数也可以使用[[#$(X,Y)$ 均为连续型且$U=G(X,Y),V=H(X,Y)$|以下]]的思路，令$U=Z=G(X,Y),V=Y$ ，最后对求出的 $f_{U,V}(u,v)$ 对 $v$ 积分求 $U$ 的边缘密度函数 $f_{U}(u)$ 即可。
### $(X,Y)$ 均为连续型且$U=G(X,Y),V=H(X,Y)$

已知$u=G(x,y),v=H(x,y)$，且反函数都存在，$\displaystyle\frac{ \partial (x,y) }{ \partial (u,v) }\neq {0}$。

则：

$$\begin{align}
F_{U,V}(u,v)
&=\iint_{D_{uv}} f_{X,Y}(x(u,v),y(u,v))
\left|\frac{\partial(x,y)}{\partial(u,v)}\right|\mathrm{d}u\mathrm{d}v \\
&=\iint_{D_{uv}} f_{X,Y}(x(u,v),y(u,v))
\left|\left|\begin{matrix}
\frac{ \partial u }{ \partial x } &\frac{ \partial u }{ \partial y } \\
\frac{ \partial v }{ \partial x } &\frac{ \partial v }{ \partial y } 
\end{matrix}\right|\right|^{-1}\mathrm{d}u\mathrm{d}v
\end{align}
$$

概率密度即：
$$
f_{X,Y}(x(u,v),y(u,v))
\left|\left|\begin{matrix}
\frac{ \partial u }{ \partial x } &\frac{ \partial u }{ \partial y } \\
\frac{ \partial v }{ \partial x } &\frac{ \partial v }{ \partial y } 
\end{matrix}\right|\right|^{-1}
$$


## 随机变量独立同分布

### 极值统计量的分布

设 $X_1,\ldots,X_n$ 独立同分布，公共分布函数为 $F$，记
$$
M_n=\max(X_1,\ldots,X_n),\qquad m_n=\min(X_1,\ldots,X_n).
$$

对最大值，把“最大值不超过 $x$”改写为所有样本都不超过 $x$：
$$
\begin{aligned}
F_{M_n}(x)
&=P(M_n\le x)\\
&=P(X_1\le x,\ldots,X_n\le x)\\
&=\prod_{i=1}^nP(X_i\le x)\quad(\text{独立})\\
&=\prod_{i=1}^nF(x)=F(x)^n.
\end{aligned}
$$

对最小值，处理“最小值大于 $x$ 取反”更简单，它等价于所有样本都大于 $x$ 取反：
$$
\begin{aligned}
F_{m_n}(x)
&=P(m_n\le x)\\
&=1-P(m_n>x)\\
&=1-P(X_1>x,\ldots,X_n>x)\\
&=1-\prod_{i=1}^nP(X_i>x)\\
&=1-\prod_{i=1}^n[1-F(x)]\\
&=1-[1-F(x)]^n.
\end{aligned}
$$

若 $F$ 可导且密度为 $f$，对上面两个分布函数分别求导：
$$
f_{M_n}(x)=\frac{\mathrm d}{\mathrm dx}F(x)^n=nF(x)^{n-1}f(x).
$$
$$
f_{m_n}(x)=\frac{\mathrm d}{\mathrm dx}\{1-[1-F(x)]^n\}=n[1-F(x)]^{n-1}f(x).
$$

> [!example] 不是样本最小值的概率
> 设随机变量 $X_1,X_2,\ldots,X_n$ 独立同分布，且有相同的概率密度。求：
>
> $$
> P\left\{X_n>\min(X_1,X_2,\ldots,X_{n-1})\right\}.
> $$
>
> 因为随机变量连续，同一最小值出现并列的概率为 $0$。$X_n$ 成为 $n$ 个样本中最小值的概率与每个 $X_i$ 相同，故：
>
> $$
> P\left\{X_n=\min(X_1,X_2,\ldots,X_n)\right\}=\frac1n.
> $$
>
> 所求事件是上述事件的补事件，所以：
>
> $$
> P\left\{X_n>\min(X_1,X_2,\ldots,X_{n-1})\right\}
> =1-\frac1n=\frac{n-1}{n}.
> $$

### 一些在独立同分布下可加的分布

- 若 $X\sim B(m,p)$、$Y\sim B(n,p)$ 且独立，则 $X+Y\sim B(m+n,p)$。
- 若 $X\sim P(\lambda_1)$、$Y\sim P(\lambda_2)$ 且独立，则 $X+Y\sim P(\lambda_1+\lambda_2)$。
- 若 $X\sim N(\mu_1,\sigma_1^2)$、$Y\sim N(\mu_2,\sigma_2^2)$ 且独立，则 $X+Y\sim N(\mu_1+\mu_2,\sigma_1^2+\sigma_2^2)$；若 $U\sim\chi^2(m)$、$V\sim\chi^2(n)$ 且独立，则 $U+V\sim\chi^2(m+n)$。

# 分布的数字特征
## 期望

> 前提条件：积分/级数和存在。

定义期望$\operatorname{E}(X)=\displaystyle\sum_{k=0}^{\infty}x_kP\{X=x_k\}$或$\operatorname{E}(X)=\int_{-\infty}^{+\infty}xf_X(x)\,\mathrm dx.$ 显然期望的求解涉及[[../Infinite-Series#级数求和|无穷级数的求和]]或[[../../posts/Improper-Integral#|反常积分]]（但实际上$f_{X}(x)$非0区域可能有限，此时退化为普通的定积分）。

性质：
1.  **在知道$X$的分布的情况下，若$Y=G(X)$ ，则求 $EY$ 不必先求 $Y$ 的分布。** $\operatorname{E}[G(X)]=\displaystyle\sum_{k=0}^{\infty}G(x_k)P\{X=x_k\}$或$\operatorname{E}[G(X)]=\int_{-\infty}^{+\infty}G(x)f_X(x)\,\mathrm dx.$
2. $\operatorname{E}A=A$ $A$为常数。
3. 期望具有线性性，不要求随机变量独立：$\operatorname{E}\left( \sum_{i=1}^{n}a_{i}X_{i} +a_{0}\right)=\sum_{i=1}^{n}a_{i}E(X_{i})+a_{0}$
> [!example]- 最大值与最小值的距离的期望
> 条件[[#极值统计量的分布|同上]]，则最大值与最小值的距离的期望即:
> $$E(M-m)=EM-Em$$
4. 若$X,Y$独立，$\operatorname{E}(XY)=\operatorname{E}X\operatorname{E}Y$

## 条件期望 

若 $Y$ 取离散值 $y_j$，则对每个 $y_j$：
$$
E(X\mid Y=y_j)=\sum_i x_iP(X=x_i\mid Y=y_j).
$$
若 $X,Y$ 有联合密度，则：
$$
E(X\mid Y=y)=\int_{-\infty}^{+\infty}x f_{X\mid Y=y}(x)\,\mathrm dx,
\qquad
f_{X\mid Y=y}(x)=\frac{f_{X,Y}(x,y)}{f_Y(y)}.
$$

条件期望 $E(X\mid Y=y)$ 是关于 $y$ 的函数。

先求出这个函数，再对 $Y$ 求期望，就得到全期望公式：
$$
E(X)=\sum_jE(X\mid Y=y_j)P(Y=y_j)=E(E(X\mid Y)),
$$
连续型时改为：
$$
E(X)=\int_{-\infty}^{+\infty}E(X\mid Y=y)f_Y(y)\,\mathrm dy=E(E(X\mid Y)).
$$



## 方差与标准差

方差$\operatorname{D}(X)=\operatorname{E}\bigl[(X-\operatorname{E}(X))^2\bigr]=\operatorname{E}(X^2)-[\operatorname{E}(X)]^2,$

标准差$\sigma_X=\sqrt{\operatorname{D}(X)}.$


方差性质：
1. $\operatorname{D}X\geqslant{0}$ 当仅当$X$几乎处处为常数时即$P\{X=A\}=1$取等，$A$为常数 
>[!info]- 为什么是“几乎处处”而不是严格取等
>因为根据几何概型，一条线上测度为 0 的点集不满足是不改变 $P\{X=A\}=1$ 的。
2. $\operatorname{D}(aX+b)=a^2\operatorname{D}(X),$
3. $\operatorname{D}\left( \sum_{i=1}^{n}a_{i}X_{i} +a_{0}\right)=\sum_{i=1}^{n}a_{i}^{2}\operatorname{D}(X_{i})+2\sum_{1\leqslant i<j \leqslant n} a_{i}a_{j}\operatorname{Cov}(X_{i},X_{j}).$
4. 若$X,Y$独立，$\operatorname{D}(XY)=\operatorname{D}X\operatorname{D}Y+\operatorname{D}X\operatorname{E}^{2}Y+\operatorname{D}Y\operatorname{E}^{2}X$


## 一维分布的期望与方差总表

记 $q=1-p$ 。

| 分布                |      $E(X)$ |        $D(X)$ |
| ----------------- | ----------: | ------------: |
| $B(n,p)$          |        $np$ |         $npq$ |
| $G(p)$            |       $1/p$ |       $q/p^2$ |
| $NB(r,p)$         |       $r/p$ |      $rq/p^2$ |
| $P(\lambda)$      |   $\lambda$ |     $\lambda$ |
| $U(a,b)$          |   $(a+b)/2$ |  $(b-a)^2/12$ |
| $E(\lambda)$      | $1/\lambda$ | $1/\lambda^2$ |
| $N(\mu,\sigma^2)$ |       $\mu$ |    $\sigma^2$ |
## 协方差与线性相关系数

$$
\operatorname{Cov}(X,Y)=E[(X-E(X))(Y-E(Y))]=E(XY)-E(X)E(Y).
$$

$$
\rho_{XY}=\frac{\operatorname{Cov}(X,Y)}{\sqrt{\operatorname{D}(X)\operatorname{D}(Y)}}.
$$

性质：

1. 协方差对每个变量都具有可加性：$\operatorname{Cov}(X_1+X_2,X_3)=\operatorname{Cov}(X_1,X_3)+\operatorname{Cov}(X_2,X_3)$。
2. 协方差对常数倍具有齐次性：$\operatorname{Cov}(aX,bY)=ab\operatorname{Cov}(X,Y)$。
3. 协方差具有对称性，且 $\operatorname{Cov}(X,Y)=\operatorname{Cov}(Y,X)$、$\operatorname{Cov}(X,X)=\operatorname{D}(X)$。
4. $-1\le\rho_{XY}\le1$。当 $\rho_{XY}=1$ 时，存在 $a>0,b$ 使 $Y=aX+b$ 几乎处处成立，表示完全正线性关系；当 $\rho_{XY}=-1$ 时，存在 $a<0,b$ 使 $Y=aX+b$ 几乎处处成立，表示完全负线性关系。

## 独立与线性相关

1. 若 $F_{X,Y}=F_XF_Y$（离散型为 $p_{ij}=p_{i\cdot}p_{\cdot j}$；有密度时也可以为 $f_{X,Y}=f_Xf_Y$）此时称 $X,Y$ 独立否则 $X,Y$ 不独立。
2. 分别对两个独立随机变量$X,Y$做函数变换$f(X),g(Y)$，$f(X),g(Y)$仍然保持独立。
3. 若$\rho_{XY}$为 0 则称$X,Y$ 线性无关否则$X,Y$线性相关。显然 $X,Y$ 线性无关$\iff\operatorname{Cov}(X,Y)=0.$
4. **独立一定线性无关，但线性无关不一定独立**。
5. 在二维正态分布中，独立就是线性无关。
