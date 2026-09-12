---
title: Linear Space
tags:
  - LinearSpace
categories:
  - Linear Algebra
date: 2026-09-06 19:53:00
katex: true
---
# 基与坐标

## 基

在线性空间中，向量相加、数乘后仍在该空间内，并满足通常的线性运算规律。这里讨论有限维实线性空间。

一组基 $\mathbf{e}_1,\ldots,\mathbf{e}_n$ 既线性无关，又能表示空间中的每个向量。因此每个向量 $\mathbf{v}$ 都有唯一表示：

$$
\mathbf{v}=x_1\mathbf{e}_1+\cdots+x_n\mathbf{e}_n
=\mathbf{E}\mathbf{x},\qquad
\mathbf{E}=(\mathbf{e}_1,\ldots,\mathbf{e}_n).
$$

$\mathbf{x}=(x_1,\ldots,x_n)^\top$ 是坐标向量，$x_i$ 是标量。$\mathbf{v}$ 是向量，$\mathbf{x}$ 是它在基 $\mathbf{E}$ 下的坐标。换基不改变向量，仅改变其坐标。

$\mathbf{E}$ 表示一组有序的基；在 $\mathbb{R}^n$ 中，它也可以直接看成基向量按列组成的可逆矩阵。

## 过渡矩阵

设旧基为 $\mathbf{E}$，新基为 $\mathbf{F}=(\mathbf{f}_1,\ldots,\mathbf{f}_n)$。将每个新基向量用旧基表示：

$$
\mathbf{f}_j=\sum_{i=1}^n p_{ij}\mathbf{e}_i.
$$

以这些坐标作为列，得到过渡矩阵 $\mathbf{P}=(p_{ij})$，即：

$$
\mathbf{F}=\mathbf{E}\mathbf{P}.
$$

$\mathbf{P}$ 的第 $j$ 列是新基向量 $\mathbf{f}_j$ 在旧基下的坐标。两组都是基，所以 $\mathbf{P}$ 可逆，反向过渡矩阵为 $\mathbf{P}^{-1}$。在 $\mathbb{R}^n$ 中，若两组基已按列写成矩阵，可直接求 $\mathbf{P}=\mathbf{E}^{-1}\mathbf{F}$。

## 坐标变换

同一个向量在两组基下分别写成 $\mathbf{v}=\mathbf{E}\mathbf{x}=\mathbf{F}\mathbf{y}$。由 $\mathbf{F}=\mathbf{E}\mathbf{P}$，得到：

$$
\mathbf{x}=\mathbf{P}\mathbf{y},\qquad
\mathbf{y}=\mathbf{P}^{-1}\mathbf{x}.
$$

# 线性变换

## 定义

设 $V$ 为实线性空间。映射 $T:V\to V$ 称为线性变换，如果对任意 $\mathbf{u},\mathbf{v}\in V$ 和任意实数 $k$，均满足：

$$
T(\mathbf{u}+\mathbf{v})=T(\mathbf{u})+T(\mathbf{v}),\qquad
T(k\mathbf{u})=kT(\mathbf{u}).
$$

即 $T$ 保持向量的加法与数乘运算。

若存在变换$K$，$K(T(\mathbf{u}))=T(K(\mathbf{u}))=\mathbf{u}$，则称$T$可逆，且将$K$称为$T$的逆变换，记为$T^{-1}$。

可以证明一个线性变换$T$最多有一个逆变换$T^{-1}$。

## 性质

