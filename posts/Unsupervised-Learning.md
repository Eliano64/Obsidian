---
title: Unsupervised Learning
tags:
  - MachineLearning
categories:
  - Artificial Intelligence
date: 2026-10-10 00:22:43
katex: true
---

# 无监督学习的两条主线

无监督学习仅面对无标签特征矩阵 $\mathbf{X}\in\mathbb{R}^{n\times d}$（行数 $n$ 为样本数，列数 $d$ 为特征数），从数据自身结构中抽取规律，主要分两条主线：

- **聚类（Clustering）**：按距离或密度把 $n$ 个样本划入 $K$ 个互斥群组（**簇**）；簇编号只代表分组标识而不自动对应先验业务含义，目标完全不同于 [[Logistic-Regression|有监督分类]]。
- **降维（Dimensionality Reduction）**：保留核心信息并把 $d$ 维特征压为 $k$ 维（$k < d$，保持样本数 $n$ 不变），缓解高维稀疏、多重共线性与可视化难题。

# K-Means 聚类

## 目标函数与交替坐标下降

**K-Means** 预先指定簇数 $K$，以每簇样本均值作为**质心**。记单样本向量为 $\boldsymbol{x}\in\mathbb{R}^d$，第 $i$ 簇集合为 $C_i$、质心为 $\boldsymbol{\mu}_i\in\mathbb{R}^d$，目标是最小化全库样本到所属质心的**簇内平方和（SSE）**（注意增大 $K$ 会单调压低 SSE，不可跨 $K$ 直接比该数值）：

$$
\operatorname{SSE} = \sum_{i=1}^{K}\sum_{\boldsymbol{x} \in C_i}\left\|\boldsymbol{x} - \boldsymbol{\mu}_i\right\|_2^2
$$

算法通过四步**交替坐标下降**求解该目标（每步必不增大 SSE，有限划分数下必能收敛）：

1. **初始化质心**：选出 $K$ 个起始质心。
2. **固定质心、分配样本**：将每个样本归入平方欧氏距离 $\lVert\boldsymbol{x}-\boldsymbol{\mu}_i\rVert_2^2$ 最近的簇。
3. **固定划分、更新质心**：令导数为零得每簇最优质心恰为簇内 $|C_i|$ 个样本的算术均值 $\boldsymbol{\mu}_i = \frac{1}{|C_i|}\sum_{\boldsymbol{x}\in C_i}\boldsymbol{x}$。
4. **迭代终止**：交替执行第 2、3 步直到划分不再变化或到达最大轮数。

![[../assets/unsupervised-kmeans.svg]]

[html-card height=520](../assets/kmeans-iterations.html)

## K 的选择、局限与替代方案

K-Means 计算复杂度随 $n,K,d$ 线性增长；但随机初值与球形欧氏假设带来边界，常用以下工程修正（量纲先在 [[Data-Preprocessing|数据预处理]] 缩放）：

- **初始敏感与大数据集提速**：采用 **K-Means++** 按距最近已有质心距离平方成正比的概率分散采点，配多轮重跑取最小 SSE；超大规模采用 MiniBatch K-Means。
- **确定簇数 $K$**：借助 SSE 下降拐点（**肘部法则**）与同时衡量簇内紧凑与簇间分离度的**轮廓系数**（区间 $[-1,1]$ 越大越好）协同业务语义选定。

| 遭遇的瓶颈 | 根本原因 | 对策或替代算法 |
| --- | --- | --- |
| 特征量纲差距悬殊 | 欧氏距离被大量纲列主导 | 先对数值特征执行标准化 |
| 存在明显离群点 | 簇均值易被极值拉偏 | 改用真实样本作中心的 **K-Medoids** |
| 非凸/环形等任意形状簇或 $K$ 未知 | 均值加欧氏距离偏爱球形等大簇 | 改用局部密度连通聚类 **DBSCAN**（兼能识别噪声点） |
| 需要输出多粒度层次关系 | K-Means 只输出单层扁平划分 | 改用自底向上合并构建树状结构的**层次聚类** |

# 主成分分析（PCA）

## 几何目标与 SVD 统一推导

