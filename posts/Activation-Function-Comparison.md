---
title: Activation Function Comparison
tags:
  - DeepLearning
categories:
  - Deep Learning
date: 2026-10-26 23:42:25
katex: true
---

# 激活函数的作用

仿射层的输出称为预激活，激活函数逐元素把预激活映射为新的输出。以下讨论仿射变换与逐点激活交替堆叠的前馈网络。在这类网络里，非线性由激活函数提供。其他结构也能提供非线性，例如最大池化对窗口取最大值、归一化中依赖输入的统计量，以及注意力中的 softmax 加权。

单个单元先做线性组合，再过一次非线性：

$$
z = \sum_i w_i x_i + b, \qquad a = \sigma(z)
$$

如果去掉第二步，多层网络退化成一次仿射变换：

$$
\boldsymbol{y} = \mathbf{W}_3\left(\mathbf{W}_2\left(\mathbf{W}_1\boldsymbol{x} + \boldsymbol{b}_1\right) + \boldsymbol{b}_2\right) + \boldsymbol{b}_3 = (\mathbf{W}_3\mathbf{W}_2\mathbf{W}_1)\,\boldsymbol{x} + \left(\mathbf{W}_3\mathbf{W}_2\boldsymbol{b}_1 + \mathbf{W}_3\boldsymbol{b}_2 + \boldsymbol{b}_3\right)
$$

括号展开后仍然是 $\mathbf{W}'\boldsymbol{x} + \boldsymbol{b}'$，堆叠不增加非线性表达能力。

# 仿射变换与激活函数

既然需要非线性，为什么不把线性那一步直接定义成非线性，例如标量映射 $x \mapsto Wx^2 + b$，省掉激活函数？

一是数值尺度。取单变量、单位权重、零偏置，并让每层采用平方映射 $x \mapsto x^2$。叠加 $L$ 层后得到 $x^{2^L}$：三层是 $x^8$，十层是 $x^{1024}$。$|x|>1$ 时迅速溢出，$|x|<1$ 时迅速衰减。这个标量例子说明，每层都做幂次映射会让尺度控制变难。

[html-card height=820](../assets/activation-scale-growth.html)

ReLU（Rectified Linear Unit，修正线性单元）定义为 $\max(0,x)$。ReLU 不做幂次映射，但正半轴无上界，深层网络叠加大权重时，激活值仍可能逐层放大。

二是工程实现。矩阵乘有成熟的库与内核。对 $z = \boldsymbol{w}^{\top}\boldsymbol{x} + b$，有 $\partial z/\partial \boldsymbol{w} = \boldsymbol{x}$；对单个样本，$\mathrm{d}L/\mathrm{d}\mathbf{W} = \delta\boldsymbol{x}^{\top}$，其中 $\delta = \partial L/\partial z$，一次外积即可给出该样本的权重梯度。整层计算时把各样本的外积相加。

自定义的非线性只要能由框架已有的算子组合得到，就同样可以借助自动微分求导。框架没有提供的新算子需要自行实现反向，写出该算子在自身定义下的导数。

# 从激活导数到梯度传播

激活函数既决定前向取值，也影响反向梯度。一层输出对输入的偏导数组成雅可比矩阵，其中同时包含权重与激活导数的作用。

反向传播需要连乘各层的雅可比矩阵，所以梯度的幅度与方向由整体连乘决定。权重的放大作用可能抵消激活导数的缩小作用，仅凭激活导数长期小于 1，不能断定完整梯度逐层衰减。

# 评估激活函数的六项指标

评估一个激活函数通常看下面六项。这六项是取舍指标，不是硬性门槛，没有哪个函数同时满足全部。

1. 非线性：不具备非线性，堆叠深度就没有意义。
2. 几乎处处可导：反向传播要乘该激活函数的导数。ReLU 在 0 处不可导，可约定该点的导数为 0；详见下文 ReLU 一节。阶跃函数的问题更严重：除 0 点外导数恒为 0，因此难以训练。
3. 梯度行为良好：导数不消失、不爆炸，理想情况接近 1。
4. 计算开销小：训练中会被调用上亿次，单次开销与具体实现和硬件有关。
5. 输出零中心：输出均值为 0。这样可以缓解同一层所有输入同号时各权重梯度只能同向、更新方向受限的问题；ReLU 不满足这一项，这是它的代价之一。
6. 单调、平滑、有界：常见的加分项，不是必需项。

# 逐点激活函数族

## 阶跃函数


阶跃函数是最简单的阈值型激活函数：