1. **零向量。** 由数乘性质取 $k=0$，得到 $T(\mathbf{0})=\mathbf{0}$。
2. **线性组合。** 对任意向量 $\mathbf{v}_1,\ldots,\mathbf{v}_s$ 和实数 $c_1,\ldots,c_s$，有 $T(\sum_{i=1}^s c_i\mathbf{v}_i)=\sum_{i=1}^s c_iT(\mathbf{v}_i)$。
3. **线性相关性。** 若 $\sum_i c_i\mathbf{v}_i=\mathbf{0}$，且系数不全为零，则 $\sum_i c_iT(\mathbf{v}_i)=\mathbf{0}$。因此，线性相关的向量组经线性变换后仍线性相关。
4. **线性无关性。** 线性无关的向量组经线性变换后不一定线性无关，例如零变换将所有向量映为零向量。若 $T$ 可逆，则由 $\sum_i c_iT(\mathbf{v}_i)=T(\sum_i c_i\mathbf{v}_i)=\mathbf{0}$ 可得 $\sum_i c_i\mathbf{v}_i=\mathbf{0}$，故可逆线性变换保持线性无关性。
5. 

## 矩阵表示

设 $\mathbf{E}=(\mathbf{e}_1,\ldots,\mathbf{e}_n)$ 为 $V$ 的一组基。将各基向量的像在这组基下展开：

$$
T(\mathbf{e}_j)=\sum_{i=1}^n a_{ij}\mathbf{e}_i,
\qquad j=1,\ldots,n.
$$

以 $T(\mathbf{e}_j)$ 的坐标为第 $j$ 列组成的矩阵 $\mathbf{A}=(a_{ij})$，称为 $T$ 在基 $\mathbf{E}$ 下的矩阵。对任意 $\mathbf{v}=\mathbf{E}\mathbf{x}$，由线性性质得：

$$
T(\mathbf{v})
=\sum_{j=1}^n x_jT(\mathbf{e}_j)
=\mathbf{E}\mathbf{A}\mathbf{x}.
$$

因此，$T(\mathbf{v})$ 在基 $\mathbf{E}$ 下的坐标为 $\mathbf{A}\mathbf{x}$。**一组基向量的像唯一确定线性变换；选定基后，线性变换与矩阵一一对应。**

## 相似

**相似的本质：同一线性变换在不同基下的不同矩阵表示。**

设 $T$ 在旧基 $\mathbf{E}$ 下表示为 $\mathbf{A}$，在新基 $\mathbf{F}=\mathbf{E}\mathbf{P}$ 下表示为 $\mathbf{B}$，则：

$$
\mathbf{B}=\mathbf{P}^{-1}\mathbf{A}\mathbf{P}.
$$



1. **自反性、对称性、传递性。** 相应的过渡矩阵分别为单位矩阵、原过渡矩阵的逆矩阵及两次过渡矩阵的乘积。
2. **幂、多项式与逆矩阵。** 相似变换保持矩阵的加法、数乘和乘法，故 $\mathbf{B}^k=\mathbf{P}^{-1}\mathbf{A}^k\mathbf{P}$；对任意多项式 $p$，有 $p(\mathbf{B})=\mathbf{P}^{-1}p(\mathbf{A})\mathbf{P}$；可逆时 $\mathbf{B}^{-1}=\mathbf{P}^{-1}\mathbf{A}^{-1}\mathbf{P}$。

## 特征值与特征向量

若 $\mathbf{u}\ne\mathbf{0}$ 且 $T(\mathbf{u})=\lambda\mathbf{u}$，则 $\lambda$ 为特征值，$\mathbf{u}$ 为对应的特征向量。$T$ 在 $\mathbf{u}$ 张成的一维子空间上的作用为数乘 $\lambda$。

在选定基下，记 $\mathbf{u}$ 的坐标为 $\mathbf{v}$，则：

$$
\mathbf{A}\mathbf{v}=\lambda\mathbf{v},\qquad
(\mathbf{A}-\lambda\mathbf{I})\mathbf{v}=\mathbf{0}.
$$

同一特征值对应的特征向量连同零向量，组成一个线性空间，其维数为 $n-r(\mathbf{A}-\lambda\mathbf{I})$。该空间内的每个向量均满足 $T(\mathbf{u})=\lambda\mathbf{u}$。