与直接丢弃原列的**特征选择**（方差阈值、L1 稀疏等）不同，**主成分分析（PCA）** 属线性**特征提取**：寻找一组彼此正交的新单位坐标轴，使去中心化样本在各主轴上**投影方差依次最大**（代数等价于最小化线性重建平方误差）。

设每列均已减去各自均值的去中心化矩阵为 $\mathbf{X}\in\mathbb{R}^{n\times d}$（若特征单位异质须先标准化，防止大方差列主导主轴），记其奇异值分解为 $\mathbf{X}=\mathbf{U}\mathbf{\Sigma}\mathbf{V}^{\top}$（$\mathbf{U}\in\mathbb{R}^{n\times n},\mathbf{V}\in\mathbb{R}^{d\times d}$ 为正交矩阵，$\mathbf{\Sigma}\in\mathbb{R}^{n\times d}$ 对角元为降序奇异值 $\sigma_1 \ge \dots \ge \sigma_{\min(n,d)} \ge 0$，且 $\mathbf{U}^{\top}\mathbf{U}=\mathbf{I}_n$）。代入样本协方差矩阵 $\mathbf{C} = \frac{1}{n-1}\mathbf{X}^{\top}\mathbf{X}$ 得：

$$
\mathbf{C} = \frac{1}{n-1}\mathbf{V}\mathbf{\Sigma}^{\top}\mathbf{U}^{\top}\mathbf{U}\mathbf{\Sigma}\mathbf{V}^{\top} = \mathbf{V}\left(\frac{\mathbf{\Sigma}^{\top}\mathbf{\Sigma}}{n-1}\right)\mathbf{V}^{\top}
$$

右奇异矩阵 $\mathbf{V}$ 的第 $i$ 列 $\boldsymbol{v}_i\in\mathbb{R}^d$ 恰是第 $i$ 主成分方向，对应特征值为 $\lambda_i = \sigma_i^2/(n-1)$。现代工程库直接对 $\mathbf{X}$ 做 SVD 而不构造 $\mathbf{X}^{\top}\mathbf{X}$，避免条件数平方放大舍入误差。取前 $k$ 列拼成投影矩阵 $\mathbf{W}=[\boldsymbol{v}_1,\dots,\boldsymbol{v}_k]\in\mathbb{R}^{d\times k}$ 右乘即得降维矩阵 $\mathbf{Y}=\mathbf{X}\mathbf{W}\in\mathbb{R}^{n\times k}$（新样本须减同一训练集均值后右乘 $\mathbf{W}$）；$k$ 常按**累计方差解释率** $R_k = \sum_{i=1}^k\lambda_i / \sum_{j=1}^d\lambda_j$ 达 95% 确定（可视化直接取 $2$ 或 $3$）。

![[../assets/unsupervised-pca.svg]]

## 局限与非线性替代

1. **纯线性假设**：弯曲非线性流形会丢失局部拓扑；探索性可视化常改用 **t-SNE** 或 **UMAP**，但两者不产生可复用投影矩阵且低维坐标间绝对距离不可按欧氏尺度解读。
2. **丧失可解释性与极值敏感**：主成分是原始特征的线性混叠且方差基于平方易被离群样本偏转主轴。

# 工程组合与取舍

高维连续特征先用 PCA 去噪降维再送入 K-Means 常比直接聚类更稳健；下游使用对尺度与共线性不敏感的 [[Decision-Tree|树模型]] 时通常无需强行降维；一切加工效果最终均须回到 [[Model-Evaluation]] 验证。下表总结两者职责差异：

| 对比项 | K-Means | PCA |
| --- | --- | --- |
| 输入预处理需求 | 距离依赖量纲，通常需先标准化 | 必须去中心化；量纲异质时须先标准化 |
| 输出形态 | 每个样本簇归属与 $K$ 个质心 | $n\times k$ 低维表示与可复用投影矩阵 $\mathbf{W}$ |
| 核心超参数 | 簇数 $K$ | 保留主成分维数 $k$ |
| 诊断排障重点 | 不同初值波动大说明 $K$ 或球形假设不匹配 | 前几维解释率极低说明信息高度分散，不宜硬压 |
