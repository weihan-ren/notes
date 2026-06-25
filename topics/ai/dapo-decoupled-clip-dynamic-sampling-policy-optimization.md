---
title: "DAPO：解耦裁剪 + 动态采样策略优化"
created: 2026-06-25
updated: 2026-06-25
sources:
  - https://arxiv.org/abs/2503.14476
  - https://dapo-sia.github.io/
  - https://github.com/verl-project/verl
has_toc: true
tags: [DAPO, GRPO, RL, ByteDance, PostTraining]
category: ai
---

# DAPO：解耦裁剪 + 动态采样策略优化

## 摘要

DAPO（Decoupled Clip and Dynamic Sampling Policy Optimization）是字节跳动 Seed + 清华 AIR 联合提出的 GRPO 改进算法。在 Qwen2.5-32B 基座模型上达到 **AIME 2024 50% 准确率**，超越 DeepSeek-R1-Zero-Qwen-32B，且仅需 50% 的训练步数。核心创新：解耦裁剪 + 动态采样过滤 + Token 级策略梯度。

---

## 1. 背景：GRPO 的熵坍缩问题

DAPO 团队在复现 DeepSeek-R1-Zero 时发现了一个关键问题——**熵坍缩（Entropy Collapse）**：

```
正常 GRPO 训练：
  G=16 条轨迹，群组归一化，策略梯度更新

观察到的现象：
  → 训练过程中，策略的 token 分布越来越"尖锐"
  → logits 熵持续下降
  → 模型失去多样性，陷入局部最优
  → 性能在达到平台期后不再提升
```

**根因**：GRPO 的 `clip(r_t, 1-ε, 1+ε)` 中，上界 `1+ε` 限制了策略对于"好动作"的探索——模型无法充分增加那些有潜力的 token 的概率，导致过早收敛。

---

## 2. 四大核心创新

### 2.1 Clip-Higher：解耦裁剪

DAPO 的核心创新——**将裁剪的上界和下界解耦**：

```
GRPO:  clip(r_t, 0.8, 1.2)    ← 上下界对称
DAPO:  clip(r_t, ε_low, ε_high)  ← 上下界独立可调

典型设置：ε_low = 0.2, ε_high = 0.28
```

**为什么这很重要**：
- 上界更大 → 允许策略对好动作给予更强奖励 → **促进多样性，防止熵坍缩**
- 下界不变 → 对坏动作的惩罚保持保守 → 不引入额外噪声
- 本质上是一种 **非对称信任域**——鼓励探索好方向，保持对坏方向的警惕

```
可视化：

GRPO (对称裁剪):
  r_t=0.5  → clip→0.8  (强制拉回)
  r_t=1.5  → clip→1.2  (截断，丢失探索信号)
  r_t=2.0  → clip→1.2  (截断)
              ↑ 好动作的梯度被限制

DAPO (非对称裁剪):
  r_t=0.5  → clip→0.8  (同 GRPO)
  r_t=1.5  → 1.5       (保留！更大的探索空间)
  r_t=2.0  → min(2.0, ε_high) → 保留更多
              ↑ 好动作获得更强的正向信号
```

### 2.2 Dynamic Sampling：动态采样过滤

**问题**：GRPO 对每个 prompt 固定生成 G 条轨迹，但：
- 某些 prompt 的全部 G 条轨迹都正确 → 零梯度，浪费计算
- 某些 prompt 的全部 G 条轨迹都错误 → 零梯度（归一化后全是零），同样浪费

**DAPO 的解决方案**：
```
对每个 batch：
  1. 过滤掉 accuracy=1.0 的 prompt（全对 → 无学习信号）
  2. 过滤掉 accuracy=0.0 的 prompt（全错 → 无比较基准）
  3. 重新采样，保持 batch 中有效 prompt 数量恒定
```

**效果**：每个 batch 中的梯度信号都来自"有学习价值"的 prompt，训练效率大幅提升。

### 2.3 Token-Level Policy Gradient Loss

GRPO 默认使用序列级损失（整条轨迹一起打分）。DAPO 采用 **Token 级策略梯度损失**：

- 每个 token 独立计算策略梯度
- 更适合长链推理（Long-CoT）场景——模型需要知道"哪一步错了"
- 与 Tree-GRPO 的步级信号理念一致，但实现更简单

### 2.4 Overlong Reward Shaping：超长奖励塑形

**问题**：GRPO 训练中，模型倾向于生成越来越长的回答（长回答在群组比较中占优）。

**DAPO 的解法**：
- 对超出期望长度的回答施加软惩罚
- 不是直接截断（会丢失信息），而是**渐进式惩罚**
- 控制生成长度，同时保留推理能力

---

## 3. 实验结果

| 方法 | 基座模型 | AIME 2024 | 训练步数 |
|------|---------|:---------:|:-------:|
| DeepSeek-R1-Zero | Qwen-32B | ~40% | 基准 |
| **DAPO** | Qwen2.5-32B | **50%** | 50% 步数 |
| VAPO | Qwen-32B | 60.4 | 5000 步 |

**关键指标**：
- 128 H20 GPU 从头训练
- 开源：代码 + 数据集 + 验证器 + 模型权重 + 训练脚本
- 基于 VeRL 框架实现

---

## 4. DAPO vs GRPO vs VAPO

| | GRPO | DAPO | VAPO |
|---|---|---|---|
| **裁剪** | 对称硬裁剪 | 非对称硬裁剪 | Value-based（不用 ratio clip） |
| **采样** | 固定 G 条 | 动态过滤 | 固定 G 条 |
| **Critic** | 不需要 | 不需要 | **需要 Value Model** |
| **AIME 2024 (32B)** | ~40% | 50% | 60.4 |
| **开源** | DeepSeek | ✅ | ❌ |
| **训练步数** | 基准 | **减半** | 5000 步 |

---

## 5. 在 GRPO 演进链中的位置

```
GRPO (2024, DeepSeek)
  ├── DAPO (2025.03, 字节+清华)
  │   Clip-Higher + Dynamic Sampling + Token-Level Loss
  │   → 解决熵坍缩，AIME 50 (32B)
  │
  ├── VAPO (2025.04, 字节)
  │   Value-based Augmented PPO
  │   → 重新引入 Critic，AIME 60.4 (32B)
  │
  └── SAPO (2025.11, 阿里)
      Soft Adaptive 替代硬裁剪
      → 更稳定的策略优化
```

---

## 更新记录

- 2026-06-25：初始创建
