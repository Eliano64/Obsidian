---
title: Gradient Descent
tags:
  - MachineLearning
categories:
  - Artificial Intelligence
date: 2026-10-08 16:49:56
katex: true
---

# 为什么需要梯度下降

**梯度下降（Gradient Descent）** 是面向无闭式解可微目标 $\min_{\boldsymbol{\theta}} L(\boldsymbol{\theta})$ 的通用一阶迭代优化方法（$\boldsymbol{\theta}$ 为参数向量，$L$ 为损失函数）。面对海量样本、高维特征或深度网络非凸地形，直接求逆解解析式（如 [[Linear-Regression#正规方程|正规方程]]）不再现实，工程转而按「前向算损失 → 反向求梯度 → 负梯度更新 → 重复」四步循环逐轮压低损失。

![[../assets/gradient-descent-training-loop.svg]]

# 梯度计算与迭代更新

损失对各参数偏导组成的向量 $\nabla L$ 指向损失上升最快的方向。沿用 [[Linear-Regression|线性回归]] MSE 记号（样本数 $n$、特征数 $d$、设计矩阵 $\mathbf{X}\in\mathbb{R}^{n\times d}$、标签 $\boldsymbol{y}\in\mathbb{R}^n$、权重 $\boldsymbol{\theta}\in\mathbb{R}^d$、偏置标量 $b$、全 1 向量 $\boldsymbol{1}\in\mathbb{R}^n$、残差 $\boldsymbol{r}=\mathbf{X}\boldsymbol{\theta}+b\boldsymbol{1}-\boldsymbol{y}$），记第 $i$ 个样本输入为 $\boldsymbol{x}_i$、残差为 $r_i$，则以学习率 $\eta > 0$ 同步执行更新：

$$
\frac{\partial L}{\partial \boldsymbol{\theta}} = \frac{2}{n}\mathbf{X}^{\top}\boldsymbol{r}, \qquad
\frac{\partial L}{\partial b} = \frac{2}{n}\sum_{i=1}^{n} r_i, \qquad
\boldsymbol{\theta} \leftarrow \boldsymbol{\theta} - \eta \frac{\partial L}{\partial \boldsymbol{\theta}}, \qquad
b \leftarrow b - \eta \frac{\partial L}{\partial b}
$$

[html-card height=720 step=80](../assets/gradient-descent-surface.html)

## 学习率与迭代骨架

**学习率 $\eta$** 决定每步更新幅度的全局比例：过小则平缓区推进缓慢，过大则在陡坡来回震荡甚至数值发散。实践中常按训练进度动态调整步长，详情见 [[Learning-Rate-Schedule|学习率调度]]。

![[../assets/gradient-descent-learning-rate.svg]]

```text
输入 X, y, 学习率 η, 梯度阈值 ε, 最大步数 T_max
初始化 θ, b
for t = 1 to T_max:
    r  = Xθ + b - y          // b 沿样本维广播
    gθ = (2/n) Xᵀ r          // ∂L/∂θ
    gb = (2/n) 1ᵀ r          // ∂L/∂b
    θ = θ - η gθ             // 使用同一组参数梯度同步更新
    b = b - η gb
    if ‖∇L(θ, b)‖_2 < ε:
        break
返回 θ, b
```

> [!important] 一阶停止条件只是必要条件
> $\lVert\nabla L(\boldsymbol{\theta},b)\rVert_2 < \varepsilon$ 仅说明一阶导趋零，在非凸损失中不能区分局部极小与鞍点；设置最大步数 $T_{\max}$ 用于封顶计算预算。

## 批大小：BGD、SGD 与 Mini-batch

批大小（Batch Size）决定单步梯度噪声水平与硬件并行效率，更改批大小时通常需联动调整学习率：

| 做法 | 单步样本量 | 梯度方差与算力特性 |
| --- | --- | --- |
| 全量梯度下降（BGD） | $n$ | 无抽样噪声、方向最稳；大数据集每步扫描耗时且显存受限 |
| 单样本随机梯度（SGD） | $1$ | 单步极轻但方差大，极小点附近剧烈抖动 |
| 小批量梯度下降（Mini-batch） | 批量 $B$（如 $32, 64$） | 方差可控且充分调用矩阵并行，是深度学习默认骨架 |

# 优化器的演化

标准小批量 SGD 对所有参数共用同一标量学习率，遇到狭长山谷地形时容易在陡轴来回折返、在缓轴推进龟速。现代一阶优化器沿两条主线改良：引入**方向惯性**与引入**逐参数尺度缩放**（下文记当前步梯度为 $\boldsymbol{g}$，$\odot$ 表示逐元素乘，$\delta$ 为防除零小常数）。

## Momentum（修方向）

![[../assets/gradient-descent-optimizer-momentum.svg]]

**Momentum** 维护与参数同维的速度向量 $\boldsymbol{v}$（零初值，动量系数 $\beta\in[0,1)$ 常取 $0.9$），用历史梯度加权惯性推进更新：

$$
\boldsymbol{v} \leftarrow \beta \boldsymbol{v} + \boldsymbol{g}, \qquad
\boldsymbol{\theta} \leftarrow \boldsymbol{\theta} - \eta \boldsymbol{v}
$$

反复震荡维上的正负梯度相互抵消，持续一致维上不断累加提速，还能靠惯性滑过平坦区；带 `momentum=0.9` 的小批量 SGD 至今仍是视觉卷积网络（如 ResNet）经典标配。

## AdaGrad 与 RMSProp（修逐参数尺度）

![[../assets/gradient-descent-optimizer-adagrad.svg]]

遇到稀疏特征（高低频词梯度方差悬殊）时，**AdaGrad** 用向量 $\boldsymbol{s}$ 累加各坐标历史梯度平方并按 $\sqrt{\boldsymbol{s}}+\delta$ 缩放步长，让低频参数获得更大相对步幅；代价是 $\boldsymbol{s}$ 单调只增不减，深网中后期步长常过早跌停：

$$
\boldsymbol{s} \leftarrow \boldsymbol{s} + \boldsymbol{g} \odot \boldsymbol{g}, \qquad
\boldsymbol{\theta} \leftarrow \boldsymbol{\theta} - \eta \frac{\boldsymbol{g}}{\sqrt{\boldsymbol{s}} + \delta}
$$

**RMSProp** 改用指数滑动平均（衰减率 $\beta_2\in[0,1)$ 常取 $0.9$）让 $\boldsymbol{s}$ 只反映近期梯度尺度，消除单调衰减缺陷：

![[../assets/gradient-descent-optimizer-rmsprop.svg]]

$$
\boldsymbol{s} \leftarrow \beta_2 \boldsymbol{s} + (1 - \beta_2)\boldsymbol{g} \odot \boldsymbol{g}, \qquad
\boldsymbol{\theta} \leftarrow \boldsymbol{\theta} - \eta \frac{\boldsymbol{g}}{\sqrt{\boldsymbol{s}} + \delta}
$$

## Adam 与 AdamW（双旋钮合体与解耦）

![[../assets/gradient-descent-optimizer-adam.svg]]

**Adam** 同时融合一阶矩估计 $\boldsymbol{m}$（方向惯性，$\beta_1$ 常取 $0.9$）与二阶矩估计 $\boldsymbol{s}$（自适应缩放，$\beta_2$ 常取 $0.999$），第 $t$ 步因零初值早期偏小，需分别除以 $1-\beta_1^t$ 与 $1-\beta_2^t$ 做偏差修正：

$$
\boldsymbol{m} \leftarrow \beta_1 \boldsymbol{m} + (1 - \beta_1)\boldsymbol{g}, \qquad
\boldsymbol{s} \leftarrow \beta_2 \boldsymbol{s} + (1 - \beta_2)\boldsymbol{g} \odot \boldsymbol{g}
$$

$$
\hat{\boldsymbol{m}} = \frac{\boldsymbol{m}}{1 - \beta_1^t}, \qquad
\hat{\boldsymbol{s}} = \frac{\boldsymbol{s}}{1 - \beta_2^t}, \qquad
\boldsymbol{\theta} \leftarrow \boldsymbol{\theta} - \eta \frac{\hat{\boldsymbol{m}}}{\sqrt{\hat{\boldsymbol{s}}} + \delta}
$$

当加入权重衰减（系数 $\lambda$）时，若直接将 L2 项写进损失，惩罚会被分母 $\sqrt{\hat{\boldsymbol{s}}}$ 扭曲放缩；**AdamW** 将权重衰减从梯度通道完全解耦直接作用在 $\boldsymbol{\theta}$ 上（见 [[Regularization#自适应优化器中的解耦：AdamW|AdamW 与权重衰减解耦]]），成为 Transformer 与大模型默认标准：

$$
\boldsymbol{\theta} \leftarrow \boldsymbol{\theta} - \eta \left(\frac{\hat{\boldsymbol{m}}}{\sqrt{\hat{\boldsymbol{s}}} + \delta} + \lambda \boldsymbol{\theta}\right)
$$

## Muon（矩阵正交化视角）

**Muon** 不再把权重看作独立标量，而是面向二维线性层权重矩阵 $\mathbf{W}$ 直接在矩阵空间做更新：对矩阵动量 $\mathbf{M}$ 施加 Newton-Schulz 近似正交化算子 $\operatorname{orth}(\cdot)$，把各奇异方向步长拉齐避免少数极大奇异值垄断更新；一维偏置、嵌入与归一化参数仍配 AdamW：

$$
\mathbf{M} \leftarrow \beta \mathbf{M} + \nabla L(\mathbf{W}), \qquad
\mathbf{W} \leftarrow \mathbf{W} - \eta \, \operatorname{orth}(\mathbf{M})
$$

## 优化器选型

优化器决定沿梯度前进方式，无法挽救错误损失或脏数据；主流场景选型如下：

| 业务与架构场景 | 首选优化器 | 核心依据 |
| --- | --- | --- |
| 视觉卷积网络（ResNet 等） | 小批量 SGD + Momentum（常取 $0.9$） | 泛化稳健，且无一/二阶矩显存状态开销 |
| Transformer、ViT 与大语言模型 | AdamW（二维线性权重可尝试 Muon） | 兼顾动量、自适应步长与一致的解耦权重衰减 |
| 快速验证与小规模原型 | Adam | 默认配置（如 $\eta=10^{-3}$）极易获得稳定基线 |
| 高维稀疏与在线流式任务 | AdaGrad | 自动保持低频稀缺特征的更新步长 |
