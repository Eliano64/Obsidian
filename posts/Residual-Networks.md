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

增加卷积层能提高表达能力，训练误差却可能随深度上升。深层普通网络在训练集上的误差高于浅层网络，说明问题出在优化，而非泛化。原论文把这种现象称为“退化”。

深层网络并不缺少解。新增层若都实现恒等映射，其余层复制浅层权重，深网络的训练误差至多与浅层持平。解存在，优化器未必能找到，残差连接改变的是寻找这个解的方式。

[html-card height=820](../assets/resnet-degradation-curve.html)

> [!cite] He et al. 2016, Deep Residual Learning for Image Recognition, §3.1-3.3
> 退化现象、残差写法与形状匹配的讨论来源。

# 残差写法与直接路径

设若干堆叠层原本要拟合的映射为 $\mathcal{H}(\boldsymbol{x})$。残差写法让这些层只拟合差值 $\mathcal{F}(\boldsymbol{x})=\mathcal{H}(\boldsymbol{x})-\boldsymbol{x}$，本示例在相加后接 ReLU，前向计算为：

$$
z = x + \mathcal{F}(x), \qquad y = \operatorname{ReLU}(z)
\boldsymbol{z} = \boldsymbol{x} + \mathcal{F}(\boldsymbol{x}), \qquad \boldsymbol{y} = \operatorname{ReLU}(\boldsymbol{z})

[html-card height=820](../assets/resnet-residual-block.html)

前向拓扑是：学习分支输出 $\mathcal{F}(\boldsymbol{x})$，恒等捷径直接传递输入，两支相加得到 $\boldsymbol{z}$，再经 ReLU 得到 $\boldsymbol{y}$。$\mathcal{F}$ 的内部层数与算子类型都可变化。

$\boldsymbol{x}$ 与 $\mathcal{F}(\boldsymbol{x})$ 形状相同时，相加按元素进行。若最优映射接近恒等，让 $\mathcal{F}$ 接近 0 比让多层非线性精确输出 $\boldsymbol{x}$ 更容易优化：参数只需描述相对输入的修正量。这是参数化上的动机，不保证任何设置下都更易训练。

> [!note] ReLU 与恒等
> $\mathcal{F}=0$ 时相加结果为 $\boldsymbol{x}$。随后的 ReLU 会截掉负分量，因此这个块只在输入非负时等于恒等映射。

## 捷径连接的直接梯度路径

设 $L$ 为损失，$\boldsymbol{x}$ 为块输入，$\boldsymbol{z}$ 为相加结果。以标量为例，$\boldsymbol{z}=\boldsymbol{x}+\mathcal{F}(\boldsymbol{x})$ 时 $\mathrm{d}\boldsymbol{z}/\mathrm{d}\boldsymbol{x}=1+\mathcal{F}'(\boldsymbol{x})$；推广到特征图，$1$ 对应单位矩阵 $\mathbf{I}$，$\mathcal{F}'$ 对应 $\mathcal{F}$ 对输入的雅可比矩阵：

$$

\frac{\partial L}{\partial x} = \frac{\partial L}{\partial z}\left(I+\frac{\partial \mathcal{F}}{\partial x}\right)
\frac{\partial L}{\partial \boldsymbol{x}} = \frac{\partial L}{\partial \boldsymbol{z}}\left(\mathbf{I}+\frac{\partial \mathcal{F}}{\partial \boldsymbol{x}}\right)

括号中的恒等项让梯度有一条不经过 $\mathcal{F}$ 中非线性层的通路。这条通路提供直接路径，不保证梯度在任何深度下都不消失。

# 残差块与形状匹配

## 同形状与形状改变

残差块的学习分支改动特征，捷径把输入送到相加处，两支必须在相加前形状一致。形状相同时捷径直接取 $\boldsymbol{x}$，没有参数，也不增加计算量；形状不同时，捷径要做一次变换 $S(\boldsymbol{x})$，把 $\boldsymbol{x}$ 映射到学习分支输出的形状。通道数、高、宽分别记为 $C$、$H$、$W$，对应的输出通道数、高、宽记为 $C'$、$H'$、$W'$。

[html-card height=820](../assets/resnet-shape-matching.html)

形状相同时捷径是恒等映射，形状改变时要做一次变换 $S(\boldsymbol{x})$。输入为 $C \times H \times W$、学习分支输出对应为 $C' \times H' \times W'$ 时，两支都以 $C' \times H' \times W'$ 进入加法节点。$64 \times 32 \times 32 \to 128 \times 16 \times 16$ 只说明形状如何对应。

变换 $S(\boldsymbol{x})$ 通常做两件事：用 $1 \times 1$ 卷积调整通道数，用步幅大于 1 的卷积或池化把空间尺寸降到学习分支的水平。

## 残差块的前向计算

```text
f = F(x)
s = S(x)
z = s + f
return ReLU(z)
```

$S$ 是捷径：形状相同时 $S(\boldsymbol{x})=\boldsymbol{x}$，形状不同时做形状变换。

# 整体网络与分类头

## 通用残差网络的结构

残差块可以逐块串联。网络先做特征提取，再串起 $B_1, B_2, \dots, B_N$ 这一串残差块，每个块都带一条捷径，块与块之间只传递特征图。

[html-card height=820](../assets/resnet-generic-network.html)

整体依次是特征提取、重复的残差块、池化与结果向量。省略号表示重复同构的块，块数 $N$ 不是固定值。

## 全局平均池化

最后一个块输出的特征图记为 $\mathbf{x}$，形状为 $C \times H \times W$。记通道下标为 $c$，空间位置为 $i$、$j$，逐通道取平均得到 $C$ 维向量：

$$
g_c=\frac{1}{HW}\sum_{i=1}^{H}\sum_{j=1}^{W}x_{c,i,j}
$$

再经任务输出层（例如 softmax 分类）给出结果。逐通道平均去掉位置信息，分类依据是各通道的总体响应；池化还让输出层的输入维数只依赖通道数。

> [!note] 适用范围
> 残差连接缓解的是深层网络的优化困难。更易优化不等于更易泛化，深度增加时泛化表现仍需实验验证。
