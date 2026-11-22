---
title: Position Encoding
tags:
  - Transformer
categories:
  - Artificial Intelligence
date: 2026-11-09 22:35:00
katex: true
---

# 为什么需要位置编码

循环网络靠时间步递推隐含次序，而 Transformer 对整个序列并行计算，[[Attention#位置信息|自注意力本身具有置换等变性、不感知词元顺序]]，因此必须显式注入位置信息。

记词元在序列中的位置索引为 $i$（从 $0$ 起），模型隐藏维数为 $d_{\text{model}}$，第 $i$ 个词元的输入词嵌入行向量为 $\boldsymbol{e}_i = \operatorname{Embedding}(x_i) \in \mathbb{R}^{1\times d_{\text{model}}}$，对应位置向量为 $\boldsymbol{p}_i = \operatorname{PositionalEncoding}(i) \in \mathbb{R}^{1\times d_{\text{model}}}$。经典绝对位置编码采用逐元素相加注入，使同一词元在不同位置形成不同输入向量 $\boldsymbol{x}_i$，从而进入后续投影后参与区分次序：

$$
\boldsymbol{x}_i = \boldsymbol{e}_i + \boldsymbol{p}_i
$$

# 绝对位置编码：正弦与余弦

绝对位置编码须在输入端确定合并方式与数值表示形式，Transformer 的设计取舍如下表所示：

| 设计决策 | 采用方案 | 弃用方案与根本缺陷 |
| --- | --- | --- |
| 合并方式 | **逐维相加**（保持 $d_{\text{model}}$ 维数） | 拼接维数为 $d_{\text{pos}}$ 的向量会扩宽所有 $\mathbf{Q},\mathbf{K},\mathbf{V}$ 投影矩阵；且投影混合后位置与语义仍落入同一输出空间 |
| 表示形式 | **多频率有界周期向量**（各维落在 $[-1, 1]$） | 直接输入整数编号随句长无限膨胀超过嵌入范围；归一化至 $[0,1]$ 则会锁死最大序列长度且无法表达相邻位移关系 |

## 公式与频率

正弦位置编码根据位置索引 $pos$ 与偶数模型维数 $d_{\text{model}}$ 构造向量 $\boldsymbol{p}_{pos}\in\mathbb{R}^{1\times d_{\text{model}}}$：将维度两两配对为 $d_{\text{model}}/2$ 组（组索引 $i=0,1,\dots,\frac{d_{\text{model}}}{2}-1$），令波长分母 $D_i = 10000^{2i/d_{\text{model}}}$，每组写出正弦与余弦两维分量：

$$
PE(pos,2i)=\sin\left(\frac{pos}{D_i}\right),\qquad
PE(pos,2i+1)=\cos\left(\frac{pos}{D_i}\right)
$$

第 $i=0$ 组波长最短、变化最快（刻画邻近位置差别）；随着 $i$ 增大，波长指数拉长、变化变慢（容纳长距跨度），类似二进制计数由低位到高位的多尺度编码。

## 相对距离

取位置 $pos$ 与 $pos+k$（相距位移 $k$）在输入端直接做点积，第 $i$ 组正余弦乘积恰好化为只依赖 $k$ 的余弦差角：

$$
\boldsymbol{p}_{pos}\cdot\boldsymbol{p}_{pos+k}
= \sum_{i=0}^{d_{\text{model}}/2-1}\left(\sin\frac{pos}{D_i}\sin\frac{pos+k}{D_i}+\cos\frac{pos}{D_i}\cos\frac{pos+k}{D_i}\right)
= \sum_{i=0}^{d_{\text{model}}/2-1}\cos\left(\frac{k}{D_i}\right)
$$

右端消去绝对位置 $pos$，只取决于相对间隔 $k$（注意点积是 $k$ 的偶函数，不直接分辨先后方向，方向需依赖投影后的相位差区分）。

# 旋转位置编码（RoPE）

输入端相加的绝对位置编码经单头投影矩阵 $\mathbf{W}_Q,\mathbf{W}_K\in\mathbb{R}^{d_{\text{model}}\times d_k}$ 变换后，位置 $m$ 与 $n$ 处的打分点积展开为四项：

$$
\left(\boldsymbol{e}_m + \boldsymbol{p}_m\right)\mathbf{W}_Q \mathbf{W}_K^{\top} \left(\boldsymbol{e}_n + \boldsymbol{p}_n\right)^{\top}
= \boldsymbol{e}_m\mathbf{W}_Q\mathbf{W}_K^{\top}\boldsymbol{e}_n^{\top}
+ \boldsymbol{e}_m\mathbf{W}_Q\mathbf{W}_K^{\top}\boldsymbol{p}_n^{\top}
+ \boldsymbol{p}_m\mathbf{W}_Q\mathbf{W}_K^{\top}\boldsymbol{e}_n^{\top}
+ \boldsymbol{p}_m\mathbf{W}_Q\mathbf{W}_K^{\top}\boldsymbol{p}_n^{\top}
$$

中间可学习矩阵 $\mathbf{W}_Q\mathbf{W}_K^{\top}$ 会跨不同频率维度任意混合，使第四项不再严格只依赖相对位移 $n-m$。**旋转位置编码（Rotary Position Embedding, RoPE）** 改为在投影**之后**，直接在注意力头内对查询与键向量按位置施加二维子空间旋转，从代数结构上直接保留相对位移信息。

> [!important] 为何 RoPE 只旋转 Q 与 K、不旋转 V
> 位置信息的作用是让查询与键在点积打分时体现相对距离；值矩阵 $\mathbf{V}$ 不参与匹配打分，只按归一化权重被汇总。如果强行旋转 $\mathbf{V}$，只会让聚合特征随绝对位置产生不必要旋转，破坏输出稳定性。

## 旋转方式

设单头查询与键维数为偶数 $d_k$，将其两两配对分为 $d_k/2$ 个二维子平面（组索引 $j=0,1,\dots,\frac{d_k}{2}-1$）。第 $j$ 组角频率 $\omega_j$ 与位置 $m$ 的旋转角 $\theta_{m,j} = m\omega_j$ 定义如下（经典底数 $\theta_{\text{base}} = 10{,}000$，长上下文现代大模型常提高至 $500{,}000$ 或 $1{,}000{,}000$）：

$$
\omega_j = \theta_{\text{base}}^{-2j/d_k}, \qquad
\mathbf{R}(\theta) = \begin{bmatrix} \cos\theta & -\sin\theta \\ \sin\theta & \cos\theta \end{bmatrix}
$$

对于位置 $m$ 的查询 $\boldsymbol{q}_m$，其第 $j$ 组二维分量 $(q_{2j}, q_{2j+1})$ 写作列向量左乘 $\mathbf{R}(m\omega_j)$ 完成正交旋转（位置 $n$ 的键 $\boldsymbol{k}_n$ 第 $j$ 组 $(k_{2j}, k_{2j+1})$ 同理旋转 $\mathbf{R}(n\omega_j)$）；正交旋转只改变角度、保持向量长度不变：

$$
\begin{bmatrix} q'_{2j} \\ q'_{2j+1} \end{bmatrix} = \mathbf{R}(m\omega_j)\begin{bmatrix} q_{2j} \\ q_{2j+1} \end{bmatrix}
$$

## 相对位置

利用旋转矩阵的正交性与角度可加性 $\mathbf{R}(\alpha)^{\top}\mathbf{R}(\beta) = \mathbf{R}(\beta-\alpha)$（$\alpha,\beta$ 为任意角度），位置 $m$ 旋转后查询与位置 $n$ 旋转后键的第 $j$ 组点积贡献化简为：

$$
\left(\mathbf{R}(m\omega_j)\begin{bmatrix} q_{2j} \\ q_{2j+1} \end{bmatrix}\right)^{\top}
\left(\mathbf{R}(n\omega_j)\begin{bmatrix} k_{2j} \\ k_{2j+1} \end{bmatrix}\right)
= \begin{bmatrix} q_{2j} \\ q_{2j+1} \end{bmatrix}^{\top}
\mathbf{R}\left((n-m)\omega_j\right)
\begin{bmatrix} k_{2j} \\ k_{2j+1} \end{bmatrix}
$$

最终内积只取决于相对位置差 $n-m$ 而彻底消除绝对位置 $m$，实现了用绝对位置旋转编码显式表达相对位置。

## 远程衰减与长上下文扩展

多频段正余弦干涉使 RoPE 注意力分数上界总体随相对距离 $|n-m|$ 增大而自然衰减。当推理长度 $L_{\text{infer}}$ 超出预训练窗口 $L_{\text{train}}$ 时，低频分量的旋转角 $m\omega_j$ 落入未见过的角度区间导致失稳。长上下文扩展主要采用两类方法：

- **位置插值（PI）**：按比例缩放位置编号 $m' = m \cdot \frac{L_{\text{train}}}{L_{\text{infer}}}$ 将其压回预训练角度范围，再配合少量微调。
- **调大旋转底数与分频缩放（如 YaRN）**：大幅拉高 $\theta_{\text{base}}$ 延长低频周期，或只插值低频、保留高频局部区分能力，支持扩展至 128K 以上。