$$
\operatorname{step}(x) = \begin{cases} 1, & x \ge 0 \\ 0, & x < 0 \end{cases}
$$

阶跃函数在 $x \ne 0$ 处导数恒为 0，在 $x = 0$ 处不可导。多层网络使用阶跃函数时，误差在反向乘上 0，参数得到的梯度为零。

另一个问题是输出只有两种取值：输入是 0.001 还是 1000 都输出 1，数值上的差别被丢掉。

![阶跃函数的值与导数：0 处跳变，导数在 0 处未定义，其余位置为 0|390](../assets/activation-step.svg)


## Sigmoid

$$
\sigma(x) = \frac{1}{1 + e^{-x}}, \qquad \sigma'(x) = \sigma(x)\left(1 - \sigma(x)\right)
$$

sigmoid 在整个实数轴上处处可导，反向传播要乘的导数处处存在，它因此成为常用的激活函数。它的输出落在 $(0,1)$ 且 $\sigma(0) = 0.5$，适合做二分类的概率输出；过渡平滑，输入 1.5 与 1.6 的输出有可分辨的差别。

> [!important] 饱和
>
> 饱和指输入绝对值很大时，导数趋近 0。sigmoid 的导数在 $x = 0$ 处取最大值 $0.25$，$|x|$ 增大后迅速趋零，函数值趋近 1 或 0。
>
> 反向传播每穿过一层都要乘一次该层激活的导数（连乘过程见 [[Backpropagation-Chain-Rule]]），五层连乘的上界是 $0.25^5 \approx 9.8 \times 10^{-4}$。这个上界只含激活导数因子，不含权重因子；完整梯度还要乘上各层权重，所以它给出的是量级上的衰减趋势，不是精确的梯度范数。

> [!note] 非零中心的适用范围
>
> 记 $\delta = \partial L/\partial z$ 为该神经元预激活处的梯度，则 $\partial L/\partial w_j = \delta\, a_j^{(\text{prev})}$。如果上一层输出全为正，同一个神经元的所有权重梯度分量都与 $\delta$ 同号，这一次更新只能整体增大或整体减小。
>
> 这个结论适用于单个样本、单个神经元、上一层输出全为正的情形。整个训练过程中梯度是否同号，需要逐次计算才能判断，不能直接套用这里的结论。

![sigmoid 与导数：导数在 0 处取最大值 0.25|390](../assets/activation-sigmoid.svg)


sigmoid 与 tanh 都涉及指数运算。`exp` 的开销高于比较与加法，具体差值取决于硬件与内核，没有固定的数量级结论。

## Tanh

$$
\tanh(x) = \frac{e^{x} - e^{-x}}{e^{x} + e^{-x}} = 2\sigma(2x) - 1, \qquad \tanh'(x) = 1 - \tanh^2(x)
$$

tanh 的值域从 $(0,1)$ 扩展到 $(-1,1)$，关于原点对称。导数最大值从 $0.25$ 提到 $1$，但 $1$ 只在 $x = 0$ 处取到，是上界而不是典型值。

$x = 1$ 时 $\tanh'(1) = 1 - \tanh^2(1) \approx 0.42$，两层 tanh 的导数连乘约为 $0.18$。该例中的激活导数乘积仍会衰减，但大于两层 sigmoid 导数乘积的上界。

与 sigmoid 相比，tanh 的值域以 0 为中心，原点附近的导数更大；两端仍会饱和，计算也仍涉及指数运算。

$\tanh$ 是奇函数，输入分布关于 0 对称时，输出分布也关于 0 对称。输入分布不对称时，输出均值可能不为 0，不能仅凭值域就认定输出零均值。

![tanh 与导数：导数在 0 处为 1，在 x = 1 处约 0.42|390](../assets/activation-tanh.svg)


## ReLU


$$
\operatorname{ReLU}(x) = \max(0, x), \qquad \operatorname{ReLU}'(x) = \begin{cases} 1, & x > 0 \\ 0, & x < 0 \end{cases}
$$

特殊地，人为定义该函数在 0 处的导数为 0


相对 sigmoid 与 tanh, ReLU 的优势是：
1. 正半轴导数恒为 1，不饱和，梯度可以连续回传；
2. 只需一次比较，计算开销小；
3. 负半轴输出 0，激活结果稀疏，即一部分单元的输出恰为 0。

但 ReLU 的截断特征也带来了问题：

