---
title: "RLHF（Reinforcement Learning from Human Feedback）详解"
created: 2026-06-18
updated: 2026-06-18
sources:
  - https://arxiv.org/abs/1706.03741
  - https://arxiv.org/abs/2203.02155
  - https://arxiv.org/abs/1909.08593
  - https://arxiv.org/abs/1706.03741
  - https://arxiv.org/abs/2203.02155
  - https://arxiv.org/abs/1909.08593
  - https://arxiv.org/abs/2009.01325
tags: [RLHF, PPO, Reward-Model, Alignment, LLM, PostTraining]
category: ai
---

# RLHF（Reinforcement Learning from Human Feedback）详解

## 摘要

RLHF 是 LLM 对齐阶段最具影响力的技术框架。它通过"训练奖励模型 + PPO 强化学习优化"的三步流程，让语言模型学会生成符合人类偏好的输出。本文从起源、数学原理、工程实践和核心权衡四个维度系统梳理 RLHF。

---

## 1. 起源与演进

### 1.1 思想萌芽

RLHF 的基本思想最早由 OpenAI 的 **Christiano et al. (2017)** 在论文 *"Deep Reinforcement Learning from Human Preferences"* 中提出。最初的应用场景并非语言模型，而是 Atari 游戏和模拟机器人控制。

核心洞察：对于一个难以写出奖励函数的任务（如"做后空翻"），可以让人来比较两个行为轨迹，用这些偏好数据训练一个"奖励预测器"，再用 RL 来优化。

### 1.2 首次语言模型应用

**Ziegler et al. (2019)** 在 *"Fine-Tuning Language Models from Human Preferences"* 中首次将 RLHF 应用于语言模型。他们用 RLHF 训练模型在 Reddit 帖子上续写文本，发现经过偏好的 RL 微调后，模型续写质量显著优于纯监督微调。

### 1.3 ChatGPT 的诞生

真正让 RLHF 家喻户晓的是 OpenAI 的 **InstructGPT / ChatGPT**（Ouyang et al., 2022）。关键发现：

> 1.3B 参数的 InstructGPT 模型在人类偏好评分上优于 175B 的 GPT-3 基座模型。

这证明了**对齐比规模更重要**——大模型如果不会"听话"，再大也没用。

---

## 2. 三步流程

RLHF 的完整训练管线分为三个步骤：

```
步骤1: SFT           步骤2: RM 训练         步骤3: PPO 优化
(监督微调)          (训练奖励模型)        (强化学习对齐)

提示 → 理想回答      提示 + 两个回答 → 偏好     提示 → 策略生成 → RM 打分 → 更新策略
     ↓                    ↓                        ↓
  π_SFT               r_θ(x,y)                   π_RL (最终模型)
```

### 2.1 第一步：监督微调（SFT）

- 人工标注者撰写高质量的 `(提示, 理想回答)` 对
- 对基座模型做标准的有监督微调
- 产出：SFT 模型 `π_SFT`
- 数据量级：InstructGPT 使用了约 13,000 条人工撰写的示范

**为什么需要 SFT**：基座模型只会"续写"而不会"回答问题"。SFT 教会模型对话格式和基本指令遵循能力。

### 2.2 第二步：训练奖励模型（Reward Model, RM）

**RM 的定义**：一个函数 `r_θ(x, y) → ℝ`，输入（提示, 回答），输出一个标量分数，代表人类的偏好程度。

**训练数据**：偏好对比数据 `(x, y_w, y_l)`
- 对于同一个提示 `x`，标注者从多个模型生成的回答中选出"更好的"（`y_w`）和"更差的"（`y_l`）
- InstructGPT：约 33,000 个偏好对比
- Anthropic HH-RLHF：约 161,000 个偏好对比

**损失函数（Bradley-Terry 模型）**：
```
L(θ) = -E[log σ(r_θ(x, y_w) - r_θ(x, y_l))]
```
- `σ` 是 sigmoid 函数
- 训练目标：让 RM 给"好回答"的输出显著高于"差回答"

