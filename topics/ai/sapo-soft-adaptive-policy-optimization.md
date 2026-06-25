---
title: "SAPO：软自适应策略优化"
created: 2026-06-25
updated: 2026-06-25
sources:
  - https://arxiv.org/abs/2511.20347
  - https://www.alibabacloud.com/blog/sapo_602734
has_toc: true
tags: [SAPO, GRPO, GSPO, RL, Alibaba, Qwen, PostTraining]
category: ai
---

# SAPO：软自适应策略优化

## 摘要

SAPO（Soft Adaptive Policy Optimization）是阿里 Qwen 团队 2025 年 11 月提出的 RL 优化算法。用**温度控制的平滑门函数**替代 GRPO/GSPO 的硬裁剪，实现连续信任域 + 序列级一致性 + Token 级自适应 + 非对称温度控制。在 Qwen3-30B-A3B 数学推理和 Qwen3-VL 多模态任务上均超越 GRPO 和 GSPO。

---

## 1. 硬裁剪的三大问题

GRPO（Token 级硬裁剪）和 GSPO（序列级硬裁剪）共享一个根本缺陷：

```
硬裁剪 (Hard Clipping):

  ratio 在 [1-ε, 1+ε] 内 → 全梯度
  ratio 在 [1-ε, 1+ε] 外 → 零梯度 ──→ 梯度不连续
                                      ├── 信息丢失：好样本被浪费
                                      ├── 不稳定：裁剪带边界处梯度突变
                                      └── 难以调参：带太紧/太松都不行
```

**在 MoE 模型上尤其严重**：专家路由的变化导致 token 级 importance ratio 方差极大，硬裁剪频繁触发。

---

## 2. SAPO 的核心创新：平滑门函数

### 2.1 从硬裁剪到软门控

```
GRPO:  g(ratio) = 1  (在带内)  or  0  (在带外)     ← 硬边界
GSPO:  g(ratio_seq) = 1 or 0                       ← 序列级硬边界

SAPO:  g(ratio) = exp( -(ratio - 1)² / (2·T²) )    ← 平滑衰减！
                    ↑
              温度 T 控制衰减速度
```

**直观理解**：
- ratio 接近 1（on-policy）→ gate 接近 1 → 全梯度
- ratio 偏离 1（off-policy）→ gate 平滑衰减 → 部分梯度
- 永远不会完全切断梯度

### 2.2 非对称温度

SAPO 对正负优势使用**不同温度**：

```
T_pos < T_neg

正优势 (好动作): 温度低 → 衰减慢 → 鼓励探索好方向
负优势 (坏动作): 温度高 → 衰减快 → 快速抑制坏方向
```

**为什么非对称**：
- 负面优势需要增加大词汇表中大量不恰当 token 的 logits → 方差极大
- 高温使负面更新更快衰减 → 大幅提升训练稳定性
- 实验验证：反转温度关系（T_pos > T_neg）会导致训练崩溃

### 2.3 三大效果

| 效果 | 说明 |
|------|------|
| **序列级一致性** | 当序列内 token ratio 方差低时，SAPO 行为近似 GSPO——但用连续信任域替代硬边界 |
| **Token 级自适应** | 序列中个别 off-policy token 只被选择性抑制，不丢弃整条序列 |
| **样本效率提升** | 保留"部分 off-policy 但仍有学习价值"的样本的梯度 |

---

## 3. SAPO vs GSPO vs GRPO

| | GRPO | GSPO | **SAPO** |
|---|---|---|---|
| **裁剪方式** | Token 级硬裁剪 | 序列级硬裁剪 | **温度控制软门控** |
| **丢弃策略** | 单个 token | 整条序列 | **选择性抑制** |
| **信任域** | 不连续 | 不连续 | **连续** |
| **MoE 适配** | 差（ratio 方差大） | 较好 | **最优** |
| **序列一致性** | 无 | 有 | **有** |
| **非对称温度** | 无 | 无 | **有** |
| **训练稳定性** | 中等 | 中等 | 最高 |

---

## 4. 实验结果

| 实验 | 模型 | SAPO 表现 |
|------|------|----------|
| 数学推理 | Qwen3-30B-A3B | 稳定训练更长，AIME25/HMMT25/BeyondAIME Pass@1 均高于 GSPO 和 GRPO-R2 |
| 多模态 VL | Qwen3-VL 系列 | 在多种模型规模和架构（MoE/Dense）上一致提升 |
| 消融 | - | T_neg < T_pos 时崩溃，验证非对称温度的必要性 |

---

## 5. 在 GRPO 演进链中的位置

```
GRPO (2024) —— Token 级硬裁剪 + 群组归一化
  │
  ├── GSPO (2025, 阿里) —— 序列级硬裁剪
  │
  ├── SAPO (2025.11, 阿里) —— 软门控 + 非对称温度
  │   → 连接 GRPO 和 GSPO 的优势
  │
  └── IB-TPO (2026, ICML, 阿里) —— 信息瓶颈驱动的树策略优化
      → 将 SAPO 的稳定性理念扩展到树结构
```

---

## 更新记录

- 2026-06-25：初始创建
