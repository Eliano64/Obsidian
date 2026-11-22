---
title: Residual Networks
tags:
  - CNN
categories:
  - Artificial Intelligence
date: 2026-10-27 16:37:40
katex: true
---

# 深度带来的优化困难

深度神经网络在堆叠更多卷积层时，训练误差有时反而高于较浅的网络。这并非过拟合（测试集才出现过拟合），而是优化器无法有效寻优导致的**退化（Degradation）**现象（He et al., 2016）：理论上新增层只要学成恒等映射，深网训练集表现至少应持平浅网；解明明存在，梯度优化却难以直接逼近。

[html-card height=820](../assets/resnet-degradation-curve.html)

# 残差写法与直接路径

**残差网络（ResNet）** 不再要求多层直接拟合目标映射 $\mathcal{H}(\boldsymbol{x})$（$\boldsymbol{x}$ 为块输入张量），而是将目标拆解为「输入恒等直连 + 残差分支 $\mathcal{F}(\boldsymbol{x}) = \mathcal{H}(\boldsymbol{x}) - \boldsymbol{x}$」。经典 Post-activation 残差块在两路逐元素相加得到 $\boldsymbol{z}$ 后接 $\operatorname{ReLU}$ 得输出 $\boldsymbol{y}$：

$$
\boldsymbol{z} = \boldsymbol{x} + \mathcal{F}(\boldsymbol{x}), \qquad \boldsymbol{y} = \operatorname{ReLU}(\boldsymbol{z})
$$

若某一层期望接近恒等映射，优化器只需把残差分支 $\mathcal{F}(\boldsymbol{x})$ 的权重压向 $0$，远比令多层非线性精确输出 $\boldsymbol{x}$ 更容易实现。

> [!note] ReLU 与恒等映射
> 当 $\mathcal{F}(\boldsymbol{x}) = \mathbf{0}$ 时，$\boldsymbol{z} = \boldsymbol{x}$；经典块末端的 $\operatorname{ReLU}$ 会把负分量归零，因此严格恒等映射仅在 $\boldsymbol{x}$ 非负时成立。

[html-card height=820](../assets/resnet-residual-block.html)

## 捷径连接的直接梯度路径

设损失为 $L$，记 $\mathbf{I}$ 为单位矩阵、$\frac{\partial \mathcal{F}}{\partial \boldsymbol{x}}$ 为分支输出的雅可比矩阵，则反向传播经过相加节点 $\boldsymbol{z} = \boldsymbol{x} + \mathcal{F}(\boldsymbol{x})$ 得：

$$
\frac{\partial L}{\partial \boldsymbol{x}} = \frac{\partial L}{\partial \boldsymbol{z}}\left(\mathbf{I}+\frac{\partial \mathcal{F}}{\partial \boldsymbol{x}}\right)
$$

单位矩阵 $\mathbf{I}$ 为梯度开辟了一条不经 $\mathcal{F}$ 内部连续权重乘法的直接回传通路。

在经典 Post-activation 块中，主干上的 $\operatorname{ReLU}$ 仍可能截断负值路径。后续**预激活残差结构（Pre-activation ResNet）**将归一化与激活移入分支内部（`BN → ReLU → Conv`），使相邻第 $l$ 块与第 $l+1$ 块的主干表示满足纯粹累加 $\boldsymbol{x}_{l+1} = \boldsymbol{x}_l + \mathcal{F}_l(\boldsymbol{x}_l)$。从任意浅层 $l$ 展开至深层 $L$ 得：

$$
\boldsymbol{x}_L = \boldsymbol{x}_l + \sum_{i=l}^{L-1}\mathcal{F}_i(\boldsymbol{x}_i) \quad\Longrightarrow\quad \frac{\partial L}{\partial \boldsymbol{x}_l} = \frac{\partial L}{\partial \boldsymbol{x}_L}\left(\mathbf{I} + \frac{\partial}{\partial \boldsymbol{x}_l}\sum_{i=l}^{L-1}\mathcal{F}_i(\boldsymbol{x}_i)\right)
$$

无论跨越多少层，梯度回传主干都保有一个不受深层连乘缩放衰减的恒等项 $\mathbf{I}$。这种「主干纯粹累加、归一化置于子层分支前」的拓扑，直接演变为现代 Transformer 中的 **Pre-Norm** 结构（见 [[Norm]] 与 [[Encoder-Workflow]]）。

# 残差块结构与形状匹配

## 形状匹配与投影捷径

残差相加要求捷径分支 $S(\boldsymbol{x})$ 与学习分支 $\mathcal{F}(\boldsymbol{x})$ 严格同形。记输入与输出特征图形状分别为 $C\times H\times W$（通道、高、宽）与 $C'\times H'\times W'$：形状完全相同时直接恒等传递 $S(\boldsymbol{x}) = \boldsymbol{x}$（零额外参数）；当阶段切换导致通道翻倍或空间降采样（如 $64\times 32\times 32 \to 128\times 16\times 16$）时，捷径采用步幅为 2 的 $1\times 1$ 卷积 $S(\boldsymbol{x})$ 同步调整通道与高宽：

```text
f = F(x)
s = S(x)          # 同形状时恒等返回 x，变形状时用步幅 1x1 卷积对齐
z = s + f
return ReLU(z)
```

## 基础块与瓶颈块

随着网络深化，残差分支 $\mathcal{F}$ 的内部结构分出两种主流设计（均充分利用 [[CNN-Architecture#1×1 卷积与深度可分离卷积|1×1 卷积]] 压缩高通道计算代价）：

| 块类型 | 典型代表网络 | 分支内部串联结构 | 设计目的 |
| --- | --- | --- | --- |
| 基础块（BasicBlock） | ResNet-18、ResNet-34 | $3\times 3 \to 3\times 3$ 两层卷积 | 通道数较小时结构紧凑、实现简单 |
| 瓶颈块（Bottleneck） | ResNet-50、101、152 | $1\times 1$ 降维（至 $1/4$）$\to 3\times 3 \to 1\times 1$ 升维 | 在低维空间执行昂贵 $3\times 3$ 卷积，大幅削减百层高通道参数量 |

# 整体结构与全局平均池化

通用残差网络依次由浅层特征提取（如 $7\times 7$ 卷积加池化）、串联多组残差块、全局平均池化与线性分类头构成。记末层特征图三维张量为 $\mathbf{X}\in\mathbb{R}^{C\times H\times W}$（第 $c$ 通道坐标 $(i,j)$ 处元素为 $x_{c,i,j}$），全局平均池化（GAP）逐通道对空间平面取平均压成 $C$ 维向量 $\boldsymbol{g}\in\mathbb{R}^{C}$：

$$
g_c=\frac{1}{HW}\sum_{i=1}^{H}\sum_{j=1}^{W}x_{c,i,j}
$$

随后仅需一层 $C\times K$ 全连接映射（$K$ 为类别数）配 Softmax 输出类别概率，参数量远低于展平方案且不绑定输入分辨率。

> [!note] 优化与泛化的边界
> 残差连接解决的是深层网络「优化退化」问题；让深网更容易训练收敛不自动等价于更好的泛化，过深模型在小数据集上仍可能过拟合。