1. 负半轴导数为 0。只要某个单元的预激活在参与训练的所有样本上都是负的，该单元收到的数据梯度就一直是 0。这种状态仍有恢复的可能：上游参数或优化器状态改变后，预激活可能重新变正，该单元的梯度随之恢复。
2. 输出非负、不是零中心。正半轴梯度直通时，由此带来的更新方向受限通常可以接受。
3. 正半轴无上界。大权重相乘可能会导致梯度爆炸。

![ReLU 与导数：0 处按约定取导数 0|390](../assets/activation-relu.svg)



## ReLU 的变体

Leaky ReLU 在负半轴给一个很小的斜率：

$$
\operatorname{LeakyReLU}(x) = \begin{cases} x, & x > 0 \\ \alpha x, & x \le 0 \end{cases}
$$

负半轴导数恒为 $\alpha$（常见取值 0.01 到 0.1），即便预激活为负，梯度也不是 0。Leaky ReLU 避免了负半轴局部导数为零造成的梯度阻断，但完整梯度仍取决于上游梯度与其他层。

$\alpha$ 是 Leaky ReLU 引入的新超参数，它的取值决定初始化所需的尺度。预激活分布关于 0 对称时，负半轴斜率使预激活的二阶矩变成原来的 $(1+\alpha^2)/2$ 倍。对应的 He 初始化给出权重方差 $2/\left((1+\alpha^2)\,\text{fan\_in}\right)$，其中 fan_in 指每个单元接收的输入个数。

![Leaky ReLU（α = 0.01）与导数：负半轴斜率很小|390](../assets/activation-leaky-relu.svg)


GELU（Gaussian Error Linear Unit，高斯误差线性单元）用标准正态分布的累积分布函数给输入加权。设 $Z \sim \mathcal{N}(0,1)$，$\Phi(x) = P(Z \le x)$，即标准正态变量不超过 $x$ 的概率：

$$
\operatorname{GELU}(x) = x\,\Phi(x) \approx 0.5x\left(1 + \tanh\left(\sqrt{\tfrac{2}{\pi}}\left(x + 0.044715x^3\right)\right)\right)
$$

这里的 $\sqrt{2/\pi}$ 是一个预先算好的常数，不是运行时的开方开销。GELU 在 0 附近平滑过渡，负半轴保留很小的负值，$x = 1$ 时输出约 0.841。

Transformer 前馈层常用这一激活（[GELU 论文](https://arxiv.org/abs/1606.08415)）。并非所有 Transformer 都如此，也有模型选择别的激活。

![GELU 的 erf 精确值与 tanh 近似，以及精确导数|390](../assets/activation-gelu.svg)


SiLU（Sigmoid Linear Unit，也称 Swish）形式更简单：

$$
\operatorname{SiLU}(x) = x \cdot \sigma(x)
$$

SiLU 只需一个 sigmoid，形状与 GELU 接近。部分轻量级卷积神经网络采用 Swish 的分段线性近似 hard-swish（[MobileNetV3 论文](https://arxiv.org/abs/1905.02244)），hard-swish 与 Swish 并不等同。

![SiLU 与导数：函数在 x = -1.28 附近取最小值约 -0.28|390](../assets/activation-silu.svg)


大模型前馈层常用门控版本 SwiGLU。门控指两组线性变换的结果逐元素相乘，其中一组经过 SiLU；公式为 $\operatorname{SwiGLU}(\boldsymbol{x}) = \operatorname{SiLU}(\mathbf{W}_1\boldsymbol{x}) \odot \mathbf{W}_2\boldsymbol{x}$，$\odot$ 表示逐元素相乘。这一结构出现在 LLaMA、Mistral、Qwen 与 PaLM 等模型中。

[html-card height=820](../assets/swiglu-gate.html)

# 按位置选型

| 位置 | 常见选择 | 原因 |
| --- | --- | --- |
| 二分类输出层 | sigmoid，配二元交叉熵 | 输出可直接解释为正类概率 |
| 多分类输出层 | softmax，配交叉熵 | 输出归一化为类别分布 |
| 无约束回归输出层 | 不加激活 | 需要无界实数输出 |
| 卷积网络隐藏层 | ReLU 与各类变体 | 计算开销小，正半轴不饱和 |
| Transformer 前馈层 | GELU 或 SiLU 系 | 平滑，负半轴保留少量信息 |

输出层与损失函数必须配对：分类时把 softmax 或 sigmoid 写在损失函数内部，比先归一化再取对数稳定得多。

激活函数的选择没有全局最优，判断依据是上表的位置、任务形式，以及训练中实际观察到的梯度与饱和现象。