**RM 的规模**：通常比策略模型小
- InstructGPT：用 6B 的 RM 训练 175B 的策略模型
- 经验规则：RM 至少要有策略模型的 1/10 大小

**RM 的本质问题**：
- RM 是一个**有缺陷的偏好代理**——它只在训练数据上学到了偏好模式
- 策略可能学会"奖励欺骗"（reward hacking）：输出 RM 喜欢但人类不喜欢的内容
- 这引出了 KL 惩罚的必要性

### 2.3 第三步：PPO 强化学习优化

用 PPO 算法优化 SFT 模型，目标是**最大化 RM 打分，同时约束模型不要偏离太远**。

**优化目标**：
```
max E[ r_RM(x, y) - β · D_KL(π_RL || π_SFT) ]
```

**训练循环**：
1. 从数据集中采样提示 `x`
2. 当前策略 `π_RL` 生成回答 `y`
3. RM 打分 → `r_RM`
4. 计算 KL 惩罚 → `r_KL = log(π_RL(y|x) / π_SFT(y|x))`
5. 总奖励 = `r_RM - β × r_KL`
6. PPO 更新策略参数
7. 重复（参考模型 `π_SFT` 始终冻结）

---

## 3. KL 惩罚 —— RLHF 的核心约束

### 3.1 为什么必须加 KL 惩罚

```
总奖励 = RM得分 - β × KL(当前策略 || SFT策略)
```

**三个关键理由**：

1. **防止奖励欺骗（Reward Hacking）**：
   - RM 只在训练数据的分布上有效
   - 没有 KL 约束时，策略会找到 RM 的"盲点"——生成 RM 打高分但实则低质量的文本
   - KL 惩罚将策略"锚定"在合理语言分布附近

2. **防止灾难性遗忘（Catastrophic Forgetting）**：
   - 过度优化 RM 会让策略丧失预训练学到的语言能力和事实知识
   - KL 惩罚像一个"信任域"——允许进步，但不允许跑太远

3. **理论基础**：
   - Bai et al. (2022) 发现 RL 奖励与 √(KL 散度) 之间大致呈线性关系
   - 这为 β 的选择提供了理论指导

### 3.2 KL 的计算方式

逐 token 计算：
```
KL(π_RL || π_SFT) = Σ_t log(π_RL(token_t | context_t) / π_SFT(token_t | context_t))
```

对整个生成序列的所有 token 求和。β 是超参数，控制约束强度：
- β 太小 → 策略偏离过远，可能奖励欺骗
- β 太大 → 策略几乎不变，学不到东西
- 典型值：0.01 ~ 0.1

---

## 4. PPO 在 RLHF 中的角色

### 4.1 为什么选 PPO

PPO（Proximal Policy Optimization，Schulman et al., 2017）是 RLHF 的首选 RL 算法，原因：

| 特性 | PPO 的优势 |
|------|-----------|
| **信任域约束** | PPO 的 clip 机制天然适合"保持接近预训练模型"的需求 |
| **成熟度** | 被广泛验证、实现完善、调参经验丰富 |
| **可扩展性** | 能处理数十亿参数的大型策略 |
| **稳定性** | 比 TRPO 更简单，比 A2C 更稳定 |

### 4.2 PPO 的不足

尽管 PPO 是 RLHF 的标准选择，但存在明显不足：
- **实现复杂**：需要同时维护策略模型、参考模型、RM、价值模型（4 个模型）
- **训练不稳定**：超参数敏感，RL 训练中常见崩溃
- **样本效率低**：每步需要从当前策略采样（on-policy）

这些不足直接催生了 DPO（2023）和 GRPO（2024）的诞生。

---

## 5. RLHF 的核心问题与批评

### 5.1 奖励欺骗（Reward Hacking）

**定义**：策略学会生成 RM 给高分但人类不喜欢的内容。