不同特征值对应的特征向量线性无关。对 $\mathbf{A}\mathbf{v}_i=\lambda_i\mathbf{v}_i$，有 $(\mathbf{A}-\lambda_j\mathbf{I})\mathbf{v}_i=(\lambda_i-\lambda_j)\mathbf{v}_i$：$i=j$ 时为零向量，$\lambda_i\ne\lambda_j$ 时为 $\mathbf{v}_i$ 的非零倍数。

具体地，设 $\lambda_1,\ldots,\lambda_s$ 互异，$\sum_{i=1}^s k_i\mathbf{v}_i=\mathbf{0}$。为确定 $k_i$，依次左乘所有 $j\ne i$ 对应的 $\mathbf{A}-\lambda_j\mathbf{I}$，其余各项都被消去，得到：

$$
k_i\prod_{j\ne i}(\lambda_i-\lambda_j)\mathbf{v}_i=\mathbf{0}.
$$

各差值非零，且 $\mathbf{v}_i\ne\mathbf{0}$，故 $k_i=0$。对每个 $i$ 都成立，所以这些向量无关。

对于 $k$ 重特征值，最多对应 $k$ 个线性无关的特征向量。令 $r$ 为 $(\mathbf{A}-\lambda\mathbf{I})\mathbf{v}=\mathbf{0}$ 的解空间的维数，于是设特征向量为 $\mathbf{v}_1,\ldots,\mathbf{v}_r$，按列写成 $\mathbf{M}$，则 $\mathbf{A}\mathbf{M}=\lambda\mathbf{M}$。

将空间里的另外$n-r$个线性无关且与特征向量线性无关的向量$\{\mathbf{\alpha_{1},\dots,\alpha_{n-r}}\}$与$\mathbf{M}$拼接得可逆矩阵 $\mathbf{P=[M,\alpha_{1},\dots,\alpha_{n-r}]}$。故 $\mathbf{A}\mathbf{P}$ 的前 $r$ 列为 $\lambda\mathbf{M}$。

又因 $\mathbf{M}$ 是 $\mathbf{P}$ 的前 $r$ 列，而 $\mathbf{P}^{-1}\mathbf{P}=\mathbf{I}$ 的前 $r$ 列是 $\begin{pmatrix}\mathbf{I}_r\\\mathbf{0}\end{pmatrix}$，故 $\mathbf{P}^{-1}\mathbf{M}=\begin{pmatrix}\mathbf{I}_r\\\mathbf{0}\end{pmatrix}$。于是 $\mathbf{P}^{-1}\mathbf{A}\mathbf{P}$ 的前 $r$ 列为 $\mathbf{P}^{-1}(\lambda\mathbf{M})=\begin{pmatrix}\lambda\mathbf{I}_r\\\mathbf{0}\end{pmatrix}$，其余列记为 $\begin{pmatrix}\mathbf{C}\\\mathbf{D}\end{pmatrix}$，所以：
$$
\mathbf{P}^{-1}\mathbf{A}\mathbf{P}=\begin{pmatrix}\lambda\mathbf{I}_r & \mathbf{C}\\ \mathbf{0} & \mathbf{D}\end{pmatrix}.
$$
因为 $\mathbf{P}^{-1}\mathbf{A}\mathbf{P}$ 与 $\mathbf{A}$ 相似，所以$\mathbf{P}^{-1}\mathbf{A}\mathbf{P}-\lambda_{x}\mathbf{I}$与$\mathbf{A}-\lambda_{x}\mathbf{I}$相似，有：
$$
\det(\mathbf{A}-\lambda_{x}\mathbf{I})=\det(\mathbf{P}^{-1}\mathbf{A}\mathbf{P}-\lambda_{x}\mathbf{I})=(\lambda-\lambda_{x})^r\det(\mathbf{D}-\lambda_{x}\mathbf{I}).
$$
令 $\det(\mathbf{A}-\lambda_{x}\mathbf{I})=0$ ，则 $\lambda$ 至少是 $r$ 重根，所以重数 $k\ge r$。

