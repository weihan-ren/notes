---
title: "GSPO / SAPO：群组采样与软自适应策略优化"
created: 2026-06-25
updated: 2026-06-27
sources:
  - https://qwen.ai/blog?id=gspo
  - https://arxiv.org/abs/2511.20347
  - https://www.alibabacloud.com/blog/sapo_602734
has_toc: true
tags: [GSPO, SAPO, GRPO, RL, Alibaba, Qwen, PostTraining]
category: ai
---

# GSPO / SAPO：群组采样与软自适应策略优化

## 摘要

**GSPO**（Group Sampling Policy Optimization）和 **SAPO**（Soft Adaptive Policy Optimization）是阿里 Qwen 团队在 GRPO 基础上提出的两个递进式改进。GSPO 将 Token 级裁剪升级为**序列级裁剪**，SAPO 进一步用**温度控制平滑门函数**替换硬裁剪，实现连续信任域。GSPO 是 SAPO 的前身，两者共同构成 Qwen3 系列 RL 训练的基础。

---

## 1. GSPO：从 Token 级到序列级裁剪

### 1.1 GRPO 的 Token 级问题

```
GRPO Token 级裁剪：
  序列中每个 token 独立计算 ρ_t 并独立裁剪

  问题场景：
  Token 1: ρ=1.05  → 在带内 → 全梯度
  Token 2: ρ=1.08  → 在带内 → 全梯度
  Token 3: ρ=1.35  → 超出上界 → 裁剪 → 零梯度
  Token 4: ρ=0.92  → 在带内 → 全梯度
  
  → 序列内部梯度不连续
  → 某些 token 被截断而相邻 token 正常更新
  → MoE 模型中尤其严重
```

### 1.2 GSPO 的序列级方案

```
GSPO 序列级裁剪：
  计算序列级 importance ratio：
    ρ_seq = (1/L) · Σ_t log(π_θ(a_t|s_t) / π_old(a_t|s_t))
  
  整条序列共享一个 ρ_seq：
    在带内 → 整条序列全梯度
    在带外 → 整条序列零梯度
  
  → 序列内部梯度一致
  → 训练更稳定
```

**GSPO 核心理念**：整条序列是一个连贯的决策整体，而非逐 token 独立决策。

**优势**：梯度一致性、MoE 友好、行为可解释  
**代价**：个别 off-policy token 导致整条序列被丢弃（样本效率低）→ 这正是 SAPO 要解决的

### 1.3 GSPO 在 Qwen3 中的应用

```
Qwen3-Base → SFT → GSPO RL → Qwen3 推理模型

GSPO 在 Qwen3 中证明了：
  - MoE 架构下比 GRPO 更稳定
  - 序列级信号天然适配链式推理任务
  - 配合 Qwen3-VL 的多模态训练
```

---

## 2. SAPO：从硬裁剪到软门控

### 2.1 硬裁剪的三大问题

GRPO（Token 级硬裁剪）和 GSPO（序列级硬裁剪）共享根本缺陷：

```
硬裁剪 (Hard Clipping):
  ratio 在 [1-ε, 1+ε] 内 → 全梯度
  ratio 在 [1-ε, 1+ε] 外 → 零梯度 ──→ 梯度不连续
                                      ├── 信息丢失：好样本被浪费
                                      ├── 不稳定：裁剪带边界处梯度突变
                                      └── 难以调参：带太紧/太松都不行
```

MoE 模型上尤其严重——专家路由的变化导致 ratio 方差极大，硬裁剪频繁触发。

### 2.2 SAPO 核心创新：平滑门函数

```
GRPO:  g(ratio) = 1 (在带内)  or  0 (在带外)     ← 硬边界
GSPO:  g(ratio_seq) = 1 or 0                     ← 序列级硬边界

SAPO:  g(ratio) = exp( -(ratio - 1)² / (2·T²) )  ← 平滑衰减！
                    ↑
              温度 T 控制衰减速度
```

直观理解：ratio 接近 1（on-policy）→ gate 接近 1 → 全梯度；ratio 偏离 1 → gate 平滑衰减 → 部分梯度；**永远不会完全切断梯度**。

### 2.3 非对称温度

```
T_pos < T_neg

正优势 (好动作): 温度低 → 衰减慢 → 鼓励探索好方向
负优势 (坏动作): 温度高 → 衰减快 → 快速抑制坏方向
```

反转温度关系（T_pos > T_neg）会导致训练崩溃。

### 2.4 三大效果

| 效果 | 说明 |
|------|------|
| **序列级一致性** | 当序列内 token ratio 方差低时，SAPO 近似 GSPO——但用连续信任域替代硬边界 |
| **Token 级自适应** | 个别 off-policy token 只被选择性抑制，不丢弃整条序列 |
| **样本效率提升** | 保留"部分 off-policy 但仍有学习价值"的样本的梯度 |

---

## 3. 三者对比

| | GRPO | GSPO | **SAPO** |
|---|---|---|---|
| **裁剪方式** | Token 级硬裁剪 | 序列级硬裁剪 | **温度控制软门控** |
| **丢弃策略** | 单个 token | 整条序列 | **选择性抑制** |
| **信任域** | 不连续 | 不连续 | **连续** |
| **MoE 适配** | 差 | 较好 | **最优** |
| **非对称温度** | 无 | 无 | **有** |
| **训练稳定性** | 中等 | 中等 | 最高 |

---

## 4. 实验结果

| 实验 | 模型 | 表现 |
|------|------|------|
| 数学推理 | Qwen3-30B-A3B | SAPO > GSPO > GRPO-R2，AIME25/HMMT25/BeyondAIME Pass@1 均领先 |
| 多模态 VL | Qwen3-VL 系列 | 多种模型规模和架构上一致提升 |
| 消融 | - | T_neg < T_pos 时崩溃，验证非对称温度的必要性 |

---

## 5. 在 GRPO 演进链中的位置

> 完整演进链见 [Agent RL 基础概念全景](agent-rl-llm-post-training.md#8-完整概念地图)

```
GRPO (2024) —— Token 级硬裁剪 + 群组归一化
  │
  ├── GSPO (2025, 阿里) —— 序列级硬裁剪
  │
  ├── SAPO (2025.11, 阿里) —— 软门控 + 非对称温度
  │   → 连接 GRPO 和 GSPO 的优势
  │
  └── IB-TPO (2026, ICML, 阿里) —— 信息瓶颈驱动树策略优化
      → 将 SAPO 稳定性理念扩展到树搜索结构
```

---

## 更新记录

- 2026-06-25：初始创建 GSPO 和 SAPO 独立笔记
- 2026-06-27：合并 GSPO 到 SAPO，消除重复内容，添加交叉引用
