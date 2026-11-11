---
title: Word Embeddings
tags:
  - NLP
categories:
  - Artificial Intelligence
date: 2026-10-27 23:29:21
katex: true
---
# 词的表示

分词器先把文本切成 Token。Token 不总等于自然语言里的词：它可以是一个词、一个子词或一个字符，一个词也可能被拆成多个 Token。序列长度按 Token 数计算。

文本进入模型前是离散符号，而模型内部只有数值运算，所以第一步是把 Token 映射成行向量，称为嵌入向量；它要同时承担两件事：区分不同 Token，让语义相近的 Token 落在相近位置。

## 余弦相似度

常使用**余弦相似度**量化 token 的语义相似关系。对于嵌入向量 $\boldsymbol{x},\boldsymbol{y}$：

$$
\cos(\boldsymbol{x}, \boldsymbol{y}) = \frac{\boldsymbol{x} \cdot \boldsymbol{y}}{\lVert \boldsymbol{x} \rVert \, \lVert \boldsymbol{y} \rVert}
$$

接近 1 表示词义相近，接近 −1 表示词义相反，接近 0 表示词义几乎无关。

# one-hot 编码

一种表示词的方案是 **one-hot 编码**。词表有 $V$ 个 Token，向量长度就是 $V$，只有对应位置为 1，其他皆为 0 。

这种方案简单，但明显存在不足。在词表很大的情况下，向量的长度变得极长，数值运算变得极为不便；而且，任意两个不同 Token 的内积为 0、欧氏距离都是 $\sqrt2$。所有 Token 对彼此同样远，相似度无从表达。

# 上下文计数向量

**上下文计数向量**基于分布假设。

先固定一个上下文窗口：以一个 Token 为中心，取它左右各 $k$ 个 Token，$k$ 称为窗口半径。比如句子“小明 今天 想 吃 苹果”中，以“想”为中心、$k=2$ 时，上下文是“小明”“今天”“吃”“苹果”。

分布假设认为：经常出现在相似上下文里的 Token，用法往往相近，因此可以用上下文构造表示。苹果与香蕉常出现在“吃了一个\_\_\_”“\_\_\_很甜”这类位置，周围常是“吃”“甜”“水果”；汽车与火车周围常是“道路”“发动机”。两类 Token 的上下文不同，表示也应不同。

选定一组上下文 Token 作为列，对每个中心 Token，统计它在语料（用于统计的文本集合）中与各列 Token 在窗口内一起出现的次数；一个中心 Token 的计数排成一行，所有行组成上下文计数矩阵。表中的行是中心 Token，列是上下文 Token，格子是次数。统计出来的结果示例：

| 中心 Token | 吃 | 甜 | 水果 | 道路 | 发动机 |
| --- | --- | --- | --- | --- | --- |
| 苹果 | 38 | 24 | 31 | 1 | 0 |
| 香蕉 | 35 | 28 | 29 | 0 | 0 |
| 汽车 | 0 | 0 | 1 | 27 | 33 |
| 火车 | 0 | 0 | 0 | 21 | 18 |

表中的每一行就是一个 Token 的上下文计数向量。五个分量依次对应吃、甜、水果、道路、发动机。对每个 Token 都取一行，就得到它的上下文计数向量。例如苹果的向量是 $\boldsymbol{c}_{\text{苹果}}=(38,24,31,1,0)$，汽车的向量是 $\boldsymbol{c}_{\text{汽车}}=(0,0,1,27,33)$。

苹果与香蕉对应的行在吃、甜、水果三列都很大，说明它们常出现在相似上下文里；汽车与火车在道路、发动机两列很大，说明它们的上下文属于另一类。

在这张表里，同类 Token 的行方向接近，跨类 Token 的行方向接近正交。例如，比较苹果与香蕉、苹果与汽车两个向量对的余弦：

$$
\cos(\boldsymbol{c}_{\text{苹果}},\boldsymbol{c}_{\text{香蕉}})
=\frac{38\cdot35+24\cdot28+31\cdot29+1\cdot0+0\cdot0}
{\sqrt{38^2+24^2+31^2+1^2+0^2}\sqrt{35^2+28^2+29^2+0^2+0^2}}
=\frac{2901}{54.598\times53.385}\approx0.995
$$

$$
\cos(\boldsymbol{c}_{\text{苹果}},\boldsymbol{c}_{\text{汽车}})
=\frac{38\cdot0+24\cdot0+31\cdot1+1\cdot27+0\cdot33}
{\sqrt{38^2+24^2+31^2+1^2+0^2}\sqrt{0^2+0^2+1^2+27^2+33^2}}
=\frac{58}{54.598\times42.649}\approx0.025
$$

苹果与香蕉的余弦接近 1，苹果与汽车的余弦接近 0。

但这种方式有两点缺陷：
1. 计数只有非负值。余弦只能落在 0 到 1，无法表示“爱”与“恨”这类相反词义；
2. 计数的偶然性大。它极度依赖窗口半径、列的选择与语料，换一个窗口半径或换一份语料，同一对 Token 的计数和向量可能会变化巨大。

# Word2Vec

