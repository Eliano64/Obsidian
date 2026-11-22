---
title: From Perceptron To MLP
tags:
  - DeepLearning
categories:
  - Artificial Intelligence
date: 2026-10-26 23:42:25
katex: true
---

# 感知机

## 感知机的结构

**感知机（Perceptron）** 是最基础的线性二分类单元：对输入特征做加权求和加偏置得到预激活 $z$，再经阶跃阈值函数输出 $0$ 或 $1$。设输入维数为 $n$，输入特征向量为 $\boldsymbol{x}=(x_1,\dots,x_n)^{\top}$，权重向量为 $\boldsymbol{w}=(w_1,\dots,w_n)^{\top}$，偏置为 $b$，输出为 $a$：

$$
z = \sum_{i=1}^{n} w_i x_i + b = \boldsymbol{w}^{\top}\boldsymbol{x} + b, \qquad a = \operatorname{step}(z) = \begin{cases} 1, & z \ge 0 \\ 0, & z < 0 \end{cases}
$$

方程 $\boldsymbol{w}^{\top}\boldsymbol{x} + b = 0$ 在输入空间定义了一个**超平面**（二维平面下即一条直线），将空间切为 $z \ge 0$ 与 $z < 0$ 两个半空间，决定样本归类。

## 学习规则与收敛条件

感知机在预测正确时不改动参数，仅当样本分类错误时沿修正方向推挪超平面。记样本真实标签为 $y \in \{0, 1\}$、感知机预测为 $\hat{y} \in \{0, 1\}$、固定学习率为 $\eta > 0$，更新规则写为：

$$
\boldsymbol{w} \leftarrow \boldsymbol{w} + \eta\,(y - \hat{y})\,\boldsymbol{x}, \qquad b \leftarrow b + \eta\,(y - \hat{y})
$$

若真实为 1 而误判为 0，更新后内积 $\boldsymbol{w}^{\top}\boldsymbol{x}$ 增加 $\eta \lVert \boldsymbol{x} \rVert^2$，使该样本更偏向正侧。例如：取 $\eta=0.5$、初值 $\boldsymbol{w}=(0,0)^{\top},\, b=0$，样本 $\boldsymbol{x}=(1,1)^{\top},\, y=1$ 判为 1 正确故不更新；再遇 $\boldsymbol{x}=(2,-1)^{\top},\, y=0$ 却判为 1 出错，更新得 $\boldsymbol{w}=(-1, 0.5)^{\top},\, b=-0.5$，新预激活 $z = -3 < 0$ 转为正确。

> [!note] 线性可分与收敛定理
> 若存在 $(\boldsymbol{w}, b)$ 使全部正类满足 $\boldsymbol{w}^{\top}\boldsymbol{x}+b > 0$ 且全部负类满足 $\boldsymbol{w}^{\top}\boldsymbol{x}+b < 0$（即所有样本严格落在超平面两侧、无一点落在边界上），称为**严格线性可分**。有限数据集在严格线性可分与正恒定学习率条件下反复遍历时，感知机误分类更新次数存在上界，必在有限步内停止；非线性可分时该规则永不收敛。

## 单层感知机的局限：XOR

异或（XOR）逻辑将二维平面上的四个顶点按对角线分为两类，真值表如下：

| $x_1$ | $x_2$ | 输出 |
| --- | --- | --- |
| 0 | 0 | 0 |
| 1 | 0 | 1 |
| 0 | 1 | 1 |
| 1 | 1 | 0 |

### 线性不可分的代数证明

把四点代入感知机分类规则（$z \ge 0$ 判为 1、$z < 0$ 判为 0），要求权重 $w_1, w_2$ 与偏置 $b$ 同时满足四个不等式：

$$
b < 0,\quad w_1 + b \ge 0,\quad w_2 + b \ge 0,\quad w_1 + w_2 + b < 0
$$

由前三式推得 $w_1 \ge -b$ 且 $w_2 \ge -b$，相加得 $w_1 + w_2 + b \ge -b > 0$，与第四式矛盾。几何上 $(1,0)$、$(0,1)$ 与 $(0,0)$、$(1,1)$ 各自占据两条相交对角线，单一直线无法把同色对角点划入同一侧，这就是单一超平面的表示边界。

![原始输入空间：两条直线都无法分对四点|602](../assets/dl-xor-space.svg)

# MLP

**多层感知机（Multi-Layer Perceptron, MLP）** 在输入与输出之间插入带有非线性激活的隐藏层，核心思路是**先做坐标变换、再做线性划分**：隐藏层将原空间非线性扭曲或投影为新坐标，使样本在新表示下变为线性可分，输出层只需再画一条超平面即可完成分类。

