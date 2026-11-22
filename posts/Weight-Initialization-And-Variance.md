---
title: Weight Initialization And Variance
tags:
  - DeepLearning
categories:
  - Artificial Intelligence
date: 2026-10-26 23:42:25
katex: true
---

# 初始化的目标

**权重初始化**决定训练第 0 步的参数取值，需同时满足三个目标：

1. **打破对称性**：同层不同单元能学出不同特征，不退化为重复副本。
2. **维持前向方差**：信号逐层传递时不呈指数膨胀或坍缩。
3. **稳定反向梯度**：误差连乘时不出现剧烈爆炸或归零。

训练开始后的分布漂移不归初始化控制，相关机制见 [[Norm]]。

# 对称性

若同层各单元的输入权重向量、输出连接、偏置与优化器状态完全相同，则各单元在前向输出与反向梯度上完全一致，参数更新后依然相同，整层退化为单一单元。实践中通过**随机初始化权重**打破输入侧对称性；偏置通常直接初始化为 0，因为各单元的权重已不相同，同一输入下的预激活与梯度自然分开。全零初始化仅在无隐藏单元区分需求的单层线性或逻辑回归模型上可行。

# 方差的传播

考虑线性层中单个输出单元的预激活：设 $\text{fan\_in}$（记作 $n$）为该单元接收的输入个数，$\text{fan\_out}$ 为本层输出单元个数，输入分量 $x_j$ 与权重 $w_{ij}$ 均零均值、相互独立且同层各坐标同方差，预激活 $z_i$ 的方差满足：

$$
z_i = \sum_{j=1}^{n} w_{ij} x_j \quad\Rightarrow\quad \operatorname{Var}(z_i) = \sum_{j=1}^{n} \operatorname{Var}(w_{ij})\operatorname{Var}(x_j) = n \cdot \operatorname{Var}(w) \cdot \operatorname{Var}(x)
$$

要使输出方差期望上等于输入方差，单权重方差需取：

$$
\operatorname{Var}(w) = \frac{1}{\text{fan\_in}}
$$

以 $\text{fan\_in}=100$、输入方差为 $1$ 为例：权重方差取 $0.01$ 时输出方差期望保持 $1$；若改为 $0.02$（每层放大 2 倍），20 层后方差放大至 $2^{20} \approx 1.05\times 10^{6}$；若改为 $0.005$（每层减半），20 层后仅剩百万分之一。真实有限样本上统计量会在理论期望附近波动，且激活饱和还会进一步改变尺度：

![方差在层间的传播](../assets/dl-init-variance.svg)

图中纯线性串联按 $1.5$ 倍 20 层后放大 $3325$ 倍、按 $0.8$ 倍缩至 $0.0115$；下方 15 层 tanh 网络即使按 $1/\sqrt{\text{fan\_in}}$ 初始化，由于 tanh 饱和压缩，输出标准差仍从 $0.98$ 降至 $0.19$。

# 为什么下一层看二阶矩（ReLU 的修正）

非零中心激活（如 ReLU）输出均值为正，但这不意味传向下一层的系数直接变为输出方差。设上一层经过 ReLU 的激活为 $a_j = \max(0, z_j)$，下一层权重 $w_{ij}$ 零均值且与 $a_j$ 独立，则乘积均值为 $\mathbb{E}[w_{ij}a_j] = \mathbb{E}[w_{ij}]\mathbb{E}[a_j] = 0$，因此该项方差等于：

$$
\operatorname{Var}(w_{ij} a_j) = \mathbb{E}[w_{ij}^2 a_j^2] - \left(\mathbb{E}[w_{ij} a_j]\right)^2 = \operatorname{Var}(w_{ij}) \cdot \mathbb{E}[a_j^2]
$$

**真正跨层传递的尺度量是激活的二阶矩 $\mathbb{E}[a^2]$，而非中心方差 $\operatorname{Var}(a)$。** 设预激活 $z$ 的概率密度 $p(z)$ 关于 0 对称且均值为 0，经过 ReLU 截去负半轴后，二阶矩恰好减半：

$$
\mathbb{E}[a^2] = \int_0^{\infty} z^2 p(z)\,\mathrm{d}z = \frac{1}{2}\mathbb{E}[z^2]
$$

这一减半结论对任意零均值对称分布均成立，不依赖高斯分布假设。

# 初始化方法

## Xavier 初始化

