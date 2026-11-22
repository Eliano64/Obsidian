---
title: Linear Regression
tags:
  - MachineLearning
categories:
  - Artificial Intelligence
date: 2026-10-08 16:38:35
katex: true
---

# 线性回归与线性性

在 [[Machine-Learning-Overview#监督学习|监督学习]] 中，待预测标签为连续实数时称为**回归**；**线性回归**用输入特征的线性加权和加偏置构造预测输出。数学上，映射 $f$ 对向量 $\boldsymbol{x},\boldsymbol{z}$ 与标量 $a,c$ 满足齐次性与可加性 $f(a\boldsymbol{x}+c\boldsymbol{z}) = af(\boldsymbol{x})+cf(\boldsymbol{z})$ 才称线性；带非零偏置 $b$ 时模型对输入是仿射映射，名称中的「线性」指固定特征时预测对**待求参数**呈线性（因此后文多项式回归仍属线性模型）。

# 从逐特征展开到矩阵形式

面对 $d$ 个特征（第 $j$ 个特征记作 $x_j$，对应权重 $\theta_j$），将单样本特征列向量记为 $\boldsymbol{x}=(x_1,\dots,x_d)^{\top}\in\mathbb{R}^d$、权重记为 $\boldsymbol{\theta}=(\theta_1,\dots,\theta_d)^{\top}\in\mathbb{R}^d$，预测写作 $\hat{y} = \boldsymbol{\theta}^{\top}\boldsymbol{x} + b$。

共有 $n$ 个样本时，把各样本行向量堆成设计矩阵 $\mathbf{X}\in\mathbb{R}^{n\times d}$（第 $i$ 行为 $\boldsymbol{x}_i^{\top}$），全 1 列向量记作 $\boldsymbol{1}\in\mathbb{R}^n$，全部样本预测向量 $\hat{\boldsymbol{y}}\in\mathbb{R}^n$ 写为：

$$
\hat{\boldsymbol{y}} = \mathbf{X}\boldsymbol{\theta} + b\boldsymbol{1}
$$

在 $\mathbf{X}\in\mathbb{R}^{n\times d}$ 中，**行数 $n$ 是样本数，列数 $d$ 是特征数**；矩阵乘法因此要求 $\boldsymbol{\theta}\in\mathbb{R}^d$ 与列数对齐，输出 $\hat{\boldsymbol{y}}$ 长度等于行数 $n$。

> [!question] 为何全部样本共享同一个偏置
> 若允许每个样本各自拥有独立偏置（向量 $\boldsymbol{b}\in\mathbb{R}^n$），对任意 $\boldsymbol{\theta}$ 都可令 $\boldsymbol{b}=\boldsymbol{y}-\mathbf{X}\boldsymbol{\theta}$ 使训练误差直接为零，模型退化为死记单个样本且解不唯一；共享单一标量 $b$（经广播加到各行）仅占 $d+1$ 个参数。

# 损失函数与凸性

记真实标签列向量为 $\boldsymbol{y}\in\mathbb{R}^n$，第 $i$ 个样本残差为 $r_i = \hat{y}_i - y_i$（残差向量 $\boldsymbol{r} = \hat{\boldsymbol{y}} - \boldsymbol{y}$）。把全 1 列并入增广设计矩阵 $\tilde{\mathbf{X}} = [\mathbf{X}, \boldsymbol{1}]\in\mathbb{R}^{n\times(d+1)}$、偏置并入增广参数 $\tilde{\boldsymbol{\theta}} = [\boldsymbol{\theta}^{\top}, b]^{\top}\in\mathbb{R}^{d+1}$，均方误差（MSE）损失写为：

$$
L(\tilde{\boldsymbol{\theta}}) = \frac{1}{n}\sum_{i=1}^{n} r_i^2 = \frac{1}{n}\boldsymbol{r}^{\top}\boldsymbol{r} = \frac{1}{n}\left\|\tilde{\mathbf{X}}\tilde{\boldsymbol{\theta}} - \boldsymbol{y}\right\|_2^2
$$

该二次型的海森矩阵 $\frac{2}{n}\tilde{\mathbf{X}}^{\top}\tilde{\mathbf{X}}$ 必半正定，$L$ 必为凸函数；当 $\tilde{\mathbf{X}}$ 列满秩时严格凸且全局极小唯一。平方项消除正负抵消并放大高残差以优先纠偏，代价是量纲平方且易受异常离群点拉偏。

![[../assets/linear-regression-fit.svg]]

# 参数求解

## 正规方程

令凸二次损失梯度 $\frac{2}{n}\tilde{\mathbf{X}}^{\top}(\tilde{\mathbf{X}}\tilde{\boldsymbol{\theta}}-\boldsymbol{y}) = \mathbf{0}$ 即得**正规方程**；当 $\tilde{\mathbf{X}}^{\top}\tilde{\mathbf{X}}$ 可逆时存在唯一解析解：

$$
\tilde{\mathbf{X}}^{\top}\tilde{\mathbf{X}}\,\tilde{\boldsymbol{\theta}} = \tilde{\mathbf{X}}^{\top}\boldsymbol{y}
\qquad \Longrightarrow \qquad
\tilde{\boldsymbol{\theta}} = \left(\tilde{\mathbf{X}}^{\top}\tilde{\mathbf{X}}\right)^{-1}\tilde{\mathbf{X}}^{\top}\boldsymbol{y}
$$

特征少、可逆时解析解一步算完；若特征共线或 $n < d+1$ 导致矩阵奇异，需删特征、用伪逆或施加正则约束（详见 [[Regularization]]）。由于显式求逆 $\tilde{\mathbf{X}}^{\top}\tilde{\mathbf{X}}$ 会将矩阵条件数平方 $\kappa(\tilde{\mathbf{X}}^{\top}\tilde{\mathbf{X}}) = \kappa(\tilde{\mathbf{X}})^2$ 进而放大舍入误差，工业库（如 `numpy.linalg.lstsq`、`scikit-learn`）底层直接对 $\tilde{\mathbf{X}}$ 做 **QR 分解**或 **SVD** 构造 Moore-Penrose 伪逆 $\tilde{\mathbf{X}}^{+}$ 求解 $\tilde{\boldsymbol{\theta}} = \tilde{\mathbf{X}}^{+}\boldsymbol{y}$，在秩亏损时还能给出最小欧氏范数解。

## 迭代求解

特征维数或样本量巨大时矩阵分解开销过高，工程改用只需矩阵乘法的一阶迭代法逐轮逼近极小点（机制见 [[Gradient-Descent]]）。两条求解路径优化同一目标函数 $L$，可逆时迭代在合适步长下收敛于解析解同一点。

# 从线性回归出发

改动特征构造、输出激活或层数，即可由线性回归衍生出三类经典结构：

- **多项式回归**：对单特征 $x$ 扩充 $p$ 阶幂次列（$\hat{y} = \theta_1 x + \theta_2 x^2 + \dots + \theta_p x^p + b$），模型对待求参数仍为线性，直接复用线性求解形式；代价是同一特征各次幂高度共线使 $\tilde{\mathbf{X}}^{\top}\tilde{\mathbf{X}}$ 接近奇异，阶数 $p$ 不宜过高。
- **逻辑回归**：将线性打分送入 Sigmoid 激活 $\sigma$ 输出概率（$P(y=1\mid\boldsymbol{x}) = \sigma(\boldsymbol{\theta}^{\top}\boldsymbol{x}+b)$）并配交叉熵处理分类任务（详见 [[Logistic-Regression]]）。
- **深度学习**：把「线性变换 + 非线性激活」多层堆叠，让模型自动学习高层特征表示而非依赖人工构造幂次列（见 [[Deep-Learning-Study-Route|深度学习]]）。