这个结论常用来反推：若 $(\mathbf{A}-\lambda\mathbf{I})\mathbf{v}=\mathbf{0}$ 的解空间有 $r$ 维，则 $\lambda$ 的重数至少为 $r$。
## 相似不变量

同一变换的性质不会因换基改变。若 $\mathbf{B}=\mathbf{P}^{-1}\mathbf{A}\mathbf{P}$，则下列六项必须相同：

1. **秩。** 线性变换的秩等于其值域的维数，不依赖基的选择，故 $r(\mathbf{A})=r(\mathbf{B})$。
2. **特征值。** 关系 $T(\mathbf{u})=\lambda\mathbf{u}$ 不依赖基的选择。若 $\mathbf{A}\mathbf{v}=\lambda\mathbf{v}$，则 $\mathbf{B}(\mathbf{P}^{-1}\mathbf{v})=\lambda(\mathbf{P}^{-1}\mathbf{v})$。特征值及其在特征多项式中的重数均不变。
3. **各特征值对应的线性无关特征向量的最大个数。** 换基只改变坐标，不改变这些向量组成的空间，故对每个 $\lambda$，都有 $n-r(\mathbf{A}-\lambda\mathbf{I})=n-r(\mathbf{B}-\lambda\mathbf{I})$。
4. **行列式。** 同一变换对有向体积的缩放倍数不变，且 $\det\mathbf{B}=\det(\mathbf{P}^{-1})\det\mathbf{A}\det\mathbf{P}=\det\mathbf{A}$。
5. **迹。** 主对角线元素之和等于全部特征值之和（计重数），故 $\operatorname{tr}\mathbf{A}=\operatorname{tr}\mathbf{B}$。单个对角元可变，总和不变。
6. **各阶主子式之和。** 每一阶的主子式之和分别不变，因为它们是特征多项式的系数，至多相差约定的符号。特征多项式不变可直接由下式看出：

$$
\det(t\mathbf{I}-\mathbf{B})
=\det\bigl(\mathbf{P}^{-1}(t\mathbf{I}-\mathbf{A})\mathbf{P}\bigr)
=\det(t\mathbf{I}-\mathbf{A}).
$$

这些都是必要条件，其中有些相互包含。有一项不同即可排除相似；一般情形下，全部相同仍不足以保证相似。

证明相似可以构造可逆 $\mathbf{P}$ 使 $\mathbf{A}\mathbf{P}=\mathbf{P}\mathbf{B}$，或证明两者能对角化为同一个对角矩阵。

## 对角化

**对角化是以线性变换的一组特征向量为基，使其矩阵成为对角矩阵。** 因而 $\mathbf{A}$ 可对角化，当且仅当存在 $n$ 个线性无关的特征向量。

将这些特征向量的坐标按列排成 $\mathbf{P}=(\mathbf{v}_1,\ldots,\mathbf{v}_n)$，则：

$$
\mathbf{A}\mathbf{P}
=\mathbf{P}\operatorname{diag}(\lambda_1,\ldots,\lambda_n),
\qquad
\mathbf{P}^{-1}\mathbf{A}\mathbf{P}
=\operatorname{diag}(\lambda_1,\ldots,\lambda_n).
$$

## 实对称矩阵

设 $\mathbf{A}$ 为 $n$ 阶实对称矩阵，即 $\mathbf{A}^\top=\mathbf{A}$。它的特征值均为实数，且每个 $k$ 重特征值都有 $k$ 个线性无关的实特征向量，因而共有 $n$ 个线性无关的实特征向量。

**不同特征值对应的特征向量相互正交。** 若 $\mathbf{A}\mathbf{u}=\lambda\mathbf{u}$、$\mathbf{A}\mathbf{v}=\mu\mathbf{v}$，利用对称性可得：

$$
\lambda\mathbf{u}^\top\mathbf{v}
=(\mathbf{A}\mathbf{u})^\top\mathbf{v}
=\mathbf{u}^\top\mathbf{A}\mathbf{v}
=\mu\mathbf{u}^\top\mathbf{v}.
$$

