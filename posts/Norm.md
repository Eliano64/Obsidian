---
title: Norm
tags:
  - DeepLearning
categories:
  - Artificial Intelligence
date: 2026-10-26 23:42:25
katex: true
---

# 归一化要解决的问题

权重初始化只能约束训练第 0 步的前向信号尺度，相关推导见 [[Weight-Initialization-And-Variance]]。训练开始后各层参数按梯度更新，后层接收到的特征分布持续漂移，初始尺度关系随之失效。

**归一化（Normalization）** 在前向传播中按固定维度统计均值、方差或均方根，将中间激活重新定标到可控范围。关于收益来源有两种互补解释：早期工作将其归因于抑制**内部协变量偏移（Internal Covariate Shift）**，后续对照实验则表明，即使人为注入分布偏移归一化依然有效，其核心机制更体现在平滑损失曲面与改善优化条件，从而允许使用更大学习率并加速收敛。

# 标准化与仿射变换

**标准化**指减去统计均值再除以标准差，随后通常接一次逐通道或逐维仿射变换恢复网络表达能力。记输入样本矩阵为 $[B, D]$（行对应 $B$ 个样本，列对应 $D$ 个特征维），对当前统计集合内任一元素 $x$，设该集合的均值与方差分别为 $\mu$ 与 $\sigma^2$，极小常数 $\epsilon$（如 $10^{-5}$）用于防止除零，可学习缩放与平移参数 $\gamma,\beta$（长度 $D$ 的向量）将标准化输出 $\hat{x}$ 映射为 $y$：

$$
\hat{x} = \frac{x - \mu}{\sqrt{\sigma^2 + \epsilon}}, \qquad y = \gamma\,\hat{x} + \beta
$$

注意 $\gamma = 1$、$\beta = 0$ 时输出停留在标准化后的尺度（而非原始输入），可学习的 $\gamma,\beta$ 允许网络在需要时重新拉开各特征的均值与方差。常用归一化方法的核心差别在于**由哪些元素共同参与计算统计量**。

# 常用归一化方法

## BatchNorm

**BatchNorm（批归一化，BN）** 沿 batch 维固定特征列，用同批 $B$ 个样本计算每个特征维的均值与方差（共 $D$ 组统计量）。记 $x_{ij}$ 为第 $i$ 个样本在第 $j$ 个特征上的取值，$\mu_j$ 与 $\sigma_j^2$ 为第 $j$ 列的均值与方差：

$$
\mu_j = \frac{1}{B}\sum_{i=1}^{B} x_{ij}, \qquad \sigma_j^2 = \frac{1}{B}\sum_{i=1}^{B}\left(x_{ij} - \mu_j\right)^2
$$

缩放与平移参数 $\gamma_j,\beta_j$ 按特征维分配，形状为 $[D]$。训练阶段用当前 batch 统计量执行标准化，同时按动量 $m$（PyTorch 默认 $0.1$）维护跨批次滑动估计：

$$
\text{running} \leftarrow (1 - m)\,\text{running} + m\,\text{batch\_stat}
$$

框架在维护 `running_var` 时采用无偏方差（分母 $B-1$），标准化当批数据则用有偏方差（分母 $B$）。推理阶段默认直接读取 `running` 估计（除非显式设置 `track_running_stats=False`），确保单个样本的输出完全独立于同批其他样本。

> [!important] batch 为 1 时 BN 能否运行
> 对二维输入 $[B, D]$ 且 $B=1$，每个通道仅有 1 个元素导致方差无意义，PyTorch 训练模式会直接报错。对四维卷积特征图 $[N, C, H, W]$ 且 $N=1$（$N$ 为样本数、$C$ 为通道数、$H$ 与 $W$ 为空间高宽），BN 对每通道跨 $N,H,W$ 统计；只要 $H\times W > 1$ 框架即可计算，但极小 batch 下统计方差噪声很大。

## LayerNorm

**LayerNorm（层归一化，LN）** 沿特征维固定样本行，用单个样本自身的全部 $D$ 个特征计算均值 $\mu_i$ 与方差 $\sigma_i^2$（共有 $B$ 组统计量）：

