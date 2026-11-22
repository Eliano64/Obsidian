---
title: Backpropagation Chain Rule
tags:
  - DeepLearning
categories:
  - Artificial Intelligence
date: 2026-10-26 23:42:25
katex: true
---

# 问题规模

**反向传播（Backpropagation）** 沿计算图从标量损失出发逆序遍历，利用链式法则一次前向加一次反向即可算出网络中全部参数的梯度。设模型总参数数为 $P$（例如 20 万），若用数值差分逐参数扰动求导，前向差分需 $P+1$ 次前向、中心差分需 $2P$ 次前向；反向传播复用共享中间结果，计算量不随参数量呈平方级膨胀。

# 符号约定

设 $L$ 为标量损失，$l$ 为网络层序号，$\mathbf{Z}^{(l)}$、$\mathbf{A}^{(l)}$ 分别为第 $l$ 层的预激活矩阵与激活矩阵，$\mathbf{W}^{(l)}$、$\boldsymbol{b}^{(l)}$ 分别为该层的权重矩阵与偏置列向量，$\sigma$ 为逐元素激活函数。下文记梯度张量 $\mathrm{d}\mathbf{A}^{(l)} := \partial L/\partial \mathbf{A}^{(l)}$、$\mathrm{d}\mathbf{Z}^{(l)} := \partial L/\partial \mathbf{Z}^{(l)}$、$\mathrm{d}\mathbf{W}^{(l)} := \partial L/\partial \mathbf{W}^{(l)}$、$\mathrm{d}\boldsymbol{b}^{(l)} := \partial L/\partial \boldsymbol{b}^{(l)}$，每个梯度矩阵与对应原矩阵同形。

单层前向计算写作：

$$
\mathbf{Z}^{(l)} = \mathbf{W}^{(l)} \mathbf{A}^{(l-1)} + \boldsymbol{b}^{(l)}, \qquad \mathbf{A}^{(l)} = \sigma\left(\mathbf{Z}^{(l)}\right)
$$

设 $n_l$ 与 $n_{l-1}$ 分别为当前层与前一层的单元数，$B$ 为一个 batch 内的样本数，则形状为 $\mathbf{A}^{(l)} \in \mathbb{R}^{n_l \times B}$、$\mathbf{W}^{(l)} \in \mathbb{R}^{n_l \times n_{l-1}}$、$\boldsymbol{b}^{(l)} \in \mathbb{R}^{n_l \times 1}$（加法时沿 batch 维广播）。普通表格数据与 PyTorch 惯例采用「行是样本数、列是特征数」（$[B, D]$），而本篇推导采用列样本排布 $[n_l, B]$，两者互为转置、数学上完全等价。

# 链式法则与计算图

对一元复合函数 $y(u(x))$，链式法则把总导数拆成局部导数的乘积：

$$
\frac{\mathrm{d}y}{\mathrm{d}x} = \frac{\mathrm{d}y}{\mathrm{d}u} \cdot \frac{\mathrm{d}u}{\mathrm{d}x}
$$

当一个中间变量在下游被多条分支复用（如 $y = x \cdot x$），总梯度等于所有下游分支传回的梯度贡献**相加**（第一分支贡献 $x$，第二分支贡献 $x$，合计 $2x$）。把算子记作节点、张量依赖记作有向边就构成**计算图**：反向阶段每个节点收集所有下游传来的上游梯度之和，乘以自身的局部偏导，再向各输入边分发。

# 一个可以手算的例子

## 前向

构造一个三层标量网络（输入、权重与中间值均为标量）：输入 $x = 1$，权重 $w_1 = 2$、$w_2 = 3$、$w_3 = -1$，前两层使用 ReLU 激活、末层线性输出，目标 $t = 0$，损失取 $L = \frac{1}{2}(y - t)^2$。前向依次算出 $z_1 = 2 \to a_1 = 2 \to z_2 = 6 \to a_2 = 6 \to y = z_3 = -6 \to L = 18$。

## 反向

从 $\mathrm{d}L/\mathrm{d}L = 1$ 出发，先求损失对输出导数 $\mathrm{d}L/\mathrm{d}y = y - t = -6$，再沿链逆序把上游梯度乘当前局部导数：

1. **输出层预激活**：线性出力局部导数为 1，$\mathrm{d}L/\mathrm{d}z_3 = -6 \times 1 = -6$。
2. **输出层权重梯度**：$\mathrm{d}L/\mathrm{d}w_3 = \mathrm{d}L/\mathrm{d}z_3 \times a_2 = -6 \times 6 = -36$。
3. **传向前一层激活**：复用已缓存的 $\mathrm{d}L/\mathrm{d}z_3$，得 $\mathrm{d}L/\mathrm{d}a_2 = \mathrm{d}L/\mathrm{d}z_3 \times w_3 = -6 \times (-1) = 6$。
4. **穿过第二层 ReLU**：因 $z_2 = 6 > 0$ 局部导数为 1，$\mathrm{d}L/\mathrm{d}z_2 = 6$，进而 $\mathrm{d}L/\mathrm{d}w_2 = 6 \times a_1 = 12$。
5. **传到首层**：$\mathrm{d}L/\mathrm{d}a_1 = 6 \times w_2 = 18$；因 $z_1 = 2 > 0$ 得 $\mathrm{d}L/\mathrm{d}z_1 = 18$，故 $\mathrm{d}L/\mathrm{d}w_1 = 18 \times x = 18$。

## 这个例子的网络图

点击下方幻灯片中的“下一步”，先逐步填满前向中间激活，再沿橙色反向虚线观察上游梯度如何与局部因子相乘（权重梯度分开汇总于对应权重旁）：