Word2Vec 通过预测任务训练嵌入向量。以句子“小明 今天 想 吃 苹果”为例，CBOW 用上下文预测中心 Token“想”；Skip-gram 用中心 Token“想”预测其上下文。

**预测任务实质是利用上下文定义向量**。上下文越接近的两个 Token，模型对它们要给出的预测越接近。

例如苹果与香蕉常出现在相似上下文，模型对它们要给出相近预测，它们的向量因此靠近；汽车出现在另一类上下文，向量就被拉开。因此，预测越准，向量越能保留可迁移的上下文差异。

## CBOW

### 窗口

先固定窗口半径 $k$。以某个 Token 为中心，取它左右各 $k$ 个 Token 作为上下文；记这个上下文 Token 集合为 $\mathcal{C}$。比如句子“小明 今天 想 吃 苹果”中，以“想”为中心、$k=2$ 时，$\mathcal{C}=\{\text{小明},\text{今天},\text{吃},\text{苹果}\}$。

窗口沿句子滑动，每个中心位置产生一组训练样本；句子两端的窗口不完整，上下文个数会少一些。

### 两张表

设 Token 词表大小为 $V$，嵌入向量维数为 $d$。Word2Vec 有两张形状相同的表。输入表 $\mathbf{E}\in\mathbb{R}^{V\times d}$，行是 Token 作为上下文或中心 Token 时的嵌入向量；输出表 $\mathbf{U}\in\mathbb{R}^{V\times d}$，行是 Token 作为被预测目标时的打分向量。两张表各自初始化、各自更新。$\boldsymbol{e}_w$ 与 $\boldsymbol{u}_w$ 分别是两表的第 $w$ 行，$w$ 是 Token 的编号。

$\mathbf{E}$ 的行就是嵌入向量，也是预测模型的参数；$\mathbf{U}$ 只在训练时给目标 Token 打分，训练结束后丢弃。

### CBOW 的预测

CBOW（Continuous Bag of Words，连续词袋）用上下文 Token 的嵌入向量 $\boldsymbol{e}$ 均值预测中心 Token：

$$
\boldsymbol{h} = \frac{1}{|\mathcal{C}|} \sum_{c \in \mathcal{C}} \boldsymbol{e}_c
$$

用 $\operatorname{softmax}(\mathbf{U}\boldsymbol{h})$ 得到词表上的概率，训练目标是最大化中心 Token 的概率。每个窗口只有一个中心 Token，因此每个窗口产生一次预测；训练更新上下文 Token 在 $\mathbf{E}$ 中的行，以及 $\mathbf{U}$ 中被预测中心 Token 的行。

### 训练

设词表编号为 $w$ 的 Token 得分为 $\boldsymbol{u}_w^{\top}\boldsymbol{h}$，概率由 softmax 归一化：

$$
p(w \mid \boldsymbol{h}) = \frac{\exp(\boldsymbol{u}_w^{\top}\boldsymbol{h})}{\sum_{v=1}^{V} \exp(\boldsymbol{u}_v^{\top}\boldsymbol{h})}
$$

设目标 Token 的编号为 $w_{\text{target}}$，损失是它的负对数概率 $L=-\ln p(w_{\text{target}}\mid \boldsymbol{h})$：目标概率越低，损失越大。反向传播更新参与计算的 $\mathbf{E}$ 行与 $\mathbf{U}$ 行。

训练结束后，$\mathbf{E}$ 的每一行就是对应 Token 的嵌入向量。这个向量在词表中固定，不随句子变化。

训练得到的嵌入向量是一串实数，单个分量通常没有可命名的含义：不能说第 3 维代表“甜”，也不能说第 5 维代表“颜色”；这些数值由训练目标共同调整。

## Skip-gram

Skip-gram（跳字模型）沿用 CBOW 的窗口、两张表与 softmax，采用相反的预测方向：用中心 Token 的嵌入向量分别预测每个上下文 Token。记中心 Token 的嵌入向量为 $\boldsymbol{e}_{\text{center}}$，对每个上下文 Token $t \in \mathcal{C}$：

$$
L = -\sum_{t \in \mathcal{C}} \ln p(t \mid \text{center}), \qquad p(t \mid \text{center}) = \operatorname{softmax}(\mathbf{U}\boldsymbol{e}_{\text{center}})_t
$$

一个窗口有多个上下文 Token，每个（中心，上下文）对产生一次预测，因此一个窗口产生多次预测；中心 Token 的嵌入向量被反复用于这些预测，$\mathbf{U}$ 中每个上下文 Token 的行都会被更新。

## 两种方向的对比

| 方向 | 输入 | 预测目标 | 每个窗口的预测次数 |
| --- | --- | --- | --- |
| CBOW | 上下文 Token 的嵌入向量均值 | 中心 Token | 1 |
| Skip-gram | 中心 Token 的嵌入向量 | 每个上下文 Token | 上下文个数 |

[html-card height=820](../assets/word2vec-cbow-skipgram.html)

## 局限

每个 Token 在词表里只有一个向量。“苹果发布新品”和“吃了一个苹果”里的“苹果”共用同一向量，模型无法仅凭嵌入区分公司与水果；需要上下文相关表示时，要靠后续序列层根据句子更新。