$$
\mu_i = \frac{1}{D}\sum_{j=1}^{D} x_{ij}, \qquad \sigma_i^2 = \frac{1}{D}\sum_{j=1}^{D}\left(x_{ij} - \mu_i\right)^2
$$

可学习参数 $\gamma_j,\beta_j$ 在样本间共享（形状 $[D]$）。LN 完全不依赖 batch 大小，训练与推理计算方式完全一致。对形状 $[B, T, D]$（$T$ 为序列长度）的序列张量，Transformer 沿最后一维 $D$ 对每个词元独立归一化，因此序列变长或尾部补齐（padding）不会污染有效位置的统计量；不过注意力和损失计算仍需显式掩码屏蔽填充位。

## RMSNorm

**RMSNorm（均方根层归一化）** 是 LLaMA、Qwen、Mistral、DeepSeek 等现代大语言模型的默认归一化层。研究表明，LN 对深层训练稳定性的贡献主要源自**特征尺度的重定标**而非减去均值的平移中心化。

RMSNorm 因此省略均值计算与可学习平移参数 $\beta$，直接按单样本向量 $\boldsymbol{x}_i \in \mathbb{R}^D$ 的均方根缩放：

$$
\operatorname{RMS}(\boldsymbol{x}_i) = \sqrt{\frac{1}{D}\sum_{j=1}^{D} x_{ij}^2 + \epsilon}, \qquad y_{ij} = \gamma_j \frac{x_{ij}}{\operatorname{RMS}(\boldsymbol{x}_i)}
$$

省去一趟均值归约与减法后，访存次数减少且仿射参数减半；在受显存带宽限制的 GPU 上，RMSNorm 可带来约 10% 至 30% 的归一化算子加速，并保持与标准 LN 相近的收敛表现。

## GroupNorm

目标检测、语义分割与扩散模型（Diffusion U-Net）常因高分辨率输入导致单卡只能容纳 $N=1$ 或 $2$ 的极小 batch，此时 BN 统计极不稳定。

**GroupNorm（组归一化）** 针对 $[N, C, H, W]$ 卷积特征图，将每个样本的 $C$ 个通道均分为 $G$ 组，在每组内的 $(C/G)\times H\times W$ 个元素上独立统计均值与方差。当 $G=1$ 时退化为整张三维特征图上的 LayerNorm，当 $G=C$ 时退化为单通道平面 $H\times W$ 上的 InstanceNorm，兼顾了 batch 无关性与通道结构划分。

# 数值例子

## 同一矩阵上的三种结果

取一个 4 样本、3 特征的小矩阵（行对应样本 1 至 4，列对应特征 1 至 3），固定考察**样本 2 的特征 2**（原值 `7`）在三种归一化下的变换差别：

| 样本 | 特征 1 | 特征 2 | 特征 3 |
| --- | --- | --- | --- |
| 1 | 1 | 5 | 2 |
| 2 | 4 | 7 | 3 |
| 3 | 2 | 6 | 9 |
| 4 | 8 | 9 | 4 |

### 格子 7 的计算对比

- **BatchNorm（沿列统计）**：特征 2 四个取值为 $[5, 7, 6, 9]$，均值 $6.75$、标准差 $1.479$，格子 `7` 标准化为 $(7-6.75)/1.479 \approx 0.169$。
- **LayerNorm（沿行统计）**：样本 2 三个特征为 $[4, 7, 3]$，均值 $4.667$、标准差 $1.700$，格子 `7` 标准化为 $(7-4.667)/1.700 \approx 1.373$。
- **RMSNorm（沿行不减均值）**：样本 2 均方根为 $\sqrt{(4^2+7^2+3^2)/3} \approx 4.967$，格子 `7` 缩放为 $7/4.967 \approx 1.409$。
- **叠加仿射参数**：取 $\gamma_2=1.5,\ \beta_2=-0.2$（RMSNorm 无 $\beta_2$），BN 输出为 $1.5\times 0.169-0.2=0.054$，LN 输出为 $1.860$，RMSNorm 输出为 $1.5\times 1.409\approx 2.114$。

下方交互卡片对比同一矩阵下 BN（沿列）与 LN（沿行）的统计集合与标准化输出：

