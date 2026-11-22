---
title: Logistic Regression
tags:
  - MachineLearning
categories:
  - Artificial Intelligence
date: 2026-10-09 23:11:36
katex: true
---

# 从线性回归到 Sigmoid 概率映射

**逻辑回归（Logistic Regression）** 针对二分类任务，在 [[Linear-Regression|线性回归]] 打分 $z = \boldsymbol{w}^{\top}\boldsymbol{x} + b$（其中特征向量 $\boldsymbol{x}\in\mathbb{R}^d$、权重 $\boldsymbol{w}\in\mathbb{R}^d$、偏置 $b\in\mathbb{R}$）末端接 **Sigmoid 函数**，将无界实数压缩进开区间 $(0,1)$ 作为正类后验概率 $\hat{y} = P(y=1 \mid \boldsymbol{x})$：

$$
\sigma(z) = \frac{1}{1+e^{-z}}, \qquad
\hat{y} = \sigma\left(\boldsymbol{w}^{\top}\boldsymbol{x} + b\right) = \frac{1}{1+e^{-(\boldsymbol{w}^{\top}\boldsymbol{x}+b)}}
$$

![[../assets/logistic-sigmoid.svg]]

Sigmoid 严格单调递增且满足 $\sigma(-z) = 1 - \sigma(z)$；取默认阈值 $0.5$ 时，$\hat{y} \ge 0.5$ 等价于线性超平面判定 $z \ge 0$。

> [!note] 为何不直接用阶跃函数，且如何非线性扩展
> 阶跃函数在 $z=0$ 不连续、其余处导数为 $0$ 无法梯度训练；Sigmoid 作为其光滑近似，导数满足 $\sigma'(z) = \sigma(z)(1-\sigma(z))$（中心最高 $0.25$、两端趋零）。线性超平面本身只能分隔线性关系，复杂边界需配合 [[Data-Preprocessing#特征构造|特征构造]] 或改用 [[Decision-Tree|决策树]]。

# 损失函数：为什么用交叉熵

## MSE 与 Sigmoid 的梯度消失

若把单样本平方误差 $L_{\text{MSE}} = \frac{1}{2}(\hat{y}-y)^2$（真实标签 $y\in\{0,1\}$）接在 Sigmoid 之后，不仅令损失非凸产生局部极小，还会触发严重**梯度消失**：

$$
\frac{\partial L_{\text{MSE}}}{\partial z} = (\hat{y}-y)\,\sigma'(z) = (\hat{y}-y)\,\hat{y}(1-\hat{y})
$$

当真实标签 $y=1$ 而模型极端错误预测 $\hat{y}\approx 0$ 时，误差项 $(\hat{y}-y) = -1$ 虽大，因子 $\hat{y}(1-\hat{y})\approx 0$ 却将总梯度抹零，造成“越自信犯错、更新越停滞”。

## 从最大似然到二元交叉熵

将二分类输出视作参数为 $\hat{y}$ 的伯努利分布，单样本命中真实标签 $y\in\{0,1\}$ 的概率为 $\hat{y}^{y}(1-\hat{y})^{1-y}$。设数据集含 $n$ 个独立样本（第 $i$ 个样本标签与预测为 $y_i, \hat{y}_i$），待求参数记作 $\boldsymbol{\theta}=(\boldsymbol{w},b)$，对联合似然 $\prod_{i=1}^n \hat{y}_i^{y_i}(1-\hat{y}_i)^{1-y_i}$ 取对数（避免连乘数值下溢）、添负号再对 $n$ 取平均，即得最小化**二元交叉熵（BCE）**：

$$
J(\boldsymbol{\theta}) = -\frac{1}{n}\sum_{i=1}^{n}\left[y_i\log\hat{y}_i + (1-y_i)\log(1-\hat{y}_i)\right]
$$

每个样本只激活真实类别对应一项：预测越接近标签损失越趋近 $0$，自信判错则损失趋向 $+\infty$；且代入 Sigmoid 后 $J(\boldsymbol{\theta})$ 是关于 $\boldsymbol{\theta}$ 的**凸函数**，消除了局部极小。

# 梯度与线性回归的统一形式

对单样本损失 $J_i = -[y\log\hat{y}+(1-y)\log(1-\hat{y})]$ 先对 $\hat{y}$ 求导，再乘 $\sigma'(z) = \hat{y}(1-\hat{y})$，分母与 Sigmoid 导数精确相消：

