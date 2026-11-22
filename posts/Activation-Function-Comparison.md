---
title: Activation Function Comparison
tags:
  - DeepLearning
categories:
  - Artificial Intelligence
date: 2026-10-26 23:42:25
katex: true
---

# 激活函数

仿射层的线性输出称为预激活，**激活函数**把预激活映射为下一层的非线性输入。对单层神经元，设输入向量 $\boldsymbol{x}$ 各分量为 $x_i$，权重向量 $\boldsymbol{w}$ 对应分量为 $w_i$，偏置为 $b$，预激活 $z$ 与输出激活 $a$ 写为：

$$
z = \sum_i w_i x_i + b = \boldsymbol{w}^{\top}\boldsymbol{x} + b, \qquad a = \sigma(z)
$$

若去掉 $\sigma$，任意多层仿射变换相乘展开后仍然等价于单次仿射映射，堆叠深度不增加任何非线性表达能力（推导见 [[From-Perceptron-To-MLP#为什么不能只堆仿射层]]）。

神经网络不直接把线性映射改成多项式（如 $x \mapsto W x^2 + b$），而是采用「矩阵乘 + 逐点非线性」解耦：一方面连续幂次映射会使激活尺度呈指数失控（如平方映射连叠 $L$ 层得 $x^{2^L}$），另一方面矩阵乘法拥有成熟硬件内核、外积梯度形态极简单。反向传播沿路径连乘各层雅可比矩阵，其中既含权重缩放也含激活函数导数 $\sigma'(z)$，因此激活函数的导数形态直接影响训练稳定性。

# 评估激活函数的六项指标

选择激活函数通常在以下六项之间取舍，没有函数在所有维度同时占优：

1. **非线性**：赋予多层网络突破线性分隔边界的表达能力。
2. **几乎处处可导**：确保反向传播能沿链式法则计算梯度（允许在孤立零点按次梯度约定）。
3. **梯度行为良好**：在工作区间内导数不趋近于 0（防梯度消失），也不过度放大（防梯度爆炸）。
4. **计算开销低**：前向与反向均被高频调用，比较与加法明显轻于指数（`exp`）运算。
5. **输出零中心**：输出均值接近 0，可避免后层各权重梯度符号被迫锁定（详见下节 Sigmoid）。
6. **单调与平滑**：优化曲面更规整，但非必需（如 GELU 并非严格单调）。

# 逐点激活函数族

## 阶跃函数

**阶跃函数**（Step Function）按阈值 0 将预激活二值截断为 0 或 1：

$$
\operatorname{step}(x) = \begin{cases} 1, & x \ge 0 \\ 0, & x < 0 \end{cases}
$$

它在 $x = 0$ 处不可导，其余位置导数恒为 0，误差无法通过链式法则传回浅层，同时把 0.001 与 1000 的幅值差别彻底丢弃，只适用于早期单层感知机，不用于现代梯度训练。

![阶跃函数的值与导数：0 处跳变，导数在 0 处未定义，其余位置为 0|390](../assets/activation-step.svg)

## Sigmoid

**Sigmoid** 函数将实数平滑压入 $(0, 1)$ 区间，且 $\sigma(0) = 0.5$，导数具有自指简洁形式：

$$
\sigma(x) = \frac{1}{1 + e^{-x}}, \qquad \sigma'(x) = \sigma(x)\left(1 - \sigma(x)\right)
$$

因为值域严格对应概率区间，Sigmoid 适合放在二分类输出层；但作为隐藏层激活存在两项结构瓶颈。

> [!important] 梯度饱和与非零中心
> 1. **梯度饱和衰减**：$\sigma'(x)$ 在原点处取最大值 $0.25$，$|x|$ 增大后导数迅速趋零。深层连乘（见 [[Backpropagation-Chain-Rule]]）时五层激活导数乘积上界仅 $0.25^5 \approx 9.8 \times 10^{-4}$，极易引发梯度消失（最终梯度仍取决于权重缩放）。
> 2. **非零中心导致的锯齿更新**：设 $L$ 为损失、$z$ 为该神经元预激活、$w_j$ 为第 $j$ 个输入权重、$a_j^{(\text{prev})}$ 为上一层对应输出，令 $\delta = \partial L/\partial z$，则 $\partial L/\partial w_j = \delta\,a_j^{(\text{prev})}$。由于 Sigmoid 输出恒正（$a_j^{(\text{prev})} > 0$），同一样本上该神经元的所有权重分量只能全部与 $\delta$ 同号（一起增大或一起减小）。当最优更新方向需要一个权重增大、另一个减小时，参数无法直达，只能沿锯齿路径反复折返，拖慢收敛。

![sigmoid 与导数：导数在 0 处取最大值 0.25|390](../assets/activation-sigmoid.svg)

## Tanh

**Tanh（双曲正切）** 可视为 Sigmoid 平移缩放后的零中心版本，值域为 $(-1, 1)$：

$$
\tanh(x) = \frac{e^{x} - e^{-x}}{e^{x} + e^{-x}} = 2\sigma(2x) - 1, \qquad \tanh'(x) = 1 - \tanh^2(x)
$$

相比 Sigmoid，Tanh 关于原点对称（输入零对称时输出亦保持零中心），原点导数峰值提升至 $1$；但饱和与指数计算开销依然存在（$x=1$ 处导数已跌至约 $0.42$，两层连乘即约 $0.18$），常用于循环网络门控与经典浅层结构。

![tanh 与导数：导数在 0 处为 1，在 x = 1 处约 0.42|390](../assets/activation-tanh.svg)

## ReLU

**ReLU（修正线性单元）** 直接保留正半轴、将负半轴截断为 0；框架对 $x=0$ 处的左右不可导点通常按次梯度约定导数为 0：

$$
\operatorname{ReLU}(x) = \max(0, x), \qquad \operatorname{ReLU}'(x) = \begin{cases} 1, & x > 0 \\ 0, & x < 0 \end{cases}
$$

相比 Sigmoid 与 Tanh，ReLU 的工程取舍十分鲜明：

- **优势**：正半轴导数恒为 1、完全不饱和，深层信号更易回传；只需阈值比较无指数运算；负半轴输出 0 天然形成稀疏激活。
- **代价**：若某神经元在所有训练样本上预激活持续为负，其收到的数据梯度恒为 0（陷入“死神经元”，需依赖上游权重或学习率变化拉回正区才可恢复）；输出非负非零中心；正半轴无上界，需配合合理初始化（如 He 初始化）或归一化控制激活爆炸。

![ReLU 与导数：0 处按约定取导数 0|390](../assets/activation-relu.svg)

## 带小斜率与门控的变体

### Leaky ReLU

为修复 ReLU 负半轴导数归零，Leaky ReLU 在 $x \le 0$ 给予固定小斜率 $\alpha$（通常取 $0.01$ 至 $0.1$）：

$$
\operatorname{LeakyReLU}(x) = \begin{cases} x, & x > 0 \\ \alpha x, & x \le 0 \end{cases}
$$

负半轴恒保留非零斜率 $\alpha$，避免局部算子截断梯度；对称零均值预激活通过后二阶矩缩放倍数为 $(1+\alpha^2)/2$，对应权重初始化方差修正为 $2/((1+\alpha^2)\,\text{fan\_in})$（其中 $\text{fan\_in}$ 为单单元输入个数，见 [[Weight-Initialization-And-Variance]]）。

![Leaky ReLU（α = 0.01）与导数：负半轴斜率很小|390](../assets/activation-leaky-relu.svg)

### GELU 与 SiLU

Transformer 前馈层（FFN）普遍采用零点平滑且负半轴保留小幅非零导数的激活函数，以消除硬折点带来的优化抖动：

- **GELU（高斯误差线性单元）**：按标准正态分布 $Z \sim \mathcal{N}(0, 1)$ 的累积分布函数 $\Phi(x) = P(Z \le x)$ 对输入进行随机门控期望加权，常用高效 $\tanh$ 近似计算（其中 $\sqrt{2/\pi}$ 为预计算常数）：

$$
\operatorname{GELU}(x) = x\,\Phi(x) \approx 0.5x\left(1 + \tanh\left(\sqrt{\tfrac{2}{\pi}}\left(x + 0.044715x^3\right)\right)\right)
$$

![GELU 的 erf 精确值与 tanh 近似，以及精确导数|390](../assets/activation-gelu.svg)

- **SiLU（又称 Swish）**：直接用 Sigmoid 自身作门控权重，形状与 GELU 高度接近且算子更简洁；部分移动端卷积网络（如 MobileNetV3）采用其分段线性近似 `hard-swish` 消除指数计算：

$$
\operatorname{SiLU}(x) = x \cdot \sigma(x)
$$

![SiLU 与导数：函数在 x = -1.28 附近取最小值约 -0.28|390](../assets/activation-silu.svg)

### 门控激活：SwiGLU

不同于上述单输入逐点算子，现代大语言模型（PaLM、LLaMA、Qwen、Mistral 等）在前馈层普遍采用双分支门控块 **SwiGLU**。设 $d_{\text{model}}$ 为隐藏维数、$d_{\text{ff}}$ 为前馈中间维数，输入向量为 $\boldsymbol{x}\in\mathbb{R}^{d_{\text{model}}}$，两个升维投影矩阵为 $\mathbf{W}_1,\mathbf{W}_2\in\mathbb{R}^{d_{\text{ff}}\times d_{\text{model}}}$，降维输出矩阵为 $\mathbf{W}_3\in\mathbb{R}^{d_{\text{model}}\times d_{\text{ff}}}$（常省略偏置），$\odot$ 表示逐元素乘法，完整结构写为：

$$
\operatorname{FFN}_{\text{SwiGLU}}(\boldsymbol{x}) = \mathbf{W}_3\left(\operatorname{SiLU}(\mathbf{W}_1\boldsymbol{x}) \odot (\mathbf{W}_2\boldsymbol{x})\right)
$$

因为 SwiGLU 含三个投影矩阵，而经典两矩阵 FFN（中间维 $4d_{\text{model}}$）参数量为 $8d_{\text{model}}^2$，现代模型常将中间维缩小至经典宽度的 $\frac{2}{3}$，即取 $d_{\text{ff}} \approx \frac{8}{3}d_{\text{model}}$（并对齐到 128 或 256 等硬件友好倍数），在保持总参数与计算量近似为 $8d_{\text{model}}^2$ 的前提下显著改善语言建模效果。

[html-card height=820](../assets/swiglu-gate.html)

# 按位置选型

激活函数的选择取决于所在网络位置与损失配合方式，没有全局唯一最优；分类任务务必把 `Softmax` 或 `Sigmoid` 融合进交叉熵损失内部计算而非输出层手加：

| 位置 | 常见选择 | 核心原因 |
| --- | --- | --- |
| 二分类输出层 | logits 直传 `BCEWithLogitsLoss`（推理用 Sigmoid） | 内部融合避免对数与指数下溢，推理输出正类概率 |
| 多分类输出层 | logits 直传 `CrossEntropyLoss`（推理用 Softmax） | 内部先 `log_softmax` 保持数值稳定，推理给出分布 |
| 无约束回归输出层 | 不加激活 | 保留无界实数预测空间 |
| 卷积网络隐藏层 | ReLU 或 Leaky ReLU | 阈值比较计算快，正半轴不饱和 |
| Transformer 前馈层 | GELU、SiLU 或 SwiGLU | 原点附近平滑过渡，负半轴保留小梯度 |
