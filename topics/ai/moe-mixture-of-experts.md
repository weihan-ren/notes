---
title: "MoE（Mixture of Experts）混合专家模型详解"
created: 2026-06-30
updated: 2026-06-30
sources:
  - https://newsletter.maartengrootendorst.com/p/a-visual-guide-to-mixture-of-experts
  - https://arxiv.org/abs/1701.06538
  - https://arxiv.org/abs/2401.04088
  - https://arxiv.org/abs/2402.03300
tags: [MoE, Mixture-of-Experts, Sparse-MoE, Router, Load-Balancing, LLM-Architecture]
category: ai
---

# MoE（Mixture of Experts）详解

## 摘要

MoE（混合专家）是一种**稀疏激活**架构，用多个子模型（"专家"）替代 Transformer 中的单一 FFNN 层。每个 token 只激活少数专家，从而**用更少的计算量获得更大的模型容量**。核心组件：**专家（Experts）** + **路由器（Router/Gate）** + **负载均衡机制**。

---

## 1. 核心思想：从 Dense 到 Sparse

### 传统 Dense FFNN

```
输入 → FFNN（所有参数激活）→ 输出
       ↑ 参数量大，计算量大
```

传统 Transformer 的 FFNN 层是**稠密**的——所有参数都参与计算。以 Llama 70B 为例，FFNN 层占绝大部分参数。

### Sparse MoE

```
输入 → Router（选择专家）→ 专家1（激活）  → 加权合并 → 输出
                          → 专家2（激活）
                          → 专家3（休眠）
                          → 专家4（休眠）
       ↑ 只激活 2/4 个专家，计算量减半
```

**核心洞察**：不同 token 需要不同类型的信息处理。与其让一个巨大的 FFNN 处理所有 token，不如训练多个小型 FFNN（"专家"），让每个 token 只走最相关的几个。

---

## 2. 两大组件

### 2.1 专家（Experts）

每个专家是一个**完整的 FFNN**，替换原来的一层 FFNN：

```
Decoder Block
├── Self-Attention（不变）
├── LayerNorm
└── MoE Layer（替代原 FFNN）
    ├── Expert 1（FFNN）
    ├── Expert 2（FFNN）
    ├── ...
    └── Expert N（FFNN）
```

**关键纠正**：专家**不是**按领域（心理学、生物学）划分的。实际上专家学到的是**句法级别的模式**——某些专家偏好特定类型的 token（如动词、数字、标点）。

Mixtral 8x7B 论文中的 token 路由可视化显示了这一点：

```
Token:  "The"  "cat"  "sat"  "on"  "the"  "mat"
Expert:   E3     E1     E5     E3     E3     E2
           ↑ 语法驱动，非语义驱动
```

### 2.2 路由器（Router / Gate Network）

路由器本身也是一个 FFNN，输出每个专家的选择概率：

```
Router(x) = SoftMax(x · W_gate)
                ↓
           概率分布 G(x) = [0.05, 0.72, 0.15, 0.08]
                                    ↑ 选专家2
```

**完整流程**：

```
输入 x
  ↓
Router: H(x) = x · W_gate           # 计算路由分数
  ↓
KeepTopK: 只保留 top-2 的分数，其他置为 -∞
  ↓
SoftMax: G(x) = softmax(H(x))       # 转为概率分布
  ↓
专家计算: y_i = Expert_i(x)          # 被选中的专家处理输入
  ↓
加权合并: output = Σ G_i(x) · y_i   # 按路由权重合并
```

---

## 3. 负载均衡（Load Balancing）

MoE 最大的训练挑战：路由器可能**总是选择同一个专家**，导致其他专家得不到训练。

### 3.1 KeepTopK + 噪声

在路由分数上添加可训练的**高斯噪声**，然后只保留 Top-K：

```
H(x)_i = x·W_gate + ε · softplus(x·W_noise)
         路由分数     ↑ 可训练噪声

KeepTopK(H(x), k=2) → 只保留最高的 2 个，其他设为 -∞
SoftMax → 被 -∞ 的专家概率 = 0
```

### 3.2 辅助损失（Auxiliary Loss）

在训练损失中加入负载均衡约束：

```
L_aux = α · N · Σ_i (f_i · P_i)

其中：
  f_i = 分配给专家 i 的 token 比例
  P_i = 路由器分配给专家 i 的平均概率
  α   = 超参数（控制负载均衡强度）
```

目标：让 **f_i** 和 **P_i** 都接近 1/N（均匀分布）。

### 3.3 专家容量（Expert Capacity）

限制每个专家能处理的 token 数量：

```
Expert Capacity = (total_tokens / num_experts) × capacity_factor
                                                      ↑ 通常 1.0~1.5
```

- 容量太大 → 浪费计算
- 容量太小 → token 溢出（跳过该层，性能下降）

---

## 4. 稀疏参数 vs 激活参数

MoE 模型有两个参数量：

| 概念 | 含义 | Mixtral 8x7B 示例 |
|------|------|-------------------|
| **总参数（Sparse Params）** | 所有专家的参数之和 + 共享参数 | **46.7B**（8×5.6B + 共享） |
| **激活参数（Active Params）** | 每次推理实际使用的参数 | **12.9B**（2×5.6B + 共享） |

> ⚠️ MoE 需要**更多 VRAM** 来加载所有专家，但推理时**计算量更小**（只激活少量专家）。

---

## 5. 代表模型

| 模型 | 总参数 | 激活参数 | Top-K | 说明 |
|------|--------|---------|-------|------|
| **Mixtral 8x7B** | 46.7B | 12.9B | 2 | 开源 MoE 先驱，性能接近 Llama 70B |
| **Mixtral 8x22B** | 141B | 39B | 2 | 更大版本 |
| **DeepSeek-V2/V3** | 236B/671B | 21B/37B | 6-8 | DeepSeek 的 MoE 架构，极致稀疏 |
| **Qwen2.5-MoE** | 多规格 | - | - | 阿里 Qwen MoE 系列 |
| **GPT-4** | ~1.8T（传闻） | ~280B（传闻） | - | 传闻使用 MoE |

---

## 6. MoE 与知识库中概念的关系

### 6.1 GRPO/SAPO 与 MoE 的冲突

知识库中 SAPO 笔记提到：MoE 模型中 **token 级 importance ratio 方差极大**，因为专家路由的变化导致不同 token 的策略偏差很大。这使得：

- GRPO 的硬裁剪在 MoE 上频繁触发 → 训练不稳定
- GSPO 的序列级裁剪在 MoE 上更稳定
- SAPO 的软门控在 MoE 上最优

### 6.2 专家并行（EP）

概念字典中的「专家并行 (EP)」是 MoE 的分布式训练策略：不同专家放到不同 GPU 上，结合数据并行/张量并行形成 3D 并行（DP+TP+EP）。

### 6.3 Megatron-LM 的 MoE 优化

AReaL 笔记中提到 Megatron-LM 对 MoE 有专门优化，支持 EP + TP + PP + CP 全并行策略。

---

## 更新记录

- 2026-06-30：初始创建，覆盖 MoE 核心原理、Router、负载均衡、代表模型
