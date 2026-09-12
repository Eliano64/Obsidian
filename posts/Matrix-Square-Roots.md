---
title: Matrix Square Roots
tags:
  - Matrices
categories:
  - Linear Algebra
date: 2026-09-02 12:23:25
katex: true
---
# 矩阵开方

矩阵开方是求矩阵 $\mathbf X$，使

$$
\mathbf X^2=\mathbf A.
$$


若 $\mathbf A$ 可对角化，先求

$$
\mathbf A=\mathbf P
\operatorname{diag}(\lambda_1,\ldots,\lambda_n)
\mathbf P^{-1}.
$$

选择 $\mu_i^2=\lambda_i$，则

$$
\mathbf X=\mathbf P
\operatorname{diag}(\mu_1,\ldots,\mu_n)
\mathbf P^{-1}
$$

满足 $\mathbf X^2=\mathbf A$。

> [!example] 具体矩阵
> 求
> $$
> \mathbf A=
> \begin{pmatrix}
> 2&1&1\\
> 1&2&1\\
> 1&1&2
> \end{pmatrix}
> $$
> 的正定平方根。
>
> 取单位特征向量
> $$
> \boldsymbol q_1=\frac1{\sqrt3}
> \begin{pmatrix}1\\1\\1\end{pmatrix},\qquad
> \boldsymbol q_2=\frac1{\sqrt2}
> \begin{pmatrix}1\\-1\\0\end{pmatrix},\qquad
> \boldsymbol q_3=\frac1{\sqrt6}
> \begin{pmatrix}1\\1\\-2\end{pmatrix}.
> $$
> 对应特征值依次为 $4,1,1$。令
> $$
> \mathbf Q=
> \begin{pmatrix}
> \boldsymbol q_1&\boldsymbol q_2&\boldsymbol q_3
> \end{pmatrix},
> $$
> 则
> $$
> \mathbf A^{1/2}
> =\mathbf Q\operatorname{diag}(2,1,1)\mathbf Q^{\top}\text{(负值舍去)}
> =\frac13
> \begin{pmatrix}
> 4&1&1\\
> 1&4&1\\
> 1&1&4
> \end{pmatrix}.
> $$
> 验算：
> $$
> \left(\mathbf A^{1/2}\right)^2
> =\frac19
> \begin{pmatrix}
> 18&9&9\\
> 9&18&9\\
> 9&9&18
> \end{pmatrix}
> =\mathbf A.
> $$

若 $\mathbf A$ 不能对角化，设 $\mathbf B^2=\mathbf A$，则

$$
\mathbf A\mathbf B
=\mathbf B^2\mathbf B
=\mathbf B^3
=\mathbf B\mathbf B^2
=\mathbf B\mathbf A.
$$

因此，先设出 $\mathbf B$ 的全部元素，用 $\mathbf A\mathbf B=\mathbf B\mathbf A$ 得到线性方程，减少未知量；再代入 $\mathbf B^2=\mathbf A$。

最后仍须检验平方。

> [!example] 不能对角化的具体矩阵
> 求矩阵
> $$
> \mathbf A=
> \begin{pmatrix}
> 1&1\\
> 0&1
> \end{pmatrix}
> $$
> 的平方根。它不能对角化。设
> $$
> \mathbf B=
> \begin{pmatrix}
> x&y\\
> z&w
> \end{pmatrix}.
> $$
> 先用 $\mathbf A\mathbf B=\mathbf B\mathbf A$：
> $$
> \mathbf A\mathbf B=
> \begin{pmatrix}
> x+z&y+w\\
> z&w
> \end{pmatrix},\qquad
> \mathbf B\mathbf A=
> \begin{pmatrix}
> x&x+y\\
> z&z+w
> \end{pmatrix}.
> $$
> 比较对应元素，得
> $$
> z=0,\qquad w=x.
> $$
> 因而
> $$
> \mathbf B=
> \begin{pmatrix}
> x&y\\
> 0&x
> \end{pmatrix},\qquad
> \mathbf B^2=
> \begin{pmatrix}
> x^2&2xy\\
> 0&x^2
> \end{pmatrix}.
> $$
> 由 $\mathbf B^2=\mathbf A$，
> $$
> x^2=1,\qquad 2xy=1.
> $$
> 故
> $$
> \mathbf B=
> \begin{pmatrix}
> 1&\dfrac12\\
> 0&1
> \end{pmatrix}
> \quad\text{或}\quad
> \mathbf B=
> \begin{pmatrix}
> -1&-\dfrac12\\
> 0&-1
> \end{pmatrix}.
> $$
