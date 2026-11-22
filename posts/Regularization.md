---
title: Regularization
tags:
  - MachineLearning
categories:
  - Artificial Intelligence
date: 2026-11-22 21:58:08
katex: true
---

**正则化（Regularization）** 在保留模型主结构的前提下，从损失函数、参数更新、网络结构、训练流程或数据侧追加约束来压缩有效容量，用略高的训练误差换取更低的验证误差，从而抑制 [[Model-Evaluation#欠拟合与过拟合|过拟合]]。约束强度须由独立验证集挑选，过强会把模型推向欠拟合；现代训练流水线常将多种机制叠加使用：

| 方法 | 作用位置 | 核心机制 | 工程要点 |
| --- | --- | --- | --- |
| L1 / L2 正则化 | 损失函数 | 追加权重绝对值或平方和惩罚，限制幅度 | 线性模型可用 L1 稀疏选特征；深网首选光滑的 L2 |
| 权重衰减（AdamW） | 参数更新 | 每步按 $(1-\eta\lambda)$ 直接缩放权重 | 配合自适应优化器时须与梯度解耦，直接使用 AdamW |
| Dropout | 网络结构 | 训练前向按概率 $p$ 置零并用 $1/(1-p)$ 缩放存活输出 | 按验证曲线调 $p$；海量语料 LLM 预训练常取 $0$ |
| 早停 | 训练流程 | 连续 `patience` 轮验证无改善即停并回退最佳权重 | 严禁用测试集代替验证集作停止判据 |
| 数据增强与标签平滑 | 数据与标签 | 保标签随机变换拓宽输入分布，软化硬标签 | 确保变换不破坏语义，可与上述正则协同叠加 |

# 损失函数约束：L1 与 L2

记原始经验损失为 $\mathcal{L}_{\text{原始}}$，全部标量权重为 $w_j$（$j$ 为下标），正则系数 $\lambda > 0$。直接在损失后追加惩罚即得 **L2 正则（Ridge）** 与 **L1 正则（Lasso）**：

$$
\mathcal{L}_{\text{L2}} = \mathcal{L}_{\text{原始}} + \frac{\lambda}{2}\sum_{j} w_j^{2}, \qquad
\mathcal{L}_{\text{L1}} = \mathcal{L}_{\text{原始}} + \lambda\sum_{j} \left|w_j\right|
$$

两者在参数空间呈现不同约束几何形态：

- **L2 正则化**：等值面为光滑超球面，导数 $\lambda w_j$ 均匀压小所有权重但通常不归零；光滑可导，契合深度神经网络的协同表达需求。
- **L1 正则化**：等值面为带尖角多面体，易先在坐标轴碰壁使大量权重恰等于 $0$，从而自动完成稀疏特征选择（只保证总体损失极小，不保证因果相关性）。

[html-card height=820](../assets/model-evaluation-l1-l2.html)

# 权重衰减与 AdamW

工程更习惯直接在参数更新步中衰减权重，但这在标准 [[Gradient-Descent|SGD]] 与自适应优化器中的等价性完全不同。

## 权重衰减与 SGD 下的等价性

记单个权重在第 $t$ 步的取值为 $w_t$、学习率为 $\eta$、衰减系数为 $\lambda$（常取 $10^{-4}$ 至 $10^{-2}$）。**权重衰减（Weight Decay）** 先按 $(1-\eta\lambda)$ 收缩 $w_t$ 再扣减原始梯度步长；将 $\mathcal{L}_{\text{L2}}$ 求导代入标准 SGD 更新式可得完全一致的结果（系数 $\frac{1}{2}$ 恰好抵消求导系数 $2$）：

$$
w_{t+1} = w_t - \eta\left(\frac{\partial \mathcal{L}_{\text{原始}}}{\partial w_t} + \lambda w_t\right) = (1 - \eta\lambda)w_t - \eta\frac{\partial \mathcal{L}_{\text{原始}}}{\partial w_t}
$$

## 自适应优化器中的解耦：AdamW

换用 Adam 时 L2 惩罚与权重衰减**不再等价**：把 $\lambda w_t$ 写入损失会令其混入总梯度 $g_t = \frac{\partial \mathcal{L}_{\text{原始}}}{\partial w_t} + \lambda w_t$ 并进入一阶矩 $m_t$ 与二阶矩 $v_t$；经偏差修正 $\hat{m}_t, \hat{v}_t$ 并除以 $\sqrt{\hat{v}_t}$ 后，历史梯度小者惩罚被放大、梯度大者衰减被削弱，$\lambda$ 失去统一语义。

**AdamW** 将权重衰减彻底从自适应梯度通道解耦，使偏差修正矩 $\hat{m}_t, \hat{v}_t$ 只由原始梯度算出，再直接对 $w_t$ 施以衰减（$\epsilon$ 为防除零常数）：

$$
w_{t+1} = (1 - \eta\lambda)w_t - \frac{\eta}{\sqrt{\hat{v}_t} + \epsilon}\hat{m}_t
$$

解耦恢复了统一的惩罚尺度，是 Transformer 与大语言模型的标准写法。

[html-card height=820](../assets/adam-vs-adamw-update.html)

# 网络结构与训练流程约束

## Dropout

**Dropout（随机失活）** 在训练前向以概率 $p$ 随机将一层神经元的激活置零（打破神经元对特定邻居的共适应依赖，并在推理时等效大量子网络的隐式集成）。

工程普遍采用**反向 Dropout（Inverted Dropout）**：训练阶段将存活神经元输出同乘 $\frac{1}{1-p}$ 维持期望不变，推理阶段直接关闭随机置零即可。丢弃率 $p$ 依场景调整：全连接隐藏层常取 $0.5$，CNN 常置于池化或全连接之后，经典 Transformer 取 $0.1$，而在数据海量的现代大模型预训练中通常设 $p=0$；注意 [[Norm|BatchNorm]] 的批内噪声不能替代专门 Dropout。

## 早停

**早停（Early Stopping）** 靠监控独立验证集（严禁看测试集，见 [[Model-Evaluation#划分与交叉验证|划分与交叉验证]]）限制参数偏离初值的搜索半径：连续 `patience` 轮（常取 `5` 到 `20`）验证指标无改善即中止训练。由于确认过拟合滞后于最优拐点数轮，停机后务必加载验证表现最佳的历史 checkpoint。

[html-card height=460](../assets/early-stopping-window.html)

# 训练数据与标签约束

- **数据增强（Data Augmentation）**：对训练样本施以水平翻转、随机裁剪或亮度抖动扩宽分布，迫使模型学习跨变换的不变表示；前提是**必须保持原标签语义成立**（切掉核心目标会直接注入标签噪声，见下图对比）。
- **标签平滑与样本混合**：**标签平滑（Label Smoothing）** 将硬 one-hot 的 $1$ 与 $0$ 微调向均匀分布软化，抑制 logits 冲向正无穷；**Mixup / CutMix** 按比例线性混合两幅图像及其标签使类间边界过渡更平滑。

![左侧平移、镜像与亮度变化保留杯子标签；右侧裁剪把杯子移出画面，标签不再成立|390](../assets/evaluation-label-transforms.svg)