当 $\lambda\ne\mu$ 时，$\mathbf{u}^\top\mathbf{v}=0$。同一特征值对应的特征向量不一定相互正交，但可将其中一组线性无关的特征向量正交化、单位化；所得向量仍对应同一特征值，因为它们是原特征向量的非零线性组合。

因此，**实对称矩阵有 $n$ 个两两正交的单位特征向量，可作为一组基。** 以它们为列组成正交矩阵 $\mathbf{Q}$，便有：

$$
\mathbf{Q}^{-1}=\mathbf{Q}^\top,\qquad
\mathbf{Q}^\top\mathbf{A}\mathbf{Q}
=\operatorname{diag}(\lambda_1,\ldots,\lambda_n).
$$

# 二次型

二次型$f(\mathbf{x})=\mathbf{x^{\top}Ax}$是只含二次项的齐次多项式。令$f(\mathbf{x})=C$，$C$ 为不为 0 的常数，可以表示空间二次图形。

## 矩阵表示

线性变换将向量映为向量，二次型则给向量赋值。记空间中向量 $\mathbf{v}$ 的二次型值为 $f(\mathbf{v})$。在基 $\mathbf{E}$ 下，$\mathbf{v}=\mathbf{E}\mathbf{x}$，可写成：

$$
f(\mathbf{v})=\mathbf{x}^\top\mathbf{A}\mathbf{x}.
$$

二次型的矩阵取为实对称矩阵：$a_{ii}$ 为 $x_i^2$ 的系数；$i\ne j$ 时，$a_{ij}=a_{ji}$ 为 $x_ix_j$ 系数的一半。例如 $x_1^2+2x_1x_2+3x_2^2$ 对应 $\begin{pmatrix}1&1\\1&3\end{pmatrix}$。

对任意实方阵 $\mathbf{A}$，有：

$$
\mathbf{x}^\top\mathbf{A}\mathbf{x}
=\mathbf{x}^\top\frac{\mathbf{A}+\mathbf{A}^\top}{2}\mathbf{x}.
$$

因此，$\mathbf{A}$ 与其对称部分对应同一二次型。

## 合同

**从几何角度，二次型的合同关注图形在可逆线性变换下的等价关系**。允许改变长度和角度。例如，令 $u=x,\ v=2y$，椭圆 $x^2+4y^2=1$ 化为圆 $u^2+v^2=1$，对应的二次型属于同一合同类；但双曲线 $x^2-y^2=1$ 对应的二次型则属于另一类。

本质即相同的二次图像在不同基向量下的表示。

