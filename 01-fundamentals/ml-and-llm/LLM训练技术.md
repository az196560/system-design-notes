# LLM 训练技术

---

## LLM 训练三阶段

```
阶段 1：预训练（Pre-training）
  海量无标注文本 → 自回归语言建模 → 基础模型（Base Model）

阶段 2：有监督微调（SFT）
  高质量指令-回答对 → 有监督训练 → 指令跟随模型

阶段 3：对齐（Alignment）
  人类偏好数据 → RLHF / DPO → 对齐模型（Chat Model）
```

---

## 有监督微调（Supervised Fine-Tuning, SFT）

**目标：** 教会 Base Model 遵循指令、对话、回答问题。

**数据格式：**
```json
{
  "messages": [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "解释量子力学"},
    {"role": "assistant", "content": "量子力学是研究微观粒子行为的物理学分支..."}
  ]
}
```

**训练方式：** 只对 assistant 回复部分计算 Loss，user/system 部分不计算（避免学习模仿用户）。

**数据质量 >> 数据数量：** 研究表明，1000 条高质量 SFT 数据就能显著提升模型的指令跟随能力（LIMA 论文）。

---

## RLHF（Reinforcement Learning from Human Feedback）

### 流程

```
1. 收集偏好数据：给同一 Prompt 生成多个回答，人工标注哪个更好
   Prompt → Response A (preferred) > Response B (rejected)

2. 训练奖励模型（Reward Model, RM）：
   - 输入：(Prompt, Response)
   - 输出：质量分数（标量）
   - 训练目标：preferred 的分数 > rejected 的分数

3. PPO 强化学习优化：
   - 用奖励模型作为信号
   - 优化策略使模型生成高奖励的回答
   - KL 惩罚：限制模型不能偏离 SFT 模型太远（防止"奖励黑客"）
```

### PPO（Proximal Policy Optimization）

```
Loss = -E[min(r(θ) × A, clip(r(θ), 1-ε, 1+ε) × A)] + β × KL(π_θ || π_ref)
```

- r(θ)：新旧策略的概率比
- A：优势函数（当前回答比平均好多少）
- clip：限制策略更新幅度，保证训练稳定
- KL 惩罚：防止偏离参考模型太远

**RLHF 的问题：**
- 流程复杂：需要训练 4 个模型（SFT、RM、Policy、Reference）
- 训练不稳定：RL 调参困难
- 奖励黑客（Reward Hacking）：模型学会欺骗奖励模型而非真正提升质量

---

## DPO（Direct Preference Optimization）

**核心思想：** 绕过奖励模型和 RL，直接用偏好数据优化语言模型。

```
Loss = -E[log σ(β × (log π_θ(y_w|x)/π_ref(y_w|x) - log π_θ(y_l|x)/π_ref(y_l|x)))]
```

- y_w：preferred 回答，y_l：rejected 回答
- π_θ：当前模型，π_ref：参考模型（SFT 后的模型）
- 本质：增加 preferred 回答的概率，降低 rejected 回答的概率

**DPO vs RLHF：**

| 维度 | RLHF | DPO |
|---|---|---|
| 复杂度 | 高（4 个模型） | 低（2 个模型） |
| 训练稳定性 | 需要精细调参 | 更稳定 |
| 效果 | 理论上更强 | 接近 RLHF |
| 工程实现 | 复杂 | 简单，类似 SFT |

**变体：** KTO（只需要好/坏二元标注，不需要成对对比）、ORPO（合并 SFT 和偏好优化为一步）。

---

## 参数高效微调（PEFT）

全参数微调大模型成本极高（70B 模型需要数百 GB 显存）。PEFT 只训练少量新增参数。

### LoRA（Low-Rank Adaptation）

**核心思想：** 冻结原始权重 W，用两个低秩矩阵 A 和 B 的乘积来近似权重更新 ΔW。

```
W' = W + ΔW = W + B × A

W: d × d（原始权重，冻结）
A: d × r（可训练，r << d）
B: r × d（可训练）
可训练参数量: 2 × d × r（远小于 d × d）
```

**r（秩）的选择：** 通常 r = 8-64。r 越大表达能力越强，但参数越多。

**适配位置：** 通常对 Attention 的 Q、K、V、O 投影矩阵加 LoRA。

**QLoRA：** 将原始模型量化到 4-bit，只训练 LoRA 参数（FP16）。单张消费级 GPU（24GB）即可微调 70B 模型。

### Adapter

在 Transformer 每层中间插入小型 Adapter 模块（两层 MLP：降维 → 激活 → 升维），只训练 Adapter 参数。

### Prefix Tuning / Prompt Tuning

在输入前添加可学习的虚拟 Token（Soft Prompt），只优化这些虚拟 Token 的 Embedding。

| 方法 | 可训练参数比例 | 效果 | 复杂度 |
|---|---|---|---|
| 全参数微调 | 100% | 最优 | 高 |
| LoRA | ~0.1-1% | 接近全参数 | 低 |
| QLoRA | ~0.1-1%（模型 4-bit） | 接近 LoRA | 极低 |
| Adapter | ~1-5% | 较好 | 低 |
| Prompt Tuning | < 0.1% | 简单任务尚可 | 极低 |

---

## 数据配比与课程学习

**Data Mixture：** 预训练数据的来源配比直接影响模型能力（详见 Training Data Pipeline 笔记）。

**课程学习（Curriculum Learning）：** 训练不同阶段使用不同的数据配比：
- 前期：大量 Web 数据，学习语言基础
- 中期：增加 Code 和 Math 数据，增强推理
- 后期（Annealing）：高质量数据，学习率衰减

**Long Context Training：** 先用短序列训练（如 4K），后期逐步增加到长序列（如 128K）。长序列训练需要 RoPE 频率调整（如 NTK-Aware Scaling）。

---

## 小结

> LLM 训练技术的核心是 **SFT → RLHF/DPO 的对齐流程和 LoRA 参数高效微调**。面试高频题：RLHF 的三步流程、DPO 如何绕过奖励模型、LoRA 低秩分解的原理和秩的选择、QLoRA 如何在消费级 GPU 上微调大模型、SFT 数据质量 vs 数量的权衡。
