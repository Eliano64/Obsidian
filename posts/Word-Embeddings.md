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

神经网络内部以数值张量运算，无法直接读取离散文本。自然语言进入模型前需依次经过三步变换：

1. **分词**：将文本切分为最小离散处理单元，称为**词元（token）**。一个词元不一定对应完整词语，生僻词常被拆分为多个子词。
2. **文本编码**：依据固定大小的词表（记词表长度为 $V$）将每个词元映射为唯一整数编号（token ID）。
3. **嵌入**：以编号作为行索引查嵌入矩阵 $\mathbf{E} \in \mathbb{R}^{V \times d}$（$d$ 为嵌入向量维数），将整段长度为 $n$ 的文本转换为形状为 $n \times d$ 的张量（行数 $n$ 为词元数、列数 $d$ 为特征维数）。

## 子词分词：BPE、WordPiece 与 Unigram

按整词切分会导致词表膨胀和大量未登录词（`<unk>`），按单字符切分则会拉长序列并割裂局部语义。Transformer 与现代大模型普遍采用**子词分词**平衡词表规模与覆盖率，主流算法有三类：

- **BPE（Byte-Pair Encoding）**：从基础字符或字节出发，反复合并语料中频次最高的相邻符号对直至达到目标词表大小（GPT、LLaMA、Qwen 等采用）。
- **WordPiece**：同样自底向上合并，但挑选使语言模型语料似然提升最大的一对进行合并（BERT 采用）。
- **Unigram**：先建立大词表，再依据概率模型迭代删减对整体似然损失最小的词元，天然兼容无空格分隔的中文文本。

# 相似度与 one-hot 的局限

评估两个词向量 $\boldsymbol{x}, \boldsymbol{y}$ 的方向接近程度常用**余弦相似度**：

$$
\cos(\boldsymbol{x}, \boldsymbol{y}) = \frac{\boldsymbol{x} \cdot \boldsymbol{y}}{\lVert \boldsymbol{x} \rVert \, \lVert \boldsymbol{y} \rVert}
$$

其取值范围为 $[-1, 1]$，越接近 $1$ 表示方向越一致。基于上下文分布习得的词向量反映的是**语境相似性**：如反义词“爱”与“恨”常出现在相近句法位置，余弦相似度往往仍为正。

最朴素的文本向量化是 **one-hot 编码**：为词表中第 $i$ 个词元构造长度为 $V$ 的向量，仅第 $i$ 位取 $1$、其余 $V-1$ 位全为 $0$。该表示有两项致命缺陷：向量极度高维稀疏导致存储与计算低效；任意两个不同词元的向量严格正交（内积恒为 $0$、欧氏距离恒为 $\sqrt{2}$），完全无法体现语义关联。

# 从分布假设到 Word2Vec

**分布假设**指出：经常出现在相似上下文中的词元，用法与语义往往相近。早期**上下文计数矩阵**直接统计每个词元在左右窗口内与其他词的共现次数作为特征向量，但面临维度高且极度稀疏、受高频功能虚词主导、静态统计表无法随新语料增量更新三大瓶颈。

**Word2Vec** 改用端到端局部预测任务学习低维稠密矩阵：设定窗口半径 $k$，将当前中心词元左右各 $k$ 个词记为上下文集合 $\mathcal{C}$；模型同时维护两张形状为 $V \times d$ 的参数表，即输入嵌入表 $\mathbf{E}$（每行 $\boldsymbol{e}_w$ 作为最终词向量保留）与辅助输出打分表 $\mathbf{U}$（每行 $\boldsymbol{u}_w$ 仅在训练期给目标词 $w$ 打分，训练结束后丢弃）。

- **CBOW（连续词袋）**：将上下文嵌入取均值 $\boldsymbol{h} = \frac{1}{|\mathcal{C}|}\sum_{c \in \mathcal{C}} \boldsymbol{e}_c$，与 $\mathbf{U}$ 各行做内积后经 Softmax 预测中心词表目标编号 $w_{\text{target}}$（每个窗口产生 1 次预测）：

$$
p(w \mid \boldsymbol{h}) = \frac{\exp(\boldsymbol{u}_w^{\top}\boldsymbol{h})}{\sum_{v=1}^{V} \exp(\boldsymbol{u}_v^{\top}\boldsymbol{h})}, \qquad L = -\ln p(w_{\text{target}} \mid \boldsymbol{h})
$$

- **Skip-gram（跳字模型）**：采用相反方向，由中心词嵌入 $\boldsymbol{e}_{\text{center}}$ 分别预测窗口内每一个上下文词元 $t \in \mathcal{C}$（每个窗口产生 $|\mathcal{C}|$ 次预测），梯度更新同样沿上下文位置聚集语义相近的向量。

下表汇总两种训练方向的输入输出对应关系，并在下方卡片中演示向量更新过程：

| 模型方向 | 模型输入 | 预测目标 | 每窗口预测次数 |
| --- | --- | --- | --- |
| CBOW | 上下文词元嵌入均值 | 中心词元 | $1$ |
| Skip-gram | 中心词元嵌入 | 逐个上下文词元 | 上下文个数 $|\mathcal{C}|$ |

[html-card height=820](../assets/word2vec-cbow-skipgram.html)

## 负采样加速

全词表 Softmax 分母遍历全部 $V$ 个词，每步需计算并更新 $\mathbf{U}$ 的全部行，复杂度达 $O(V\cdot d)$。工程上常采用**负采样（Negative Sampling）**将其改写为 $1 + K$ 个二分类问题：记真实目标词为正例 $w_+$，按词频的 $3/4$ 次幂分布 $P_n(w) \propto \operatorname{count}(w)^{3/4}$（抑制超高频词垄断）抽取 $K$ 个非目标词作为负例 $w_1^-,\dots,w_K^-$；记输入隐向量为 $\boldsymbol{h}$、$\sigma$ 为 Sigmoid 函数，目标函数变为：

$$
L_{\text{NEG}} = -\ln\sigma\left(\boldsymbol{u}_{w_+}^{\top}\boldsymbol{h}\right) - \sum_{j=1}^{K}\ln\sigma\left(-\boldsymbol{u}_{w_j^-}^{\top}\boldsymbol{h}\right)
$$

每步仅更新 $\mathbf{U}$ 中 $1 + K$ 行，复杂度降为 $O((K+1)\cdot d)$。

# 静态嵌入的局限与向 Transformer 的演进

Word2Vec 为词表中的每个词元分配唯一**静态向量**，查表结果不随所在句子语境变化，“苹果发布新品”与“吃了一个苹果”中的“苹果”共用同一向量，无法处理一词多义。

在 [[Transformer-Architecture|Transformer]] 中，词嵌入矩阵 $\mathbf{E}$ 退居第 0 层的静态查表入口；离散词元先经查表转换为初始向量，再进入后续多层 [[Attention|自注意力]] 根据全句上下文动态汇总信息，最终形成依赖语境的动态表示。
