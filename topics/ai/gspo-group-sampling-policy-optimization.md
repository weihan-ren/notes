---
title: "GSPO：群组采样策略优化"
created: 2026-06-25
updated: 2026-06-25
sources:
  - https://qwen.ai/blog?id=gspo
  - https://arxiv.org/abs/2511.20347
has_toc: true
tags: [GSPO, GRPO, RL, Alibaba, Qwen, PostTraining]
category: ai
---

# GSPO：群组采样策略优化

## 摘要

GSPO（Group Sampling Policy Optimization）是阿里 Qwen 团队在 GRPO 基础上提出的序列级裁剪变体。核心创新：将 GRPO 的 Token 级硬裁剪**升级为序列级硬裁剪**——整条序列共享一个 importance ratio，避免 token 级裁剪带来的不连续梯度问题。GSPO 是 SAPO 的前身，也是 Qwen3 系列模型 RL 训练的基础算法。

---

## 1. Token 级 vs 序列级裁剪

### 1.1 GRPO 的 Token 级问题

```
GRPO Token 级裁剪：
  序列中每个 token 独立计算 ρ_t 并独立裁剪

  问题场景：
  Token 1: ρ=1.05  → 在带内 → 全梯度
  Token 2: ρ=1.08  → 在带内 → 全梯度
  Token 3: ρ=1.35  → 超出上界 → 裁剪 → 零梯度
  Token 4: ρ=0.92  → 在带内 → 全梯度
  ...
  
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

---

## 2. GSPO 的核心思想

```
GRPO:  每个 token 是独立决策点
GSPO:  整条序列是一个连贯的决策整体

直觉：
  "这条推理链整体上有多好？"
  vs
  "第 47 个 token 有多好？"
```

**序列级视角的优势**：
1. **梯度一致性**：序列内所有 token 获得统一的梯度信号
2. **MoE 友好**：个别 token 的 ratio 异常不影响整条序列
3. **行为可解释**：OOD 检测更自然——"这条推理链偏离了吗？"

**序列级视角的代价**：
1. 个别 off-policy token 导致整条序列被丢弃（样本效率低）
2. 无法精确定位"哪个 step 出了问题"
3. 这正是 SAPO 要解决的问题

---

## 3. GSPO 在 Qwen3 中的应用

Qwen3 系列模型的 RL 后训练采用 GSPO 作为基础算法：

```
Qwen3-Base → SFT → GSPO RL → Qwen3 推理模型

GSPO 在 Qwen3 中证明了：
  - MoE 架构下比 GRPO 更稳定
  - 序列级信号天然适配链式推理任务
  - 配合 Qwen3-VL 的多模态训练
```

---

## 4. 从 GSPO 到 SAPO

GSPO 的局限性催生了 SAPO：

| | GSPO | SAPO |
|---|---|---|
| **裁剪方式** | 序列级硬裁剪 | 温度控制软门控 |
| **off-policy token** | 整条序列丢弃 | 仅抑制该 token |
| **样本效率** | 低（浪费整条序列） | 高（保留有效部分） |
| **MoE 稳定性** | 好 | **更好** |
| **非对称温度** | 无 | 有 |

---

## 5. 与其他方法的关系

```
PPO (2017)
  └── GRPO (2024, DeepSeek)
       ├── GSPO (2025, 阿里)
       │   序列级裁剪 → MoE 稳定性
       │   └── SAPO (2025.11, 阿里)
       │       软门控 + 非对称温度 → 连接 GRPO 和 GSPO
       │
       ├── DAPO (2025.03, 字节)
       │   非对称硬裁剪 + 动态采样
       │
       └── VAPO (2025.04, 字节)
           重新引入 Value Model → 逐 token 价值信号
```

---

## 更新记录

- 2026-06-25：初始创建
