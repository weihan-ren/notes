---
title: "VAPO：基于价值的增强 PPO 推理优化"
created: 2026-06-25
updated: 2026-06-25
sources:
  - https://arxiv.org/abs/2504.05118
  - https://github.com/verl-project/verl
has_toc: true
tags: [VAPO, PPO, RL, ByteDance, PostTraining, Value-Model]
category: ai
---

# VAPO：基于价值的增强 PPO 推理优化

## 摘要

VAPO（Value-based Augmented Proximal Policy Optimization）是字节跳动 2025 年 4 月发布的推理 RL 算法。在 Qwen-32B 基座模型上达到 **AIME 2024 60.4 分**，超过 DeepSeek-R1-Zero 和 DAPO 各 10+ 分。核心创新：在 GRPO 主导的"去 Critic 化"浪潮中，VAPO **重新引入价值模型**——但用系统性设计解决了价值模型在长推理链中的三个核心挑战。

---

## 1. 背景：为什么要"反潮流"重新引入 Critic

2024-2025 年的主流趋势是**消除 Critic（价值模型）**：

| 方法 | Critic | 理由 |
|------|:-----:|------|
| GRPO | ❌ | 群组归一化替代 |
| REINFORCE++ | ❌ | 均值基线替代 |
| DAPO | ❌ | 保持 GRPO 简洁 |

VAPO 的立场：Critic 被消除是因为**用不好**，而不是**不该用**。如果能解决价值模型的三个核心问题，Critic 提供的逐 token 价值信号对长推理链训练有根本性优势。

---

## 2. 三个核心挑战与 VAPO 的解决方案

### 挑战 1：价值模型偏差（Value Model Bias）

**问题**：Critic 是学习的函数，它的估计本身有误差。在长推理链中，误差会逐 token 累积。

**VAPO 的解法**：
- 用 **GAE（广义优势估计）** 而非一步 TD 误差来平滑偏差
- 引入价值模型的正则化训练（防止过拟合到噪声奖励）

### 挑战 2：异构序列长度（Heterogeneous Sequence Lengths）

**问题**：推理链长度差异巨大——有的 500 token，有的 5000 token。Critic 需要在不同长度的序列上工作。

**VAPO 的解法**：
- **长度感知的价值归一化**——按序列长度分组处理
- 短序列和长序列的价值估计分别归一化后统一训练

### 挑战 3：奖励稀疏（Reward Sparsity）

**问题**：推理任务中，奖励只在最终答案出现——中间的数千个 token 没有直接信号。Critic 需要从稀疏的终端奖励中学习逐 token 的价值估计。

**VAPO 的解法**：
- 结合 **过程奖励信号**（如 format reward、step verification）
- 在关键中间步骤插入辅助奖励（如 `<think>` 和 `</think>` 的边界奖励）
- 使用更长的 GAE λ 来传播终端奖励到早期步骤

---

## 3. VAPO 架构

```
┌──────────────────────────────────────────┐
│              VAPO 训练架构                │
│                                          │
│  Actor (策略)        Critic (价值)        │
│  Qwen-32B-Base       Qwen-32B (共享骨干)  │
│       │                    │              │
│       │  GAE 优势估计 ←───  V(s_t)        │
│       │       │                           │
│       ▼       ▼                           │
│  PPO-Clip 更新     价值损失更新            │
│                                          │
│  三个关键增强：                            │
│  1. 长度感知价值归一化                     │
│  2. 辅助中间步骤奖励                       │
│  3. 长 GAE λ 信用传播                     │
└──────────────────────────────────────────┘
```

---

## 4. VAPO vs 其他方法

| | PPO (经典) | GRPO | DAPO | **VAPO** |
|---|---|---|---|---|
| **Critic** | ✅ 需要 | ❌ | ❌ | ✅ 需要 |
| **AIME 2024 (32B)** | - | ~40% | 50% | **60.4** |
| **训练步数** | - | 基准 | 减半 | **5000 步** |
| **训练稳定性** | 中等 | 中等 | 较好 | **极高（零崩溃）** |
| **核心创新** | TRPO→PPO | 群组归一化 | Clip-Higher | **长度感知 Value Model** |

---

## 5. 核心洞察

> VAPO 证明了 Critic 不是被淘汰了，而是需要更精细的设计。在当前群组归一化主导的时代，VAPO 是唯一坚持 Value-based 路线并取得 SOTA 的方法。

**关键教训**：
1. Token 级价值信号对长推理链有根本性优势——群组归一化丢失了逐步信息
2. GAE + 辅助奖励可以解决 reward sparsity
3. 长度感知的价值归一化是处理异构序列的关键
4. 5000 步达到 SOTA，训练极快且稳定

---

## 更新记录

- 2026-06-25：初始创建
