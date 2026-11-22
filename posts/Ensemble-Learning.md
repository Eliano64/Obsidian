---
title: Ensemble Learning
tags:
  - MachineLearning
categories:
  - Artificial Intelligence
date: 2026-10-09 23:58:09
katex: true
---

# 偏差-方差双路径：Bagging 与 Boosting

单棵深 [[Decision-Tree|决策树]] 偏差小但**方差极高**（训练集增删几个样本就会剧烈改写整棵树结构）。**集成学习**组合多棵基树（棵数记为 $T$）以改善泛化，依降方差或降偏差分出两条路线：

| 路线 | 训练方式与目标 | 核心优化对象 | 代表算法 |
| --- | --- | --- | --- |
| **Bagging** | 各基学习器相互独立并行，各自拟合原始标签 $y$ 后平均/投票 | 压低**方差**（基学习器用低偏差深树） | 随机森林（Random Forest） |
| **Boosting** | 基学习器按序串行，每轮新树拟合前序模型未修完的负梯度 | 压低**偏差**（基学习器用低方差浅树） | GBDT、XGBoost、LightGBM、CatBoost |

## 平均如何削减方差

设集合有 $T$ 棵同分布基树 $h_1,\dots,h_T$，每棵误差方差均为 $\sigma^2$，任意两棵误差相关系数为 $\rho$，简单平均 $\bar{h} = \frac{1}{T}\sum_{t=1}^T h_t$ 的误差方差满足：

$$
\operatorname{Var}(\bar{h}) = \rho\sigma^2 + \frac{1 - \rho}{T}\sigma^2
$$

若树之间完全独立（$\rho=0$），平均将方差线性压至 $\sigma^2/T$；但若树高度相关，即使 $T\to\infty$，方差仍被下界 $\rho\sigma^2$ 锁死。此外平均不能削减成员共有的系统偏差，故 Bagging 须用低偏差深树并尽量压低树间相关性 $\rho$。

# 随机森林

**随机森林**以两层随机性打破各树同质化、刻意降低 $\rho$：

1. **样本随机（Bootstrap）**：每棵树从 $n$ 条样本中有放回抽取 $n$ 条训练，单样本全程未被抽中比例趋近 $(1-1/n)^n \to e^{-1} \approx 36.8\%$（称**袋外样本 OOB**，可直接作无额外划分误差估计，Boosting 无此机制）。
2. **特征随机**：每个节点只在随机抽出的特征子集（分类常取 $\sqrt{d}$ 个，$d$ 为总特征数）中挑最优分裂，避免强特征主导全部树顶层。

充分生长 $T$ 棵深树后（记第 $t$ 棵树对 $\boldsymbol{x}$ 的预测为 $h_t(\boldsymbol{x})$，类别索引 $c$，指示函数 $\mathbb{I}[\cdot]$），分类取多数表决、回归取均值：

$$
\hat{y}_{\text{cls}} = \arg\max_c \sum_{t=1}^{T} \mathbb{I}\left[h_t(\boldsymbol{x}) = c\right], \qquad
\hat{y}_{\text{reg}} = \frac{1}{T}\sum_{t=1}^{T} h_t(\boldsymbol{x})
$$

特征重要度常用不纯度下降加权总和 **MDI**（易高估高基数列）与验证集打乱列的 **Permutation**（无高基数偏好，两者皆不代表因果）。随机森林天然并行、不要求特征缩放、抗过拟合稳定，适合作为表格强基线，但丧失单树规则可读性且推理显存与延迟上升。

# 梯度提升树（GBDT）

**GBDT** 将 [[Gradient-Descent|梯度下降]] 迁移至函数空间：由 $M$ 棵弱树按学习率（收缩系数）$\eta\in(0,1]$ 串行加法累积（第 $m$ 轮基树为 $h_m(\boldsymbol{x})$）：

$$
F_m(\boldsymbol{x}) = F_{m-1}(\boldsymbol{x}) + \eta\,h_m(\boldsymbol{x}), \qquad
F_M(\boldsymbol{x}) = \sum_{m=1}^{M} \eta\,h_m(\boldsymbol{x})
$$