以求解 XOR 为例，构造含两个隐藏单元 $h_1$（逻辑“或”）与 $h_2$（逻辑“与非”）的隐藏层，输出层再执行逻辑“与”：

$$
h_1 = \operatorname{step}(x_1 + x_2 - 0.5), \qquad h_2 = \operatorname{step}(-x_1 - x_2 + 1.5), \qquad y = \operatorname{step}(h_1 + h_2 - 1.5)
$$

各顶点在新坐标 $(h_1, h_2)$ 下的映射如下表，两个负例 $(0,0)$ 与 $(1,1)$ 被区分至直线 $h_1 + h_2 = 1.5$ 的左下方，而两个正例重合在 $(1,1)$ 位于直线右上方：

| $(x_1, x_2)$ | $h_1$ | $h_2$ | $h_1 + h_2$ | $y$ |
| --- | --- | --- | --- | --- |
| (0, 0) | 0 | 1 | 1 | 0 |
| (1, 0) | 1 | 1 | 2 | 1 |
| (0, 1) | 1 | 1 | 2 | 1 |
| (1, 1) | 1 | 0 | 1 | 0 |

![隐藏层表示：四个样本与一条分界|602](../assets/dl-xor-hidden.svg)

从组合几何角度看，每个阈值隐藏单元定义一个半空间划分，输出层对其进行逻辑组合：「与」对应半空间交集构成凸区域，「或」对应并集构成非凸区域，因此多层单元叠加可拼出任意复杂的分段边界；深度越深，「变换表示、再划分」的嵌套层数越多，边界构造也越精细。

## 为什么不能只堆仿射层

若去掉隐藏层的非线性激活函数（如 $\operatorname{step}$ 或 ReLU），只串联多层仿射变换，设输入向量为 $\boldsymbol{x}$，三层权重矩阵与偏置向量分别为 $\mathbf{W}_1, \mathbf{W}_2, \mathbf{W}_3$ 与 $\boldsymbol{b}_1, \boldsymbol{b}_2, \boldsymbol{b}_3$，则最终输出 $\boldsymbol{y}$ 仍坍缩为一次仿射映射（其中等效权重 $\mathbf{W}' = \mathbf{W}_3\mathbf{W}_2\mathbf{W}_1$，等效偏置 $\boldsymbol{b}' = \mathbf{W}_3\mathbf{W}_2\boldsymbol{b}_1 + \mathbf{W}_3\boldsymbol{b}_2 + \boldsymbol{b}_3$）：

$$
\boldsymbol{y} = \mathbf{W}_3\left(\mathbf{W}_2\left(\mathbf{W}_1 \boldsymbol{x} + \boldsymbol{b}_1\right) + \boldsymbol{b}_2\right) + \boldsymbol{b}_3 = (\mathbf{W}_3\mathbf{W}_2\mathbf{W}_1)\,\boldsymbol{x} + (\mathbf{W}_3\mathbf{W}_2 \boldsymbol{b}_1 + \mathbf{W}_3 \boldsymbol{b}_2 + \boldsymbol{b}_3) = \mathbf{W}' \boldsymbol{x} + \boldsymbol{b}'
$$

无论串联多少层纯粹的线性层，本质仍只能画一个超平面，且映射矩阵的秩受限于全网最窄层的宽度。**深度的非线性表达能力完全源自层间激活函数，而不来自堆叠层数本身。**

# 表达能力与可训练性

**万能逼近定理**证明：在紧集上，一个足够宽的单隐藏层网络（激活函数采用两端极限为 0 与 1 的连续 sigmoidal 函数，输出层取线性组合）可以以任意精度一致逼近任意连续函数（[Cybenko](https://doi.org/10.1007/BF02551274)；多项式激活不在保证之列）。

然而万能逼近定理只给出**存在性**保证：面对图像纹理到部件再到物体这类层次化真实数据，单隐藏层所需的宽度可能呈指数级膨胀；**深层架构**通过逐层复用低阶特征表示，能以少几个数量级的参数实现同等表达力。

更关键的是，表达能力存在不等于梯度下降能成功找到这组参数、也不等于能泛化。深层网络的实际可训练性依赖四项机制协同支撑：[[Activation-Function-Comparison|非饱和激活函数]]、[[Weight-Initialization-And-Variance|方差保持初始化]]、[[Norm|归一化层]]与 [[Residual-Networks|残差连接]]，梯度计算与参数尺度推导见 [[Backpropagation-Chain-Rule]] 与 [[Weight-Initialization-And-Variance]]。
