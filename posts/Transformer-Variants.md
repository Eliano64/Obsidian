---
title: Transformer Variants
tags:
  - Transformer
categories:
  - Artificial Intelligence
date: 2026-11-11 12:20:00
katex: true
---

# 三种路线

[[Transformer-Architecture|Transformer]] 原为机器翻译设计；为服务更广泛的 [[Natural-Language-Processing|NLP]] 任务，后续衍生出三条代表架构路线，下表对比各自保留的拓扑与预训练目标：

| 路线代表 | 保留架构模块 | 注意力可见范围 | 预训练目标与主要定位 |
| --- | --- | --- | --- |
| **BERT** | 仅 Encoder | 全序列双向可见 | 掩码语言建模（MLM），主攻文本理解与表示提取 |
| **T5** | Encoder + Decoder | 编码器双向 + 解码器因果 | 全任务统一为 Text-to-Text 去噪填空生成 |
| **GPT** | 仅 Decoder | 单向因果掩码（只看前文） | 下一词元预测（NTP），主攻开放式自回归生成与大语言模型 |

![BERT、T5、GPT 三种架构|670](../assets/transformer-variants.svg)

# BERT

## 预训练与微调

传统有监督任务需要为每个下游场景独立标注并从头学起语言规律；**BERT** 率先将过程拆为「无标注语料自监督**预训练**（只训练一次通用主干）+ 少量标注数据下游**微调**（接任务头快速迁移）」。

![预训练与微调|820](../assets/pretrain-finetune.svg)

## 掩码语言模型

BERT 保留编码器执行双向上下文编码，通过**掩码语言建模（MLM）**自监督训练：随机选中约 15% 的词元（80% 替换为 `[MASK]`、10% 替换为随机词、10% 保持原样），让模型结合左右文恢复原始词元且只在该 15% 位置算损失。之所以不全替换为 `[MASK]`，是为了缓解推理真实文本中不含 `[MASK]` 的分布失配，并迫使模型也为未遮挡词元维持优质上下文表示。

![BERT 的掩码语言建模|675](../assets/bert-mlm.svg)

# T5

**T5（Text-to-Text Transfer Transformer）** 保留完整编码器与解码器，将分类、翻译、摘要与问答统统改写为带任务前缀（如 `summarize:`、`translate English to German:`）的文本输入到文本输出，从而免除多任务专用输出头，但需承担同时维护编解码器与交叉注意力的系统复杂度。

![T5 的文本到文本统一|675](../assets/t5-text-to-text.svg)

# GPT

**GPT** 仅保留解码器执行自回归**下一词元预测（NTP）**，随规模跃升演化出无须改写权重的通用能力：GPT-1 采用预训练加微调，GPT-2（15 亿参数）验证提示词续写做零样本迁移，GPT-3（175B 参数）确立**上下文学习（In-Context Learning）**范式。

![GPT 的上下文学习|664](../assets/gpt-in-context.svg)

> [!note] 上下文学习与微调的本质分界
> **微调**用下游标注更新模型参数，能力写入权重长期保留；**上下文学习**把指令与示例放进输入文本，完全不改变权重，仅在本次推理上下文窗口内生效。

## Scaling Law

当模型容量与数据互不卡脖时，同步或单独扩增模型参数量、训练数据规模与计算投入，语言建模损失呈稳定的幂律（Scaling Law）下降趋势，解释了规模推进后大模型涌现通用能力的来源。

![Scaling Law|675](../assets/scaling-law.svg)

## Prompt Engineering 与 Context Engineering

- **Prompt Engineering**：研究如何在不改动权重的前提下编排静态指令、少样本示例筛选排序与输出格式约束。
- **Context Engineering**：面向多步执行的 Agent 系统，每轮行动前动态检索外部资料、工具定义与精简历史状态拼入上下文窗口，决定模型当前行动所能感知的全部信息边界。

# Decoder-Only 路线的架构优势

当前大语言模型普遍收敛到 Decoder-Only（GPT）路线，主因有三点：

1. **监督密度高且预训练目标与推理零失配**：长 $N$ 文本的连续 $N-1$ 个位置全部参与 NTP 损失计算（高于 BERT 仅 15% 采样位置）；所有网页、代码与对话只需同一“续写下一词”目标，不像 MLM/填空式存在预训练“完形”与推理“续写”差异。
2. **统一提示接口降低任务切换门槛**：新需求先靠改写上下文试跑，不必每换任务重标数据与换头重训（上下文窗口受限或领域特化时再辅以微调）。
3. **单一堆栈极致适配系统优化**：重复堆叠同构因果解码器层更利于分布式训练切分，推理期围绕逐词生成可直接复用 KV Cache、连续批处理与显存调度技术。

![GPT 路线的三个优势|675](../assets/gpt-advantages.svg)

%%# 现代大模型的后训练与高效微调

经 NTP 预训练得到的**基座模型**具备语言知识，但尚未适配对话指令与偏好约束，因此通常接两步后训练：

1. **监督微调（SFT）**：用高质量“指令-回复”对话数据继续做自回归训练（只对回复部分算损失），使模型学会遵循指令与格式。
2. **偏好对齐（RLHF / DPO）**：利用同一输入下的优、劣回答对，让输出更贴合人类偏好。RLHF 通过奖励模型与强化学习优化策略；DPO 则跳过显式奖励模型，直接在偏好对上用对比目标更新权重。

全量微调成本高昂，工程上常用 **LoRA（Low-Rank Adaptation）**：冻结原始权重 $\mathbf{W}_0 \in \mathbb{R}^{d_{\text{in}} \times d_{\text{out}}}$，在旁路并联低秩矩阵 $\mathbf{A} \in \mathbb{R}^{d_{\text{in}} \times r}$ 与 $\mathbf{B} \in \mathbb{R}^{r \times d_{\text{out}}}$（秩 $r \ll \min(d_{\text{in}}, d_{\text{out}})$），记输入行向量 $\boldsymbol{x} \in \mathbb{R}^{1\times d_{\text{in}}}$、输出行向量 $\boldsymbol{h} \in \mathbb{R}^{1\times d_{\text{out}}}$、缩放系数 $\alpha$：

$$
\boldsymbol{h} = \boldsymbol{x}\mathbf{W}_0 + \frac{\alpha}{r}\boldsymbol{x}\mathbf{A}\mathbf{B}
$$

训练初始令 $\mathbf{A}$ 随机高斯初始化、$\mathbf{B}$ 全零初始化，保证起点增量 $\mathbf{A}\mathbf{B}=\mathbf{0}$ 而不破坏预训练底座输出；训练期间只更新少量旁路参数；推理时既可将增量合并回 $\mathbf{W}_0$ 消除额外延迟，也可保持分离以在同一底座上服务多组适配器（见 [[vLLM#前缀缓存|LoRA 服务]]）。%%
%%
# 稀疏架构扩展：混合专家模型（MoE）

**混合专家模型（MoE）** 将每层的前馈网络替换为 $E$ 个独立专家，路由器为每个 Token 仅激活得分最高的 $K$ 个专家加权求和（例如每步只启用 8 个专家中的 2 个）。这一结构将“模型总容量（总参数）”与“单步计算开销（激活参数）”解耦，推理更快，但训练需加负载均衡约束防止少数专家过载，多卡部署对应 [[vLLM#并行|专家并行]]。%%