每轮依次执行三步：计算当前 $F_{m-1}$ 上各样本损失负梯度 $\to$ 用浅树 $h_m$ 拟合负梯度 $\to$ 按步长 $\eta$ 累加进总模型。

> [!note] 为何 Boosting 必须配浅树
> 逐轮修正已能充分压低偏差；若每棵基树过深，既压缩了后轮修正空间，又会把单轮拟合出的随机噪声逐轮累加放大。

![[../assets/ensemble-gbdt.svg]]

[html-card height=460](../assets/ensemble-gbdt-steps.html)

## 为什么常说“拟合残差”

对单样本平方误差 $L=\frac{1}{2}(y-F(\boldsymbol{x}))^2$（$y$ 为真实标签）求负梯度 $-\frac{\partial L}{\partial F(\boldsymbol{x})} = y - F(\boldsymbol{x})$ 恰等于残差。下表对照三大损失函数的负梯度目标；整体虽精度强且不惧特征缩放，但树间无法跨轮并行、小样本高噪需仔细早停，且不擅长超高维稀疏词袋特征：

| 损失函数 | 负梯度（基树拟合目标） | 核心特性 |
| --- | --- | --- |
| 均方误差 MSE | $y - F(\boldsymbol{x})$ | 恰好等于残差，偏差越大修正幅值越大 |
| 绝对误差 MAE | $\operatorname{sign}(y - F(\boldsymbol{x}))$ | 只保留符号 $\pm 1$，抑制离群异常点拉偏 |
| 对数损失（分类） | $y - p$（$p=\sigma(F(\boldsymbol{x}))$ 为正类概率） | 形式同残差，衡量真实标签与预测概率差值 |

# 三大工业实现与选型

## XGBoost

在 GBDT 一阶负梯度基础上引入二阶泰勒展开与树复杂度正则。设训练样本 $i=1,\dots,n$（输入 $\boldsymbol{x}_i$），前 $t-1$ 轮预测 $\hat{y}_i^{(t-1)}$ 下损失一阶、二阶导分别为 $g_i, h_i$；候选树 $f_t$ 含 $J$ 个叶子（第 $j$ 叶权重 $w_j$、落入样本集 $I_j$），$\gamma,\lambda$ 为叶数与叶权重 L2 正则系数，则该轮目标及固定树结构最优叶权重为：

$$
\tilde{\mathcal{L}}^{(t)} = \sum_{i=1}^{n}\left[g_i f_t(\boldsymbol{x}_i) + \frac{1}{2}h_i f_t^2(\boldsymbol{x}_i)\right] + \gamma J + \frac{1}{2}\lambda\sum_{j=1}^{J} w_j^2, \qquad
w_j^\ast = -\frac{\sum_{i \in I_j} g_i}{\sum_{i \in I_j} h_i + \lambda}
$$

分母 $\sum h_i$ 自动按损失曲率缩放步长，$\lambda$ 在样本稀少叶片防权重虚高，回代即可闭式评估切分增益。

## LightGBM 与 CatBoost

- **LightGBM**：主打百万级以上大样本吞吐，核心采用**直方图分桶**降切分搜索代价、带深限的 **Leaf-wise**（优先分裂增益最大叶子）提精度、单边梯度采样（**GOSS**）与互斥特征捆绑（**EFB**）降计算量。
- **CatBoost**：专为大量离散类别列打造，用按随机序仅看前面样本的**有序目标统计量（Ordered Target Statistics）**联合**排序提升（Ordered Boosting）**消除目标编码泄露；默认同层共用条件的**对称树**利于极速批量推理。

## 选型对照

新表格任务先用随机森林跑通稳健基线，再依据样本规模与类别特征比例选用对应 GBDT 实现：

| 业务场景 | 推荐框架 | 核心依据 |
| --- | --- | --- |
| 竞赛与中大表格高精度需求 | XGBoost / LightGBM | 二阶精确切分增益、正则完善、生态成熟 |
| 百万以上样本需极速训练 | LightGBM | 直方图分桶 + Leaf-wise + GOSS/EFB 吞吐极高 |
| 包含大量离散类别列 | CatBoost | 原生有序目标统计防泄露，对称树推理快 |
