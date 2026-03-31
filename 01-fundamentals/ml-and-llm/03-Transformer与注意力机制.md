# Transformer 与注意力机制

---

## 注意力机制（Attention Mechanism）

**核心思想：** 给输入序列的不同位置分配不同的权重（注意力），让模型"关注"与当前任务最相关的部分。

### Self-Attention（自注意力）

每个 Token 与序列中所有 Token（包括自己）计算相关性：

```
Attention(Q, K, V) = softmax(QK^T / √d_k) × V
```

- **Q（Query）：** "我在找什么信息"
- **K（Key）：** "我有什么信息可以提供"
- **V（Value）：** "我实际提供的信息内容"
- **√d_k：** 缩放因子，防止点积值过大导致 Softmax 梯度消失

**计算流程：**
```
1. 输入 X 分别乘以 W_Q, W_K, W_V 得到 Q, K, V
2. Q × K^T 得到注意力分数矩阵（n × n）
3. 除以 √d_k 缩放
4. Softmax 归一化为注意力权重
5. 权重 × V 得到加权输出
```

**复杂度：** O(n²d)，n 是序列长度，d 是维度。序列越长，计算和显存开销越大。

### Multi-Head Attention（多头注意力）

将 Q, K, V 投影到 h 个不同的子空间，各自独立做 Attention，最后拼接：

```
MultiHead(Q, K, V) = Concat(head_1, ..., head_h) × W_O
head_i = Attention(QW_Q^i, KW_K^i, VW_V^i)
```

**为什么要多头？** 不同的 Head 可以关注不同类型的关系（如语法关系、语义关系、位置关系），增加模型的表达能力。

---

## Transformer 架构

### 原始 Transformer（Encoder-Decoder）

```
输入序列 → [Encoder × N] → 编码表示
                                ↓
目标序列 → [Decoder × N] → 输出序列
```

**Encoder Block：**
```
Input → Multi-Head Self-Attention → Add & LayerNorm → FFN → Add & LayerNorm → Output
```

**Decoder Block：**
```
Input → Masked Self-Attention → Add & LayerNorm
      → Cross-Attention (Q from Decoder, K/V from Encoder) → Add & LayerNorm
      → FFN → Add & LayerNorm → Output
```

**Masked Self-Attention：** Decoder 中，每个位置只能看到前面的 Token（因果掩码），防止"偷看"未来信息。

### 三种变体

| 变体 | 结构 | 代表模型 | 典型任务 |
|---|---|---|---|
| Encoder-Only | 只有 Encoder，双向注意力 | BERT | 文本分类、NER、相似度 |
| Decoder-Only | 只有 Decoder，因果注意力 | GPT、LLaMA、Claude | 文本生成、对话 |
| Encoder-Decoder | 完整结构 | T5、BART | 翻译、摘要 |

**当前主流：** Decoder-Only 架构统治 LLM 领域，因为自回归生成天然适合通用任务。

---

## FFN（前馈网络）

Transformer 中每个 Block 的另一个核心组件：

```
FFN(x) = W₂ × activation(W₁ × x + b₁) + b₂
```

- 通常 W₁ 将维度从 d_model 扩展到 4 × d_model，W₂ 再压缩回来
- 激活函数：GELU（GPT）或 SwiGLU（LLaMA）
- FFN 占 Transformer 参数量的约 2/3

**MoE（Mixture of Experts）：** 将单个 FFN 替换为多个"专家"FFN，每个 Token 只激活其中 Top-K 个专家。参数量大但实际计算量小。GPT-4、Mixtral 使用此架构。

---

## 位置编码（Positional Encoding）

Self-Attention 是排列不变的（Permutation Invariant），不包含位置信息。需要额外注入位置信息。

### 绝对位置编码

**正弦位置编码（Sinusoidal）：** 原始 Transformer，用不同频率的 sin/cos 函数编码位置，可泛化到未见过的长度。

**可学习位置编码（Learned）：** 每个位置学习一个向量（如 BERT、GPT-2）。问题：无法泛化到训练时未见过的位置。

### 相对位置编码

**RoPE（Rotary Position Embedding）：**
- 将位置信息编码为旋转矩阵，作用在 Q 和 K 上
- Attention 分数自然包含相对位置信息
- 支持长度外推（配合技术可扩展到更长序列）
- **LLaMA、Claude 等主流 LLM 的标配**

**ALiBi（Attention with Linear Biases）：** 在 Attention 分数上加一个与距离成正比的偏置项，距离越远惩罚越大。简单有效，天然支持长度外推。

| 方法 | 长度外推能力 | 代表模型 |
|---|---|---|
| Sinusoidal | 一般 | 原始 Transformer |
| Learned | 差 | BERT, GPT-2 |
| RoPE | 好（配合 NTK 插值等） | LLaMA, Claude |
| ALiBi | 好 | BLOOM, MPT |

---

## KV Cache（推理优化）

**问题：** 自回归生成时，每生成一个新 Token 都需要重新计算整个序列的 Attention。

**KV Cache：** 缓存已生成 Token 的 K 和 V 向量，新 Token 只需计算自己的 Q，与缓存的 K/V 做 Attention。

```
无 Cache：生成第 n 个 Token，计算 n × n 的 Attention → O(n²)
有 Cache：生成第 n 个 Token，计算 1 × n 的 Attention → O(n)
```

**显存开销：** KV Cache 随序列长度线性增长，长序列时占大量 GPU 显存（参见 LLM Serving 笔记）。

---

## 高效注意力机制

解决 Self-Attention 的 O(n²) 复杂度：

**FlashAttention：** 不改变数学计算，而是优化 GPU 内存访问模式（利用 SRAM 分块计算），减少 HBM 访问次数。速度提升 2-4 倍，显存减少 5-20 倍。**训练和推理的标配。**

**Multi-Query Attention (MQA)：** 所有 Head 共享同一组 K、V，只有 Q 是独立的。KV Cache 大小减少 h 倍。

**Grouped-Query Attention (GQA)：** MQA 和 MHA 的折中，将 Head 分组，每组共享 K、V。如 8 组各 4 个 Head 共享。LLaMA 2/3 使用。

| 方法 | KV Cache 大小 | 质量 | 代表模型 |
|---|---|---|---|
| MHA（标准多头） | h × d | 最优 | GPT-3, BERT |
| GQA（分组查询） | g × d（g < h） | 接近 MHA | LLaMA 2/3 |
| MQA（多查询） | 1 × d | 略低 | PaLM, Falcon |

---

## 小结

> Transformer 八股的核心是**Self-Attention 计算流程、位置编码、KV Cache**。面试高频题：Attention 的 QKV 含义和计算公式、为什么要除以 √d_k、Multi-Head 的作用、RoPE 原理、KV Cache 的工作机制、FlashAttention 和 GQA 的优化思路。这是 AI Infra 面试的理论基础。