记 $w$ 为权重矩阵中的单个标量元素。**Xavier 初始化**在满足激活函数于原点附近近似线性（如 tanh）的前提下，折中前向方差守恒（$1/\text{fan\_in}$）与反向梯度方差守恒（$1/\text{fan\_out}$）（见 [Glorot 与 Bengio](https://proceedings.mlr.press/v9/glorot10a.html)）：

$$
\operatorname{Var}(w) = \frac{2}{\text{fan\_in} + \text{fan\_out}}
$$

Sigmoid 虽然非零中心且原点斜率为 $0.25$，但因与 tanh 同属有界平滑激活，经典场景下也常以 Xavier 作为同量级起点。

## He 初始化

**He 初始化**针对 ReLU 截断负半轴引起的二阶矩减半修正权重尺度（见 [He 等](https://arxiv.org/abs/1502.01852)）。设输入层预激活 $z$ 零均值对称、方差为 $q$，经 ReLU 后激活 $a_j$ 满足 $\mathbb{E}[a_j^2] = q/2$，则下一层预激活 $z' = \sum_{j=1}^{\text{fan\_in}} w_j a_j$ 的方差为 $\text{fan\_in} \cdot \operatorname{Var}(w) \cdot \frac{q}{2}$。令其等于 $q$ 即得：

$$
\operatorname{Var}(w) = \frac{2}{\text{fan\_in}}
$$

对负半轴斜率为 $\alpha$ 的 Leaky ReLU，二阶矩保留比例变为 $(1+\alpha^2)/2$，对应权重方差修正为 $\operatorname{Var}(w) = \frac{2}{(1+\alpha^2)\,\text{fan\_in}}$（$\alpha=0$ 回到 ReLU，$\alpha=1$ 退化为线性层 $1/\text{fan\_in}$）。

# 框架默认行为

PyTorch 中 `nn.Linear` 在构造时默认从 $[-1/\sqrt{\text{fan\_in}}, 1/\sqrt{\text{fan\_in}}]$ 均匀分布采样权重（见 [nn.Linear 文档](https://pytorch.org/docs/stable/generated/torch.nn.Linear.html)），方差为 $\frac{1}{3\,\text{fan\_in}}$，标准差仅为标准 He 初始化的 $1/\sqrt{6} \approx 0.408$。中等深度配合归一化层时默认尺度通常可训，纯 ReLU 深层网络建议显式初始化：

```python
import torch
from torch import nn

layer = nn.Linear(256, 128)
# 显式指定 nonlinearity="relu" 让意图清晰，避免默认参数混淆
nn.init.kaiming_normal_(layer.weight, nonlinearity="relu")
```

> [!note] `kaiming_*` 的参数默认值
> 公开接口 `nn.init.kaiming_normal_` / `kaiming_uniform_` 的 `nonlinearity` 默认虽为 `'leaky_relu'`，但其负斜率 `a` 默认取 `0`，因此实际增益为 $\sqrt{2}$（等价于纯 ReLU）。仅当配合非零斜率 $\alpha$ 的 Leaky ReLU 时，才需要显式传入参数 `a`。

# 残差网络与 Transformer 的深层初始化

Xavier 与 He 初始化假定信号沿层依次串行变换；而在 [[Residual-Networks|残差网络]] 与 Pre-Norm Transformer 中，每个残差子层将分支输出直接加回残差主干。记 $l$ 为子层编号，$\boldsymbol{x}_l$ 为第 $l$ 个子层的主干输出，$\mathcal{F}_l$ 为第 $l$ 个残差分支，$\boldsymbol{x}_0$ 为主干入口输入。若总计 $M$ 个残差子层且各分支近似不相关、单分支输出方差为 $\sigma^2$，则由 $\boldsymbol{x}_l = \boldsymbol{x}_{l-1} + \mathcal{F}_l(\boldsymbol{x}_{l-1})$ 累加后主干方差随深度线性增长：

$$
\operatorname{Var}(\boldsymbol{x}_M) \approx \operatorname{Var}(\boldsymbol{x}_0) + M\sigma^2
$$

在数十至上百层的模型中，这一累加会导致激活尺度膨胀与训练初期梯度失稳，现代深层网络常用两类修正：

1. **残差分支末层零初始化（Zero-Init Residual）**：在 ResNet 中将残差块最后一个归一化层的缩放参数初始化为 $\gamma = 0$，使第 0 步所有残差分支输出严格为零、网络初始等效恒等映射，随后再随梯度逐步开启各分支。
2. **按深度缩放输出投影（Depth-Scaled Init）**：在含 $L$ 个编码器或解码器块（每块包含注意力与前馈网络两个残差子层，共 $M = 2L$ 个分支）的 Transformer（如 GPT-2、LLaMA）中，将每个子层汇入主干前最后一个投影矩阵（注意力输出投影 $\mathbf{W}_O$ 与前馈降维矩阵 $\mathbf{W}_2$ 或 $\mathbf{W}_3$）的初始化标准差额外乘以 $\frac{1}{\sqrt{2L}}$，使每层方差贡献降至 $O(1/(2L))$，从而令主干总方差维持在 $O(1)$ 量级。

# 选型

下表汇总常见激活函数与深层结构的默认初始化方案与权重尺度；引入归一化层可吸收部分尺度偏差，但不能替代合理初始化：

| 激活函数 / 结构位置 | 常用初始化 | 权重方差 / 缩放 | 适用依据 |
| --- | --- | --- | --- |
| ReLU | He / Kaiming | $2/\text{fan\_in}$ | 补偿负半轴被截断导致的二阶矩减半 |
| Leaky ReLU（斜率 $\alpha$） | He 变体 | $2/\left((1+\alpha^2)\,\text{fan\_in}\right)$ | 负半轴以斜率 $\alpha$ 保留二阶矩贡献 |
| tanh、sigmoid | Xavier / Glorot | $2/(\text{fan\_in}+\text{fan\_out})$ | 假设工作在零点附近线性区，兼顾前向与反向 |
| 恒等映射（无激活） | 方差守恒 | $1/\text{fan\_in}$ | 线性叠加的直接期望保方差解 |
| 深层残差输出投影 | 按深度缩放 | 标准差乘 $1/\sqrt{2L}$ | 抑制 $2L$ 个残差分支累加引起的方差线性膨胀 |
