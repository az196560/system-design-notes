# LLM 推理优化

---

## 推理的两个阶段回顾

| 阶段 | 特点 | 瓶颈 | 指标 |
|---|---|---|---|
| Prefill | 并行处理所有输入 Token | GPU 计算（Compute-bound） | TTFT（Time to First Token） |
| Decode | 逐 Token 串行生成 | 显存带宽（Memory-bound） | TBT（Time Between Tokens） |

Decode 阶段每步只做 1 个 Token 的矩阵-向量乘法，GPU 算力利用率极低（通常 < 5%），瓶颈在于从 HBM 读取模型权重和 KV Cache 的带宽。

---

## KV Cache 优化

### PagedAttention

（详见 LLM Serving 笔记）将 KV Cache 按 Page 管理，消除显存碎片，利用率从 ~50% 提升到 ~95%。

### Multi-Query Attention (MQA) / Grouped-Query Attention (GQA)

减少 KV Cache 的大小：

```
标准 MHA: KV Cache = num_layers × 2 × num_heads × seq_len × head_dim
GQA:      KV Cache = num_layers × 2 × num_groups × seq_len × head_dim
MQA:      KV Cache = num_layers × 2 × 1 × seq_len × head_dim
```

GQA 将 Head 分组共享 KV，MQA 所有 Head 共享同一组 KV。在质量损失极小的情况下，KV Cache 缩小数倍。

### Prefix Caching

相同 System Prompt 的请求共享 KV Cache 前缀，避免重复计算。

```
请求 A: [System Prompt | User Query A]
请求 B: [System Prompt | User Query B]
          └─── 共享 ───┘
```

适用场景：API 服务中大量请求使用相同 System Prompt。

---

## 模型量化（Quantization）

将模型权重从高精度（FP16/BF16）压缩到低精度（INT8/INT4），减少显存占用和访存带宽。

### 量化方法

**Post-Training Quantization（PTQ）：** 训练后直接量化，不需要重新训练。

**Weight-Only Quantization：** 只量化权重，激活值保持 FP16。Decode 阶段是 Memory-bound，减少权重大小直接提升带宽利用率。

**Weight + Activation Quantization（W8A8）：** 权重和激活都量化到 INT8，可以使用 INT8 Tensor Core 加速计算。

### 常见量化方案

| 方案 | 精度 | 压缩比 | 质量损失 | 代表方法 |
|---|---|---|---|---|
| FP16/BF16 | 16-bit | 1x（基线） | 无 | 标准推理 |
| INT8 (W8A8) | 8-bit | 2x | 极小 | SmoothQuant, LLM.int8() |
| INT4 (W4A16) | 4-bit 权重 | 4x | 小 | GPTQ, AWQ |
| INT4 (W4A4) | 4-bit 全量化 | 4x+ | 中等 | QuIP# |

**GPTQ：** 基于二阶信息（Hessian）的逐层量化，最小化量化误差。需要校准数据集。

**AWQ（Activation-aware Weight Quantization）：** 发现少数"重要"权重（对应激活值大的通道）对量化敏感，保护这些通道用更高精度。

**SmoothQuant：** 将激活的量化难度"转移"到权重上（通过数学等价变换），使 W8A8 量化成为可能。

---

## Speculative Decoding（投机解码）

**核心思想：** 用小模型（Draft Model）快速生成 K 个候选 Token，再用大模型一次性验证。

```
1. Draft Model（小模型）自回归生成 K 个 Token（快）
2. Target Model（大模型）一次前向传播验证 K 个 Token（并行）
3. 从左到右检查：如果 Draft Token 被接受，保留；否则用 Target 的分布重新采样
4. 最好情况：一次验证通过 K 个 Token，等效加速 K 倍
5. 最坏情况：第 1 个就被拒绝，但至少得到 1 个正确 Token
```

**加速比：** 通常 2-3 倍。Draft 和 Target 模型越接近，接受率越高。

**变体：**
- **Self-Speculative：** 用大模型的早退（Early Exit）或跳层作为 Draft，无需额外模型
- **Medusa：** 给大模型加多个预测头，同时预测未来多个位置的 Token
- **Lookahead Decoding：** 基于 Jacobi 迭代的并行解码

---

## FlashAttention

**问题：** 标准 Attention 需要将 n×n 的注意力矩阵完整写入 GPU HBM（高带宽内存），显存和带宽开销巨大。

**FlashAttention 的解决方案：**
```
1. 将 Q, K, V 分成小块（Tile）
2. 每次只加载一个 Tile 到 SRAM（片上高速缓存，~20MB）
3. 在 SRAM 内完成 Attention 计算
4. 用 Online Softmax 算法避免需要全局 Softmax 分母
5. 结果直接写回 HBM，中间矩阵不需要存到 HBM
```

**效果：**
- 速度提升 2-4 倍
- 显存从 O(n²) 降到 O(n)
- 数学结果完全等价，不是近似

**FlashAttention-2/3：** 进一步优化 GPU 线程分配和 Warp 调度，提升硬件利用率。

---

## 模型并行推理

单张 GPU 放不下大模型时的拆分方案：

**Tensor Parallelism（TP）：** 将每一层的矩阵运算切分到多张 GPU 上并行计算。通信频繁（每层前向都要 AllReduce），要求 NVLink 高带宽连接。通常在同一节点内使用。

**Pipeline Parallelism（PP）：** 将模型按层分到不同 GPU，形成流水线。推理时 Token 依次经过各 GPU。通信量小但延迟增加。

**常见配置：**
- 70B 模型：2 × H100（TP=2）
- 400B+ 模型：8 × H100（TP=8）或跨节点 TP+PP

---

## 其他优化技术

**Continuous Batching：** 动态管理 Batch，已完成请求即时移出，新请求即时加入。吞吐提升 5-20 倍。（详见 LLM Serving 笔记）

**Kernel Fusion：** 将多个小算子合并为一个 GPU Kernel，减少 Kernel Launch 开销和 HBM 读写。如将 LayerNorm + Linear 融合为一个 Kernel。

**CUDA Graph：** 预先录制一系列 GPU 操作的执行图，推理时直接回放，消除 CPU-GPU 同步和 Kernel Launch 开销。Decode 阶段每步操作相同，特别适合 CUDA Graph。

---

## 优化技术总结

| 优化类别 | 技术 | 加速倍数 | 影响 |
|---|---|---|---|
| 显存效率 | PagedAttention, GQA | - | 更多并发请求 |
| 计算效率 | FlashAttention | 2-4x | 更快 Attention |
| 带宽效率 | 量化 (INT4/INT8) | 2-4x | 更快 Decode |
| 算法效率 | Speculative Decoding | 2-3x | 更快生成 |
| 批处理效率 | Continuous Batching | 5-20x | 更高吞吐 |
| 系统效率 | CUDA Graph, Kernel Fusion | 1.1-1.5x | 减少系统开销 |

---

## 小结

> LLM 推理优化的核心是**量化、KV Cache 管理、Speculative Decoding、FlashAttention**。面试高频题：量化的原理和 INT4/INT8 的 trade-off、KV Cache 为什么是瓶颈以及 GQA 如何缓解、Speculative Decoding 的流程和加速原理、FlashAttention 避免写入中间矩阵的思路。这些技术是 AI Infra 面试的硬核考点。
