# LLM 基础

---

## 什么是大语言模型

大语言模型（Large Language Model）是基于 Transformer Decoder-Only 架构、在海量文本数据上预训练的自回归语言模型。核心能力是**给定上文，预测下一个 Token**。

**规模定义：** 参数量从数十亿（7B）到数千亿（400B+），训练数据从数万亿 Token 起步。

---

## Tokenization（分词）

将文本转换为模型可处理的 Token 序列。

### BPE（Byte Pair Encoding）

**训练过程：**
```
1. 初始词表 = 所有单个字符（或字节）
2. 统计训练语料中所有相邻 Token 对的频率
3. 合并频率最高的 Token 对为一个新 Token
4. 重复步骤 2-3 直到词表大小达到目标（如 32K/64K/128K）
```

**推理过程：** 贪心地用词表中最长的匹配来分词。

**优点：** 高频词完整保留，低频词拆分为子词（Subword），OOV（未登录词）问题自然解决。

### 常见分词器

| 分词器 | 算法 | 词表大小 | 使用模型 |
|---|---|---|---|
| GPT-2 Tokenizer | BPE | 50K | GPT-2/3 |
| tiktoken (cl100k) | BPE | 100K | GPT-4 |
| SentencePiece | BPE/Unigram | 32K-128K | LLaMA, T5 |

**Token 与字符的关系：** 英文约 1 Token ≈ 4 字符 ≈ 0.75 词。中文约 1 Token ≈ 1-2 字。

---

## 预训练（Pre-training）

### 训练目标

**自回归语言建模（Causal LM）：** 给定前 t-1 个 Token，预测第 t 个 Token。

```
P(x₁, x₂, ..., xₙ) = ∏ P(xₜ | x₁, ..., x_{t-1})
```

损失函数 = 负对数似然（Cross-Entropy），对每个位置的预测与真实 Token 计算交叉熵，取平均。

### 训练数据

| 数据来源 | 占比（典型） | 特点 |
|---|---|---|
| Web Crawl（CommonCrawl） | 60-70% | 量大、质量参差 |
| Books | 10-15% | 高质量长文本 |
| Code（GitHub） | 10-15% | 增强推理能力 |
| Wikipedia | 3-5% | 知识密集 |
| 学术论文 | 3-5% | 专业知识 |

**数据清洗：** 去重、语言过滤、质量评分、有害内容过滤（详见 Training Data Pipeline 笔记）。

### Scaling Law

**Chinchilla Scaling Law（2022）：** 在固定计算预算下，模型参数量和训练数据量应等比例扩大。

```
最优训练 Token 数 ≈ 20 × 参数量
```

即 7B 模型应训练 ~140B Token，70B 模型应训练 ~1.4T Token。

**实践中：** 很多模型训练远超 Chinchilla 最优值（如 LLaMA 3 用 15T Token 训练 8B 模型），因为推理成本 >> 训练成本，用更小模型 + 更多数据可以在推理时省成本。

### 涌现能力（Emergent Abilities）

某些能力（如 Chain-of-Thought 推理、In-Context Learning）只在模型达到一定规模后才"涌现"，小模型几乎不具备。

---

## 解码策略（Decoding Strategy）

模型输出的是下一个 Token 的概率分布，如何从中选择 Token：

### Greedy Decoding
每步选概率最高的 Token。简单但容易陷入重复。

### Top-K Sampling
从概率最高的 K 个 Token 中随机采样。K 控制多样性。

### Top-P Sampling（Nucleus Sampling）
从累计概率达到 P 的最小 Token 集合中采样。自适应：高置信时选择少，低置信时选择多。

### Temperature

```
P(xᵢ) = exp(logit_i / T) / Σ exp(logit_j / T)
```

- T < 1：分布更尖锐，倾向选高概率 Token（更确定）
- T > 1：分布更平坦，增加多样性（更随机）
- T = 0：等价于 Greedy Decoding

**常见组合：** Temperature 0.7 + Top-P 0.9，平衡质量和多样性。

### Beam Search
维护 B 个候选序列，每步扩展每个候选的 Top-K，保留总概率最高的 B 个。用于翻译等对质量要求高的任务，但不适合开放式生成。

---

## 上下文学习（In-Context Learning）

**Zero-Shot：** 直接给任务描述，不给示例。
```
Translate English to French: "Hello" →
```

**Few-Shot：** 给几个示例，让模型从示例中"学习"任务模式。
```
Translate English to French:
"Hello" → "Bonjour"
"Thank you" → "Merci"
"Good morning" →
```

**本质：** 模型并未更新参数，而是通过 Attention 机制在上下文中"检索"示例的模式并应用。

---

## Prompt Engineering 核心技巧

**Chain-of-Thought（CoT）：** 让模型先输出推理步骤，再给出最终答案。显著提升数学、逻辑推理能力。

**System Prompt：** 定义模型的角色和行为规范，放在对话最前面。

**结构化输出：** 要求模型输出 JSON / XML 等格式，便于下游程序解析。

---

## 幻觉（Hallucination）

**定义：** 模型生成看起来合理但实际上不正确或无依据的内容。

**类型：**
- **事实性幻觉：** 虚构不存在的事实、人物、论文
- **逻辑性幻觉：** 推理步骤看似合理但结论错误
- **忠实性幻觉：** 回答与给定上下文不一致

**缓解方法：**
- RAG（检索增强生成）：让模型基于检索到的事实回答
- 要求引用来源
- Self-Consistency：多次生成取多数投票
- 后处理验证（Fact-Checking）

---

## 小结

> LLM 基础八股的核心是**Tokenization、预训练目标、Scaling Law、解码策略**。面试高频题：BPE 分词过程、自回归 vs 自编码（GPT vs BERT）、Temperature 和 Top-P 的作用、Scaling Law 的含义、In-Context Learning 的原理、幻觉的成因和缓解方法。