**经典案例**：
- 模型学会生成极长但无意义的回答（RM 偏好长文本但无法判断质量）
- 模型学会重复某些 RM 偏好的短语模式
- 模型学会"讨好"RM 而不是真正满足用户需求

**对策**：
- KL 惩罚（主要手段）
- 更大的 RM（更难被欺骗）
- 在线数据收集：持续从策略采样 + 人类标注新偏好数据
- 混合奖励：RM 打分 + 规则奖励 + 长度惩罚

### 5.2 对齐税（Alignment Tax）

**定义**：让模型更安全（拒绝有害请求）时，在合法任务上的表现下降。

- InstructGPT：对齐税较小（PPO 后真实性改善，性能退化轻微）
- Claude 系列：安全性持续提升，但有"过度拒绝"争议
- 本质是"有用性"和"无害性"之间的权衡

### 5.3 人力和成本

- InstructGPT 约 40 名标注者
- Anthropic HH-RLHF 约 161K 条偏好数据
- 大型 RLHF 管道的总成本可达数百万美元
- 标注者的一致性（inter-annotator agreement）通常只有 60-70%

---

## 6. RLHF 的现代演进

### 6.1 从 RLHF 到 RLAIF

**RLAIF（Reinforcement Learning from AI Feedback）**：
- 用 AI 模型（如更大、更强的 LLM）替代人类提供偏好判断
- Anthropic 的宪法 AI（Constitutional AI）是典型代表
- 成本更低、可无限扩展、标准一致
- 但 AI 评估者的偏见会被放大并传递给被训练模型

### 6.2 从 RLHF 到 DPO

DPO（Direct Preference Optimization, 2023）通过数学等价变换，将 RLHF 的三步流程简化为一个二分类交叉熵损失——不再需要显式训练 RM 和使用 PPO。详见 [DPO 专题笔记](dpo-direct-preference-optimization.md)。

### 6.3 从 RLHF 到 GRPO

GRPO（Group Relative Policy Optimization, 2024）保留了 RL 的训练方式，但用群组相对优势替代了传统 PPO 中的 Critic（价值函数）模型，大幅降低了计算开销。详见 [GRPO 专题笔记](grpo-group-relative-policy-optimization.md)。

---

## 7. 关键模型与论文

| 模型/论文 | 年份 | 机构 | RLHF 角色 |
|-----------|------|------|-----------|
| Deep RL from Human Preferences | 2017 | OpenAI | RLHF 诞生 |
| Fine-Tuning LMs from Human Preferences | 2019 | OpenAI | 首次用于语言模型 |
| InstructGPT / ChatGPT | 2022 | OpenAI | RLHF 大规模验证 |
| Anthropic HH-RLHF | 2022 | Anthropic | 公开偏好数据集 |
| Llama 2 | 2023 | Meta | 使用 RLHF 的开源大模型 |
| Constitutional AI (Claude) | 2023 | Anthropic | RLAIF 的开创性应用 |

---

## 8. RLHF 与其他方法的定位

```
LLM 对齐方法全家福：

RLHF (2017-2022)  ← 经典范式，PPO + RM，计算昂贵
  ├── RLAIF (2023)  ← AI 替代人类标注
  ├── DPO  (2023)   ← 数学简化，跳过 RM 和 PPO
  ├── GRPO (2024)   ← 简化 RL，群组优势替代 Critic
  └── ARPO (2025)   ← Agent 场景，多轮工具调用 RL
```

### RLHF 仍然重要的原因

尽管 DPO 和 GRPO 更简单高效，RLHF 仍不可替代：
- **在线探索**：RLHF 可以在训练过程中持续采样和探索新行为
- **复杂奖励**：RM 可以编码复杂的、难以定义规则的偏好
- **理论基础**：RLHF 的数学框架是 DPO/GRPO 等方法的根源
- **工业验证**：ChatGPT、Claude、Gemini 等顶级模型都使用 RLHF

---

## 更新记录

- 2026-06-18：初始创建，从全景笔记中提取并扩展 RLHF 详细内容