[html-card height=930](../assets/dl-backprop-graph.html)

# 矩阵形式的三个梯度

记 $\odot$ 为逐元素乘，$\boldsymbol{1} \in \mathbb{R}^{B}$ 为全 1 列向量，第 $l$ 层的反向传播核心完成三项矩阵梯度与一向上游回传：

$$
\begin{aligned}
\mathrm{d}\mathbf{Z}^{(l)} &= \mathrm{d}\mathbf{A}^{(l)} \odot \sigma'\left(\mathbf{Z}^{(l)}\right) \\
\mathrm{d}\mathbf{W}^{(l)} &= \mathrm{d}\mathbf{Z}^{(l)} \left(\mathbf{A}^{(l-1)}\right)^{\top} \\
\mathrm{d}\boldsymbol{b}^{(l)} &= \mathrm{d}\mathbf{Z}^{(l)} \boldsymbol{1} \\
\mathrm{d}\mathbf{A}^{(l-1)} &= \left(\mathbf{W}^{(l)}\right)^{\top}\mathrm{d}\mathbf{Z}^{(l)}
\end{aligned}
$$

**转置位置的维度对齐记忆法**：所有梯度张量必须与原张量形状严格一致。已知 $\mathrm{d}\mathbf{Z}^{(l)}$ 为 $[n_l, B]$、$\mathbf{A}^{(l-1)}$ 为 $[n_{l-1}, B]$、$\mathbf{W}^{(l)}$ 为 $[n_l, n_{l-1}]$：要凑出 $\mathrm{d}\mathbf{W}^{(l)}$ 的 $[n_l, n_{l-1}]$ 必须将批维 $B$ 在中间相消，故唯独右乘 $(\mathbf{A}^{(l-1)})^{\top}$；要凑出 $\mathrm{d}\mathbf{A}^{(l-1)}$ 的 $[n_{l-1}, B]$ 必须将输出维 $n_l$ 在中间相消，故唯独左乘 $(\mathbf{W}^{(l)})^{\top}$；而偏置在 batch 上共享，需沿 $B$ 维求和汇总。

> [!important] 损失归一化与 $1/B$
> 损失函数采用 batch 平均时，系数 $1/B$ 已自动进入顶层梯度并传遍每一层 $\mathrm{d}\mathbf{Z}^{(l)}$；若损失采用求和则不包含 $1/B$。不要在优化器步骤中重复除以 $B$，梯度定义完全忠实于目标函数。

# 自动微分的记录与释放

深度学习框架的**自动微分（Autograd）** 在前向计算中动态搭建计算图：只要张量带 `requires_grad=True` 标记且处于梯度记录上下文中，每次算子调用都会记录输入依赖与对应的反向函数。调用 `loss.backward()` 时按拓扑逆序求导并将梯度累加至各参数 `param.grad`，随后由 `optimizer.step()` 真正更新权重（因此每轮训练前需先 `zero_grad()` 清空旧梯度）。

计算图在前向结束后仍驻留内存以支撑反向计算，在 `backward()` 完成后立即释放；若需对同一计算图二次求导，必须在首次调用时显式传入 `retain_graph=True`。

由于常规做法需把全部 $L$ 层的中间激活留给反向阶段使用，激活显存开销呈 $O(L)$ 增长。**梯度检查点（Gradient / Activation Checkpointing）** 以局部重算换显存：若把 $L$ 层均分为 $K$ 段并仅缓存每段边界激活，前向边界存储占 $O(K)$，反向进入单段重算时需临时缓存段内激活占 $O(L/K)$，总显存开销 $O(K + L/K)$ 由均值不等式在 $K = \sqrt{L}$ 处取得极小值 $O(\sqrt{L})$，代价仅是额外执行约一次前向计算（耗时通常增加约 20% 至 33%）。

# 梯度裁剪与数值稳定

反向传播算出的梯度向量 $\boldsymbol{g} = \nabla_{\boldsymbol{\theta}} L$ 指向参数使损失上升最快的方向；按学习率 $\eta$ 沿负梯度迭代更新的步长选择与自适应优化器详见 [[Gradient-Descent]] 与 [[Learning-Rate-Schedule]]。

深层网络（如 RNN 与深层 Transformer）在反向连乘矩阵后容易出现两类数值病态：

- **梯度爆炸**：雅可比连乘放大造成单步梯度尖峰，容易直接冲飞当前参数。标准防线是更新前执行**全局范数梯度裁剪（Gradient Clipping）**（PyTorch 中为 `torch.nn.utils.clip_grad_norm_`）：统计全网梯度 L2 范数 $\lVert \boldsymbol{g} \rVert_2$，当超过阈值 $c$（大模型预训练常取 $c=1.0$）时按同比例缩回阈值球内，只压缩尖峰幅度、不改变各参数梯度的相对方向：

$$
\boldsymbol{g} \leftarrow \frac{c}{\lVert \boldsymbol{g} \rVert_2} \, \boldsymbol{g} \qquad (\text{当 } \lVert \boldsymbol{g} \rVert_2 > c)
$$

- **梯度消失**：连乘小于 1 的局部因子导致深层梯度信号趋零，裁剪对此无能为力，必须从结构与尺度源头解决，详见 [[Weight-Initialization-And-Variance]]、[[Activation-Function-Comparison]] 与 [[Norm]]。注意零梯度既可能对应极小值，也可能是鞍点或平坦高原，判断收敛应观察损失曲线与验证指标，不可仅看梯度范数。