若 $\mathbf{v}=\mathbf{E}\mathbf{x}=\mathbf{F}\mathbf{y}$，且 $\mathbf{F}=\mathbf{E}\mathbf{P}$，则 $\mathbf{x}=\mathbf{P}\mathbf{y}$。由[[#过渡矩阵]]可知，$\mathbf{P}$ 可逆。代入两种坐标表示：

$$
f(\mathbf{v})
=\mathbf{x}^\top\mathbf{A}\mathbf{x}
=\mathbf{y}^\top(\mathbf{P}^\top\mathbf{A}\mathbf{P})\mathbf{y}.
$$

因此新基下的矩阵为 $\mathbf{B}=\mathbf{P}^\top\mathbf{A}\mathbf{P}$。

若两组基均为正交单位基，则 $\mathbf{P}$ 为正交矩阵。

> [!info]- 施密特正交化
> 设 $\mathbf{\alpha}_1,\ldots,\mathbf{\alpha}_s$ 为线性无关的实向量。依次减去各向量在已得正交向量上的投影：
>
> $$
> \begin{aligned}
> \mathbf{\beta}_1&=\mathbf{\alpha}_1,\\
> \mathbf{\beta}_k&=\mathbf{\alpha}_k-
> \sum_{i=1}^{k-1}
> \frac{\mathbf{\beta}_i^\top\mathbf{\alpha}_k}
> {\mathbf{\beta}_i^\top\mathbf{\beta}_i}\mathbf{\beta}_i,
> \qquad k=2,\ldots,s.
> \end{aligned}
> $$
>
> 所得 $\mathbf{\beta}_1,\ldots,\mathbf{\beta}_s$ 均非零且两两正交，并且前 $k$ 个向量与原向量组的前 $k$ 个向量张成相同的空间。再作单位化：
>
> $$
> \mathbf{q}_k=\frac{\mathbf{\beta}_k}{\|\mathbf{\beta}_k\|},
> \qquad k=1,\ldots,s.
> $$
>
> 若原向量组是 $\mathbb{R}^n$ 的一组基，则 $\mathbf{Q}=(\mathbf{q}_1,\ldots,\mathbf{q}_n)$ 满足 $\mathbf{Q}^\top\mathbf{Q}=\mathbf{I}$，即为正交矩阵。

一般方阵也可按 $\mathbf{B}=\mathbf{P}^\top\mathbf{A}\mathbf{P}$ 定义合同。
## 合同性质

以下均指实可逆矩阵给出的合同：

1. **自反性、对称性、传递性。** 分别对应保持原基、反向换基和连续换基。
2. **秩。** $r(\mathbf{B})=r(\mathbf{A})$，因为左右乘的矩阵都可逆。
3. **对称性。** $\mathbf{B}-\mathbf{B}^\top=\mathbf{P}^\top(\mathbf{A}-\mathbf{A}^\top)\mathbf{P}$，所以一个对称，当且仅当另一个也对称。
4. **行列式。** $\det\mathbf{B}=(\det\mathbf{P})^2\det\mathbf{A}$，因此同为零或同号，数值不必相等。
5. **正负性。** 对实对称矩阵，$\mathbf{x}=\mathbf{P}\mathbf{y}$ 是非零向量之间的一一对应，所以正定性与正、负惯性指数保持不变，详见 [[#惯性定理]]。特征值只需符号一致，具体数值不要求。

## 标准型与规范型

标准型只含平方项：

$$
f=d_1y_1^2+\cdots+d_ry_r^2,\qquad d_i\ne0.
$$

规范型进一步把非零系数化为 $\pm1$：

$$
f=z_1^2+\cdots+z_p^2-z_{p+1}^2-\cdots-z_{p+q}^2.
$$

正、负平方项个数分别为 $p,q$，满足 $p+q=r$。


> [!example] 分块二次型的规范形
> 设 $\mathbf{A}=\begin{pmatrix}-1&1\\1&1\end{pmatrix}$，$\mathbf{\alpha}=(a,b)^\top$ 为非零实向量，$\mathbf{x}=(x_1,x_2,x_3)^\top$。给定二次型
>
> $$
> f=\mathbf{x}^\top\mathbf{M}\mathbf{x},\qquad
> \mathbf{M}=\begin{pmatrix}\mathbf{A}+\mathbf{\alpha}\mathbf{\alpha}^\top&\mathbf{\alpha}\\\mathbf{\alpha}^\top&1\end{pmatrix}.
> $$
>
> 求它的规范形。
>
> 记 $\mathbf{Y}=(x_1,x_2)^\top$，则 $\mathbf{x}=\begin{pmatrix}\mathbf{Y}\\x_3\end{pmatrix}$，
>
> $$
> \mathbf{M}\mathbf{x}
> =\begin{pmatrix}(\mathbf{A}+\mathbf{\alpha}\mathbf{\alpha}^\top)\mathbf{Y}+\mathbf{\alpha} x_3\\\mathbf{\alpha}^\top\mathbf{Y}+x_3\end{pmatrix}.
> $$
>
> 于是
>
> $$
> \begin{aligned}
> f
> &=\begin{pmatrix}\mathbf{Y}^\top&x_3\end{pmatrix}\mathbf{M}\mathbf{x}\\
> &=\mathbf{Y}^\top(\mathbf{A}+\mathbf{\alpha}\mathbf{\alpha}^\top)\mathbf{Y}+2x_3\mathbf{\alpha}^\top\mathbf{Y}+x_3^2\\
> &=\mathbf{Y}^\top\mathbf{A}\mathbf{Y}+\underbrace{(\mathbf{\alpha}^\top\mathbf{Y})^2+2x_3\mathbf{\alpha}^\top\mathbf{Y}+x_3^2}_{(\mathbf{\alpha}^\top\mathbf{Y}+x_3)^2}\\
> &=\mathbf{Y}^\top\mathbf{A}\mathbf{Y}+(\mathbf{\alpha}^\top\mathbf{Y}+x_3)^2.
> \end{aligned}
> $$
>
> 两部分彼此独立，可分别化规范型：
>
> 1. $\mathbf{Y}^\top\mathbf{A}\mathbf{Y}$：由 $\det(\mathbf{A}-\lambda\mathbf{I})=\lambda^2-2$，特征值为 $\pm\sqrt{2}$（一正一负），规范型为 $y_1^2-y_2^2$。
> 2. $(\mathbf{\alpha}^\top\mathbf{Y}+x_3)^2$：因 $\mathbf{\alpha}\ne\mathbf{0}$，线性型 $\mathbf{\alpha}^\top\mathbf{Y}+x_3$ 非零，可作新坐标，贡献一个正的平方项 $y_3^2$。
>
> 所以正惯性指数 $p=2$、负惯性指数 $q=1$，规范形为 $y_1^2+y_2^2-y_3^2$。


化为标准型与规范型的方法：配方法/正交变换法。
### 配方法

设二次型的对称矩阵为 $\mathbf{A}=(a_{ij})$。

1. **选取平方项。** 若有非零平方项，交换变量使 $a_{11}\ne0$；将全部含 $x_1$ 的项配成一个平方：

$$
f
=a_{11}\left(x_1+\sum_{j=2}^n\frac{a_{1j}}{a_{11}}x_j\right)^2
+\sum_{i,j=2}^n\left(a_{ij}-\frac{a_{i1}a_{1j}}{a_{11}}\right)x_ix_j.
$$

2. **分离平方项。** 令 $y_1=x_1+\sum_{j=2}^n(a_{1j}/a_{11})x_j$，并令 $y_j=x_j\ (j\ge2)$。替换可逆，余下的二次型只涉及 $n-1$ 个变量，继续同样处理。
3. **无平方项的情形。** 若所有平方项系数都为零，但存在 $a_{ij}\ne0$，先令 $x_i=y_i+y_j$、$x_j=y_i-y_j$，其余变量不变。交叉项 $2a_{ij}x_ix_j$ 变成 $2a_{ij}y_i^2-2a_{ij}y_j^2$，从而可以回到第一步；若全部系数为零，则停止。
4. **规范化。** 得到标准型后，对非零系数令 $z_i=\sqrt{|d_i|}\,y_i$，其余坐标不变，就把各非零平方项系数化为 $\operatorname{sgn}(d_i)$。

各步均为可逆线性替换，并逐次分离平方项。配方法的标准型系数一般不是特征值。

### 正交变换法

1. 求实对称矩阵 $\mathbf{A}$ 的全部特征值及对应特征向量。
2. 不同特征值对应的特征向量相互正交；对重复特征值，在其特征向量组成的空间内选取一组正交单位基。以全部单位特征向量为列组成 $\mathbf{Q}$。
3. 令 $\mathbf{x}=\mathbf{Q}\mathbf{y}$，得到标准型：

$$
f=\mathbf{y}^\top(\mathbf{Q}^\top\mathbf{A}\mathbf{Q})\mathbf{y}
=\lambda_1y_1^2+\cdots+\lambda_ny_n^2.
$$

这里系数就是特征值，并且 $\|\mathbf{x}\|=\|\mathbf{y}\|$。若再缩放为规范型，通常就不再保持长度。

## 惯性定理

实二次型经过任意可逆线性替换，标准型中正平方项、负平方项的个数都不变，分别称为正、负惯性指数。

正惯性指数等于使二次型限制为正定的子空间的最大维数；负惯性指数等于使二次型限制为负定的子空间的最大维数。换基不改变二次型及子空间维数，故正、负惯性指数不变。

若正、负惯性指数为 $p,q$，则秩为 $p+q$，零系数个数为 $n-p-q$。由此：

**两个同阶实对称矩阵合同，当且仅当正、负惯性指数分别相同，即能化为同一个规范型。**

## 正定

实对称矩阵 $\mathbf{A}$ 正定，指对每个非零 $\mathbf{x}$ 都有 $\mathbf{x}^\top\mathbf{A}\mathbf{x}>0$。

下列条件分别与正定等价：

1. **全部特征值为正。** 在正交单位特征向量组成的基下，二次型为 $\sum_i\lambda_i y_i^2$。全部 $\lambda_i>0$ 时，二次型在任意非零向量处取正值；反过来，代入任一特征向量即可得到对应特征值为正。
2. **全部顺序主子式为正。** 左上角的 $1,\ldots,n$ 阶主子式 $\Delta_1,\ldots,\Delta_n$ 都大于零。按变量顺序配方时，平方项系数依次为 $\Delta_1,\Delta_2/\Delta_1,\ldots,\Delta_n/\Delta_{n-1}$，所以都为正；反向由各左上角主子矩阵仍正定得到。
3. **正惯性指数为 $n$。** 规范型为 $z_1^2+\cdots+z_n^2$，即 $\mathbf{A}$ 合同于 $\mathbf{I}$。可逆换基使非零向量仍对应非零坐标，所以二次型严格为正。
4. **可写成 $\mathbf{A}=\mathbf{C}^\top\mathbf{C}$，其中 $\mathbf{C}$ 为实可逆矩阵。** 此时 $\mathbf{x}^\top\mathbf{A}\mathbf{x}=\|\mathbf{C}\mathbf{x}\|^2>0$；反向可从正特征值取平方根构造 $\mathbf{C}$。若 $\mathbf{C}$ 是一般实矩阵，则 $\mathbf{C}^\top\mathbf{C}$ 总半正定，正定当且仅当 $\mathbf{C}$ 满列秩。

## 瑞利商

二次型满足 $f(t\mathbf{x})=t^2f(\mathbf{x})$。对实对称矩阵 $\mathbf{A}$，瑞利商定义为：

$$
R_{\mathbf{A}}(\mathbf{x})
=\frac{\mathbf{x}^\top\mathbf{A}\mathbf{x}}{\mathbf{x}^\top\mathbf{x}},
\qquad \mathbf{x}\ne\mathbf{0}.
$$

它满足 $R_{\mathbf{A}}(t\mathbf{x})=R_{\mathbf{A}}(\mathbf{x})$（$t\ne0$）；当 $\mathbf{x}$ 为特征向量时，瑞利商等于对应特征值。

取正交矩阵 $\mathbf{Q}$ 使 $\mathbf{Q}^\top\mathbf{A}\mathbf{Q}=\operatorname{diag}(\lambda_1,\ldots,\lambda_n)$，令 $\mathbf{x}=\mathbf{Q}\mathbf{y}$，则：

$$
R_{\mathbf{A}}(\mathbf{x})
=\frac{\lambda_1y_1^2+\cdots+\lambda_ny_n^2}{y_1^2+\cdots+y_n^2}.
$$

右侧是特征值的加权平均，权重非负且和为 $1$，所以：

$$
\lambda_{\min}\le R_{\mathbf{A}}(\mathbf{x})\le\lambda_{\max}.
$$

等号分别在最小、最大特征值对应的非零特征向量处取得。
