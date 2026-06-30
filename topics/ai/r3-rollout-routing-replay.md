---
title: "R3（Rollout Routing Replay）：MoE RL 训练稳定性方法"
created: 2026-06-30
updated: 2026-06-30
sources:
  - https://arxiv.org/abs/2510.11370
  - https://arxiv.org/html/2510.11370v2
tags: [R3, Rollout-Routing-Replay, MoE, RL-Stability, GRPO, Training-Collapse, Router-Alignment]
category: ai
---

# R3（Rollout Routing Replay）详解

## 摘要

R3（Rollout Routing Replay）由北京大学 + 小米于 2025 年 10 月提出，解决 **MoE 模型 RL 训练中因路由器不一致导致的训练崩溃**。核心思想：**记录推理时路由器选择了哪些专家，训练时原样重放**，消除训练-推理间的路由差异。

---

## 1. 问题：MoE RL 为什么崩溃？

### 1.1 训练-推理分离架构

现代 RL 框架使用不同的引擎：

```
推理阶段（Rollout）
  SGLang / vLLM  → 生成回答
  路由器选择专家：E2, E5, E1 ...

训练阶段（Update）
  Megatron / FSDP  → 计算梯度更新
  路由器选择专家：E2, E4, E1 ...  ← 可能不同！
```

**同一个 token，推理和训练阶段可能被路由到不同的专家！**

### 1.2 量化分析

论文对 Qwen3-30B-A3B（MoE）和 Qwen3-8B（Dense）的对比：

| 指标 | Dense (Qwen3-8B) | MoE (Qwen3-30B-A3B) | MoE + R3 |
|------|------------------|---------------------|----------|
| KL 散度 (训练/推理) | 6.4×10⁻⁴ | 1.5×10⁻³（**2.4 倍**） | 7.5×10⁻⁴ |
| 极端 token 比例 (τ>2) | 低 | **高一个数量级** | 接近 Dense |
| 路由器差异比例 | - | **~10% 的路由器选择不同专家** | - |
| 至少一层路由不同的 token | - | **94%** | - |

### 1.3 训练崩溃模式

RL 训练进行到一定步数后，KL 散度和极端 token 比例突然飙升，训练随即崩溃（reward 暴跌，梯度异常）。

---

## 2. R3 方法

### 2.1 核心思想

```
标准 RL 训练：
  Rollout 阶段: x → Router → I_infer（选择专家 2, 5）→ 生成 y
  Train 阶段:   x → Router → I_train（选择专家 2, 4）→ 计算 loss  ← 不一致！

R3:
  Rollout 阶段: x → Router → I_infer → 生成 y  +  保存 I_infer ▾
  Train 阶段:   x → Router → 使用 ▾ I_infer（而非重新选择）→ 计算 loss
```

**训练时不用自己的路由器选专家，而是直接复用推理阶段保存的路由决策。**

### 2.2 技术细节

在训练的前向过程中：

```
标准 MoE 层：
  s = x · W_r              # 路由器 logits
  I_train = TopK(s, k=2)   # 训练时重新选专家
  y = Σ g_i · E_i(x)       # 加权合并输出

R3 修改后：
  s = x · W_r              # 仍计算 logits（保留梯度流）
  I_infer = 从推理阶段加载  # 使用推理时的 mask
  y = Σ g_replay_i · E_i(x) # 按推理选择合并
  其中 g_replay_i = I_infer_i · exp(s_i) / Σ
```

**关键**：只重放 mask（哪些专家被选），不重放 gating weights（权重仍由训练 logits 计算），保持梯度流畅通。

### 2.3 Mask 缓存（多轮 Agent 场景）

推理引擎常用 KV Cache 避免重复 prefill。R3 的路由 mask 同样可以缓存：

- 同一前缀 token 的路由结果相同 → 缓存 mask 到 KV Cache 旁边
- 多轮 Agent 对话中，直接复用缓存 mask，无需重新计算
- 额外开销 < 3%

---

## 3. 实验结果

### 3.1 数学推理（Qwen3-30B-A3B）

| 方法 | Avg 得分 | 是否崩溃 |
|------|---------|---------|
| GRPO | 48.84 | **120 步崩溃** |
| GSPO | 66.76 | 不崩溃 |
| **GRPO + R3** | **68.05** | 不崩溃 |
| **GSPO + R3** | **69.00** | 不崩溃 |
| GRPO + TIS | 66.24 | **105 步崩溃** |
| **GRPO + R3** | **71.83** | 不崩溃 |

### 3.2 SWE-bench（代码 Agent）

| 方法 | Pass@1 | 崩溃 |
|------|--------|------|
| GRPO | 31.80 | **90 步崩溃** |
| **GRPO + R3** | **38.60** | 不崩溃 |

### 3.3 训练动态

R3 带来的改善：
- **梯度范数**：始终更低、更稳定
- **熵**：更早开始上升（更早探索好策略）
- **生成长度**：快速增长后稳定，而非剧烈波动
- **KL 散度**：保持 < 10⁻⁴，远低于崩溃阈值

---

## 4. 与知识库中概念的关系

### 4.1 R3 vs GSPO / SAPO

| 方法 | 解决什么问题 | 机制 |
|------|-------------|------|
| **GSPO** | GRPO 的 token 级重要性采样偏差 | 序列级 importance ratio |
| **SAPO** | 硬裁剪的不连续性 | 温度控制平滑门函数 |
| **R3** | MoE 路由器训练-推理不一致 | 记录并重放推理路由决策 |

**R3 与 GSPO/SAPO 正交**：可叠加使用（GSPO+R3 达到最高分 69.00）。

### 4.2 R3 vs TIS（截断重要性采样）

TIS 也尝试解决训练-推理偏差，但 R3 从根本上消除了**路由层面的不一致**。实验中 R3 比 TIS 高 5.58 分，且 R3+TIS 组合无额外收益（因为 R3 已消除 TIS 要解决的问题）。

### 4.3 与 Relax 框架的关系

Relax（arXiv:2604.11554，异步 RL 引擎）原生支持 R3，在 MoE 模型上仅有 1.9% 开销（对比 veRL 的 32% 性能退化）。

---

## 更新记录

- 2026-06-30：初始创建