$$
\frac{\partial J_i}{\partial \hat{y}} = \frac{\hat{y}-y}{\hat{y}(1-\hat{y})}
\quad\Longrightarrow\quad
\frac{\partial J_i}{\partial z} = \frac{\partial J_i}{\partial \hat{y}}\cdot \sigma'(z) = \hat{y} - y
$$

自信犯错时的梯度保持在最大幅值而不消失。再对全部 $n$ 个样本平均（$x_{i,j}$ 为第 $i$ 样本第 $j$ 特征），逻辑回归与线性回归在梯度形态上完全统一（如下表；决策阈值 $\tau$ 与分类评估指标详析见 [[Model-Evaluation#分类评估指标|分类评估指标]]）：

| 梯度项 | 线性回归（MSE） | 逻辑回归（交叉熵） |
| --- | --- | --- |
| 权重分量 $w_j$ 梯度 | $\frac{2}{n}\sum_{i=1}^{n}(\hat{y}_i - y_i)x_{i,j}$ | $\frac{1}{n}\sum_{i=1}^{n}(\hat{y}_i - y_i)x_{i,j}$ |
| 偏置 $b$ 梯度 | $\frac{2}{n}\sum_{i=1}^{n}(\hat{y}_i - y_i)$ | $\frac{1}{n}\sum_{i=1}^{n}(\hat{y}_i - y_i)$ |
| 输出 $\hat{y}_i$ 形式 | 线性输出 $\boldsymbol{w}^{\top}\boldsymbol{x}_i + b$ | 概率输出 $\sigma(\boldsymbol{w}^{\top}\boldsymbol{x}_i + b)$ |

# 多分类：Softmax 与单层神经网络

扩充至 $K$ 类（$k=1,\dots,K$）时，先由各组参数 $(\boldsymbol{w}_k, b_k)$ 算出未归一化 logits $z_k = \boldsymbol{w}_k^{\top}\boldsymbol{x}+b_k$，再经 **Softmax** 归一化为概率分布（记真实类别为第 $c$ 类、one-hot 标签向量为 $\boldsymbol{y}$，仅 $y_c=1$ 其余 $y_j=0$）：

$$
\hat{y}_k = \frac{e^{z_k}}{\sum_{j=1}^{K}e^{z_j}}, \qquad
L = -\sum_{k=1}^{K}y_k\log\hat{y}_k = -\log\hat{y}_c
$$

分别求导得 $\frac{\partial\hat{y}_c}{\partial z_c} = \hat{y}_c(1-\hat{y}_c)$、$\frac{\partial\hat{y}_c}{\partial z_k} = -\hat{y}_c\hat{y}_k$（$k\ne c$），代入 $-\frac{1}{\hat{y}_c}\frac{\partial\hat{y}_c}{\partial z_k}$ 后合并为统一简洁形式：

$$
\frac{\partial L}{\partial z_k} = \hat{y}_k - y_k
$$

「线性层 + Softmax + 交叉熵」本质就是一个零隐藏层单层神经网络，也是现代深层分类网络末端的标准模块（更多模型体系见 [[Machine-Learning-Overview]]）。

# 常见错误：`nn.Softmax` 与 `CrossEntropyLoss` 叠加

PyTorch 的 `nn.CrossEntropyLoss` 内部默认接收未归一化 logits 并自动融合 `log_softmax` 与负对数似然 `NLLLoss`。若先手动加 `nn.Softmax(dim=-1)` 得到 $\boldsymbol{p}=(p_1,\dots,p_K)^{\top}\in(0,1)^K$，即使正确类完美预测 $p_c\to 1$、其余 $p_{j\ne c}\to 0$，二次归一化概率 $\tilde{p}_c$ 仍会被封顶：

$$
\tilde{p}_c = \frac{e^{1}}{e^{1} + (K-1)e^{0}} = \frac{e}{e + K - 1}
$$

在 10 分类下 $\tilde{p}_c \le 0.232$、损失永远卡在 $1.46$ 以上，且反向还多乘一层近零导数 $p_c(1-p_c)$。正确写法是训练期直接传 logits 给 `CrossEntropyLoss`，仅在推理读取概率时调用 `torch.softmax`：

```python
model = nn.Sequential(
    ...,
    nn.Linear(128, 10),           # 直接输出未归一化的 logits
)
criterion = nn.CrossEntropyLoss() # 内部融合 log-sum-exp 保证数值稳定
```