[html-card height=500](../assets/dl-norm-statistics.html)

## BN 训练与推理的数值差别

仍看格子 `7`：训练时该列当批均值 $6.75$、标准差 $1.479$，标准化值为 $0.169$。若推理时累积的 running 均值为 $6.9$、running 方差为 $2.4$，则同一输入在推理阶段变为 $(7-6.9)/\sqrt{2.4+\epsilon}\approx 0.065$；因为统计量来源不同，BN 必须正确切换 `model.train()` 与 `model.eval()`。

# 四类归一化对比

对于四维卷积张量 $[N, C, H, W]$，BatchNorm 按通道跨 $N, H, W$ 产出 $C$ 组统计量，而其余三类均按样本独立统计、不随 $N$ 变化。下表汇总四类方法的统计范围、运行约束与典型场景：

| 项目 | BatchNorm | LayerNorm | RMSNorm | GroupNorm |
| --- | --- | --- | --- | --- |
| 统计范围 | 同特征跨样本（卷积跨 $N,H,W$） | 单样本内全部特征 | 单样本内全部特征（不减均值） | 单样本内每组 $(C/G)\times H\times W$ |
| 是否依赖 batch | 训练依赖，推理用 running 估计 | 不依赖 | 不依赖 | 不依赖 |
| 训练与推理行为 | 不同 | 相同 | 相同 | 相同 |
| 可学习参数 | $\gamma_j,\beta_j$（形状 $[D]$） | $\gamma_j,\beta_j$（形状 $[D]$） | 仅 $\gamma_j$（形状 $[D]$） | $\gamma_c,\beta_c$（形状 $[C]$） |
| 典型场景 | 大 batch 卷积与全连接网络 | 经典 Transformer、变长序列 | 现代大语言模型（LLaMA、Qwen） | 小 batch 视觉检测、分割与扩散模型 |

## Pre-Norm 与 Post-Norm

记第 $l$ 层残差输入为 $\boldsymbol{x}_l$，子层变换为 $F_l$，归一化算子为 $\operatorname{Norm}$：

- **Post-Norm**：$\boldsymbol{x}_{l+1} = \operatorname{Norm}(\boldsymbol{x}_l + F_l(\boldsymbol{x}_l))$。归一化位于主干串联路径上，反向梯度会随子层尺度逐层折减，深层训练通常依赖学习率预热。
- **Pre-Norm**：$\boldsymbol{x}_{l+1} = \boldsymbol{x}_l + F_l(\operatorname{Norm}(\boldsymbol{x}_l))$。主干保留恒等梯度直通更易训稳深层网络，但在离开末层后需补充一次最终归一化（详见 [[Encoder-Workflow#Pre-Norm|Pre-Norm]]）。

# 尺度不变性与偏置设计

归一化具有**群组内正线性变换不变性**：当同一统计组内所有输入变为 $x' = c x + d$（其中比例 $c > 0$，常数偏移 $d$ 在组内所有元素上相同，忽略 $\epsilon$）时，组均值与标准差同步变为 $c\mu + d$ 与 $c\sigma$，标准化结果 $\frac{(cx+d)-(c\mu+d)}{c\sigma}$ 与原本完全一致。这一性质直接引出两条工程规律：

1. **承受更大学习率，但不能替代初始化**：前层权重整体放大或缩小 $c$ 倍不会改变标准化输出（从而减少梯度暴涨风险），但入口输入、残差相加分支与末层输出仍未经过层内重定标，仍需配合合适初始化尺度。
2. **为什么 BN 前线性/卷积层常设 `bias=False`，而 LN / RMSNorm 前保留偏置**：BN 跨样本按同通道减均值，同通道上的线性偏置 $b_j$ 在该通道所有元素上为同一常数 $d$，减均值时被直接抵消而纯属冗余（由 BN 自带 $\beta_j$ 承担平移）；LN 沿特征维减均值，不同特征维的偏置 $b_j$ 互不相同（不能抵消为常数），RMSNorm 更是完全不减均值、甚至没有 $\beta_j$，因此前置层需保留偏置或由结构自身提供自由度。
