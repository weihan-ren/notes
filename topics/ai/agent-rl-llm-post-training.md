---
title: "Agent RL 与 LLM 后训练基础概念全景"
created: 2026-06-14
updated: 2026-06-19
sources:
  - https://arxiv.org/abs/1706.03741
  - https://arxiv.org/abs/2203.02155
  - https://arxiv.org/abs/2305.18290
  - https://arxiv.org/abs/2402.03300
  - https://arxiv.org/abs/2501.12948
  - https://arxiv.org/abs/2212.08073
  - https://arxiv.org/abs/2507.19849
sources:
  - https://arxiv.org/abs/1706.03741
  - https://arxiv.org/abs/2203.02155
  - https://arxiv.org/abs/2305.18290
  - https://arxiv.org/abs/2402.03300
  - https://arxiv.org/abs/2501.12948
  - https://arxiv.org/abs/2212.08073
tags: [RLHF, DPO, GRPO, DeepSeek, LLM, AgentRL, PostTraining, Alignment]
category: ai
---

# Agent RL 与 LLM 后训练基础概念全景

## 摘要

本文系统梳理了大型语言模型（LLM）后训练阶段的核心技术栈，涵盖从经典 RLHF 到 DeepSeek-R1 的 GRPO 创新，以及 Agent 场景下的强化学习范式。重点解释各概念之间的关系、数学直觉和工程权衡。

---

## 1. LLM 训练管线全景

LLM 的完整训练分为四大阶段，每个阶段解决不同的问题：

```
预训练 (Pre-training) → 监督微调 (SFT) → 对齐 (RLHF/DPO/GRPO) → 蒸馏 (Distillation)
```

### 1.1 预训练（Pre-training）

- **做什么**：在海量互联网文本（数万亿 token）上做下一个 token 预测（自回归语言建模）
- **目标**：让模型学会语言的统计规律、世界知识、推理能力的"原材料"
- **产物**：Base Model（基座模型），能续写文本但不擅长遵循指令
- **典型代表**：GPT-3、Llama、DeepSeek-V3-Base、Qwen-Base
- **关键特点**：
  - 计算量最大（占总训练成本的 95%+）
  - 只做"补全"，不问"你想要什么"
  - 推理能力已被"植入"但未被"激活"

### 1.2 监督微调（SFT, Supervised Fine-Tuning）

- **做什么**：在高质量 `(提示, 理想回答)` 数据对上做有监督训练
- **数据来源**：人工撰写的示范、精选的高质量对话
- **目标**：让模型学会对话格式、遵循指令、输出风格良好
- **关键发现（InstructGPT）**：1.3B 的 SFT 模型在人类偏好上优于 175B 的纯基座模型
- **局限性**：
  - 只能学到"回答长什么样"，学不到"什么回答更好"
  - 对于难以言说但容易判断的质量（如"有帮助性"），SFT 不够

### 1.3 对齐阶段（Alignment）

这是后训练的核心，包含 RLHF、DPO、GRPO 等多种方法（详见后续章节）：

| 方法 | 年份 | 核心思路 | 是否需要 RM | 是否需要 PPO |
|------|------|----------|-------------|-------------|
| **RLHF** | 2017-2022 | 训练 Reward Model + PPO 优化 | ✅ 需要 | ✅ 需要 |
| **DPO** | 2023 | 将 Reward 函数重新参数化为策略比值 | ❌ 不需要 | ❌ 不需要 |
| **GRPO** | 2024 | 群组相对优势替代 Critic 模型 | ✅ 需要（规则/模型） | 部分简化 |
| **ARPO** | 2025 | 熵驱动自适应 Rollout（Agent 场景）；GRPO + 经验回放（GUI 场景） | ✅ 需要（规则/模型） | ❌ 不需要 |

**共同目标**：让模型输出符合人类偏好（有帮助、无害、诚实）。

### 1.4 蒸馏（Distillation）

- **做什么**：用大模型（Teacher）生成的优质数据训练小模型（Student）
- **在 LLM 中的应用**：
  - **黑盒蒸馏**：用 GPT-4 生成数据，微调开源小模型（如 Alpaca、Vicuna）
  - **推理蒸馏**：DeepSeek-R1 的 `<think>` 推理轨迹→ 训练 R1-Distill 系列
  - **自我蒸馏**：SPIN、Self-Rewarding LM（模型自我改进）
- **DeepSeek-R1 的关键发现**：大模型通过 RL 发现的推理模式，可以通过监督学习蒸馏给小模型——小模型不需要再做 RL
- **意义**：降低推理成本、提升推理速度、在消费级硬件上运行强大模型

---


> **详见**：[RLHF 专题笔记](rlhf-reinforcement-learning-from-human-feedback.md) — 三步流程、KL 惩罚机制、奖励欺骗问题及对策
> **详见**：[PPO 专题笔记](ppo-proximal-policy-optimization.md) — 裁剪机制、GAE 优势估计、Actor-Critic 架构

## 2. RLHF（基于人类反馈的强化学习）

> **详见**：[RLHF 专题笔记](rlhf-reinforcement-learning-from-human-feedback.md) — 三步流程、KL 惩罚机制、奖励欺骗问题及对策
> **详见**：[PPO 专题笔记](ppo-proximal-policy-optimization.md) — 裁剪机制、GAE 优势估计、Actor-Critic 架构

**一句话**：RLHF 是 LLM 对齐的经典范式——训练 Reward Model 替代人类打分 + PPO 强化学习优化 + KL 惩罚防止跑偏。

**三步流程**：SFT（学格式）→ 训练 RM（学偏好）→ PPO 优化（对齐偏好）。经典问题是奖励欺骗和 KL 惩罚权衡。PPO 需要 4 个模型同时驻留 GPU（策略+参考+RM+价值），这是 DPO 和 GRPO 诞生的主要动机。

**代表模型**：InstructGPT/ChatGPT (OpenAI, 2022)、Claude (Anthropic)、Llama 2 (Meta)

[arXiv:2203.02155](https://arxiv.org/abs/2203.02155) · [arXiv:1706.03741](https://arxiv.org/abs/1706.03741)

---
## 3. DPO（直接偏好优化）

> **详见**：[DPO 专题笔记](dpo-direct-preference-optimization.md) — 数学等价变换、隐式奖励、DPO 变体家族（IPO/KTO/ORPO/SimPO）
## 3. DPO（直接偏好优化）

> **详见**：[DPO 专题笔记](dpo-direct-preference-optimization.md) — 数学等价变换、隐式奖励、DPO 变体家族（IPO/KTO/ORPO/SimPO）

**一句话**：通过数学等价变换，将 RLHF 的三步流程简化为一个二分类交叉熵损失——"你的语言模型本身就是隐含的奖励模型"。

**核心公式**：L_DPO = -E[log σ(β·log(π_θ(y_w)/π_ref(y_w)) - β·log(π_θ(y_l)/π_ref(y_l)))]

**优势**：极简实现（~20行）、无需 PPO、无需 RM、训练稳定。**劣势**：off-policy（无法在线探索）、容易过拟合、不适合 Agent 多轮场景。

**变体家族**：IPO（抗过拟合）、KTO（无需成对数据）、ORPO（无需参考模型）、SimPO（长度归一化）

[arXiv:2305.18290](https://arxiv.org/abs/2305.18290)

---
## 4. GRPO（群组相对策略优化）

> **详见**：[GRPO 专题笔记](grpo-group-relative-policy-optimization.md) — 群组相对优势、DeepSeek-R1 推理涌现、think token 机制
## 4. GRPO（群组相对策略优化）

> **详见**：[GRPO 专题笔记](grpo-group-relative-policy-optimization.md) — 群组相对优势、DeepSeek-R1 推理涌现、think token 机制

**一句话**：用同一提示下 G 个输出的群组归一化奖励替代传统 PPO 的 Critic 模型，将 RL 训练的内存开销减半。

**核心公式**：优势_i = (r_i - mean(r_1..r_G)) / std(r_1..r_G)

**DeepSeek-R1 的革命性应用**：将 GRPO 直接作用于基座模型（完全跳过 SFT），仅靠规则可验证奖励（数学答案对/错），模型自发涌现了自我验证、反思、动态策略调整等推理行为——"Aha Moment"。TinyZero 用 $30 和 3B 模型复现了这种涌现。

**与 PPO 关键区别**：不需 Critic（省一半内存）、KL 惩罚直接在损失中、天然适配可验证奖励。代价是每次需生成 G 个回答。

[arXiv:2402.03300](https://arxiv.org/abs/2402.03300) (DeepSeekMath) · [arXiv:2501.12948](https://arxiv.org/abs/2501.12948) (DeepSeek-R1)

---
---

## 5. Agent 专用强化学习

### 5.1 结果奖励 vs 过程奖励模型（PRM）

**结果奖励（Outcome-based Reward）**：
- 仅根据最终答案给出奖励 ——"对"或"错"
- 简单、可自动扩展（数学题答案、代码执行结果）
- 问题：奖励稀疏——不知道是哪一步出了错

**过程奖励模型（Process Reward Model, PRM）**：
- 对推理的每一步给出奖励信号
- 来自 OpenAI 论文 *"Let's Verify Step by Step"* (Lightman et al., 2023)
- 发布了 PRM800K（80 万条步骤级人类反馈）
- 优势：更高效的学分分配，更好的样本效率
- 代价：需要训练额外的 PRM 模型

**DeepSeek 的中间路线**：不用 PRM，只用结果奖励 + 足够的 RL 探索，让模型自发学会自我验证和反思。

### 5.2 可验证奖励（Verifiable Rewards）

**定义**：可以通过自动化手段确真伪的奖励信号，无需人类判断。

- **数学问题**：已知标准答案（MATH, GSM8K）
- **代码执行**：运行代码看是否通过测试用例
- **逻辑谜题**：答案确定唯一
- **STEM 问题**：可对答案

**为什么"更容易"**：
1. 不需要训练 RM——信号客观、无噪声
2. 可无限扩展——自动生成数百万训练样本
3. 不会被欺骗——答案要么对其么错
4. 但奖励稀疏——这正是 GRPO 擅长处理的场景

**反之**：创意写作、幽默、安全对齐等任务——必须依赖 RM 或人类反馈，这是"更难"的 RL 问题。

### 5.3 工具调用作为动作空间

在标准 RLHF 中，动作空间是"下一个 token"。Agent RL 扩展了动作空间：

| 传统 RLHF 动作 | Agent RL 动作 |
|---------------|---------------|
| 生成下一个 token | 生成 API 调用（如 `get_weather("London")`）|
| 生成下一个 token | 发起网络搜索 |
| 生成下一个 token | 执行 Python 代码 |
| 生成下一个 token | 查询数据库 |
| 生成下一个 token | 读写文件 |

**RL 含义**：
- 动作空间变得异构——不同动作有不同奖励结构
- 环境变为部分可观测——不执行工具就不知道结果
- 工具执行结果提供可验证的监督信号
- 多轮交互——模型需要交替进行推理、工具调用、结果解读

这模糊了 LLM 对齐与 **Agentic AI** 的界限——模型学的不只是"说什么"，更是"何时用什么工具"。

### 5.4 多步推理 RL

模型生成完整的推理链后再得出最终答案：
```
"步骤 1：理解问题……步骤 2：计算……步骤 3：验证……因此，答案 = 42"
```

- **结果 RL**：只有最终答案正确才给奖励，模型必须自学"信用分配"
- **PRM RL**：每一步都有奖励信号
- **DeepSeek-R1 的发现**：足够规模的 RL + 长推理轨迹，足以让推理模式自然涌现，不需要显式教

涌现出的推理模式：
- 自我反思："等等，我检查一下……"
- 验证："让我验证这个结果……"
- 回溯："不对，上一步是错的……"
- 动态策略调整：切换解题方法

### 5.5 推理 token 机制总结

- `<think>...</think>` 标记推理段
- RL 只关心 `<think>` 之外的最终答案是否正确
- 但模型被激励在 `<think>` 中有效利用"思考预算"
- 推理链可以提取、蒸馏给更小的模型
- 形成自然的"计算换准确率"权衡

---

### 5.6 Trajectory（轨迹）详解

**轨迹（Trajectory）** 是 RL 中最基础的概念——Agent 与环境完整交互的一条"路径记录"。

#### 形式定义

```
轨迹 = S₀ → A₀ → R₁ → S₁ → A₁ → R₂ → ... → S_T
       状态   动作   奖励   状态   动作   奖励     终态
```

#### Agent RL 中的具体含义

| 场景 | 轨迹是什么 | 长度 |
|------|-----------|------|
| **RLHF (PPO)** | 一条完整生成的回答（多 token 序列） | 几十到几百 token |
| **GRPO（推理 RL）** | 一条完整推理链（`<think>` + `<answer>`） | 几百到几千 token |
| **Agent RL（工具调用）** | 一轮多步任务（推理 → 工具调用 → 观察 → 推理 → ...） | 多轮对话 |
| **OmniRL（In-Context RL）** | 在上下文中展示的多步决策序列 | 可达 512K 步 |

#### GRPO 中轨迹的关键作用

GRPO 对同一 prompt 生成 G 条轨迹（如 G=16），然后在群组内归一化奖励：

```
同一 Prompt: "计算 3x + 5 = 14, x=?"
  ├── 轨迹 A: Token序列 → 答案"x=3"  → 奖励=1 (正确)
  ├── 轨迹 B: Token序列 → 答案"x=9"  → 奖励=0 (错误)
  ├── 轨迹 C: Token序列 → 答案"x=3"  → 奖励=1 (正确)
  └── ...     (共16条)

优势_i = (奖励_i - mean(奖励_1..G)) / std(奖励_1..G)
```

**GRPO 只关相对排序**——同组内哪条轨迹更好，不关心绝对分数。
这正是 RULER（LLM-as-Judge）能工作的原因：Judge 不需要打绝对分，只需要对同组轨迹进行相对排序。

#### ARPO 对轨迹的扩展

传统 GRPO 在**轨迹级别**采样（每条轨迹从 prompt 到 answer 完整生成），但 ARPO 观察到：

```
传统 GRPO:  全部轨迹级采样
  Prompt → [整条轨迹A] [整条轨迹B] ...  → 群组归一化 → 更新

ARPO:  混合采样（轨迹级 + 步级）
  Prompt → 推理 → [工具调用] ← 熵增！
                         ├── 后续路径X (步级采样)
                         ├── 后续路径Y (步级采样)
                         └── 后续路径Z (步级采样)
  → 高熵步骤上增加探索，低熵步骤保持高效
```

#### 三个关联概念

| 概念 | 说明 | Agent RL 中的含义 |
|------|------|------------------|
| **Trajectory（轨迹）** | 完整交互序列 | 一条完整回答 / 一轮完整任务 |
| **Step（步）** | 轨迹中的单步 | 生成一个 token / 一次工具调用 |
| **Group（群组）** | 同一 prompt 的多条轨迹 | GRPO 中 G 条轨迹构成一个 group |

> 详见：[Agent RL 框架生态](agent-rl-frameworks-ecosystem.md) 中 VeRL/OpenRLHF 的具体轨迹处理

## 6. 其他后训练技术

### 6.1 宪法 AI（Constitutional AI）

**提出者**：Anthropic（2022 年，arXiv:2212.08073）

**核心思想**：用一份书面"宪法"（原则清单）替代海量人类标注，引导模型的无害化训练。

**两阶段流程**：

**阶段一 — 监督学习（SL-CAI）**：
1. 采样有害提示，模型生成回答
2. 让模型根据宪法 **自我批评**："这条回答在哪些方面违反了原则 X？"
3. 让模型 **自我修改** 回答以符合宪法
4. 用修改后的回答微调模型

**阶段二 — 强化学习（RL-CAI / RLAIF）**：
1. 采样提示，生成两个回答
2. 让 AI 模型（而非人类）根据宪法评判哪个更好 → 即 RLAIF
3. 用 AI 偏好训练偏好模型
4. 用 PPO 训练最终模型

**宪法内容演变**：2023 版 75 条准则；2026 版扩展至 23,000 字（来源：联合国人权宣言、公司价值观、安全原则等）

### 6.2 RLAIF（基于 AI 反馈的强化学习）

- **做法**：用 AI 模型替代人类提供反馈
- **优势**：可大规模扩展，成本低，标准一致
- **风险**：AI 评估者的偏见会被放大并传递给被训练模型
- **应用**：
  - Anthropic 宪法 AI 的第二阶段
  - Meta Self-Rewarding Language Models
  - Llama 2 部分使用 AI 反馈

### 6.3 迭代自我改进 / 自我博弈

**SPIN（Self-Play Fine-Tuning）**（Chen et al., 2024, ICML）：
- LLM 与自己的历史版本"对弈"
- 生成回答 → 学习区分自己的生成 vs 人类数据 → 模型提升 → 循环
- 理论保证：全局最优在 LLM 分布与目标数据分布一致时达到
- **结果**：不需要额外人类数据或 GPT-4 偏好数据，SPIN 超越 DPO

**Self-Rewarding Language Models**（Meta, 2024, ICML）：
- 模型通过 LLM-as-a-Judge 提示给自己打分
- 指令遵循能力和打分能力 **同步提升**
- 三轮迭代后，Llama 2 70B 超越 Claude 2、Gemini Pro、GPT-4 0613

### 6.4 指令微调 vs RLHF

| 维度 | 指令微调（SFT）| RLHF |
|------|---------------|------|
| **训练信号** | 有监督：(提示, 理想回答) | 奖励信号：偏好对比 |
| **数据需求** | 人工撰写的示范 | 人类偏好对比（更易收集）|
| **目标** | 模仿人类回答 | 最大化人类偏好 |
| **学到什么** | 好回答"长什么样" | 人类"偏好什么" |
| **优点** | 简单、稳定 | 能优化难以言说的质量 |
| **缺点** | 只能模仿示范数据 | 管道复杂、可能奖励欺骗 |

**关键认知**：SFT 和 RLHF 是互补的，而非替代。SFT 教格式，RLHF 教偏好。大多数对齐模型同时使用两者。

### 6.5 对齐税（Alignment Tax）

**定义**：在对齐模型时，有用性（helpfulness）与安全性（harmlessness）之间的权衡代价。

- 让模型更安全（拒绝有害请求）→ 可能降低在合法任务上的表现
- **过度拒绝**：模型拒绝看似有害实际无害的请求
- 这是"能力-安全权衡"的根本问题

**各方处理方式**：
- InstructGPT：对齐税较小，RLHF 改善真实性的同时性能退化轻微
- Llama 2：使用 **独立的 helpfulness 和 safety RM**，试图平衡两者
- Claude：各版本之间安全性持续提升，但仍有"过度拒绝"报告

**开放问题**：是否能做出既有用又绝对安全的模型？多数证据表明存在根本性的权衡——对齐税是一个光谱，而非已解决的问题。

---

## 7. 关键术语表

| 术语 | 英文 | 通俗解释 |
|------|------|----------|
| **策略** | Policy (π) | "给定上下文，下一个 token 选什么"的决策规则。LLM 本身就是一个策略 |
| **奖励模型** | Reward Model (RM) | 对任意 (提示, 回答) 给一个偏好分数，是"人类偏好的数学代理" |
| **偏好数据** | Preference Data | "回答 A 比回答 B 更好"的标注数据，通常是从多个模型中挑选 |
| **On-Policy** | On-Policy | 策略用自己当前生成的数据来学习——每次更新后旧数据作废 |
| **Off-Policy** | Off-Policy | 策略用别人（或旧版本）生成的数据来学习——更省数据但可能不稳定 |
| **Actor-Critic** | Actor-Critic | 双组件架构：Actor 做决策（策略），Critic 评估好坏（价值函数） |
| **PPO** | Proximal Policy Optimization | 约束每次更新幅度的策略梯度算法，通过裁剪阻止策略突变 |
| **KL 散度** | KL Divergence | 衡量两个概率分布"有多不像"，RLHF 中用于约束策略不偏离太远 |
| **优势** | Advantage A(s,a) | "这个动作比平均好/差多少"——比直接使用奖励方差更小 |
| **推理 token** | Reasoning Tokens | 模型用于内部思考的 token（如 `<think>` 内的内容），不直接贡献最终答案评分 |
| **Bradley-Terry 模型** | Bradley-Terry Model | 将成对比较转化为概率的统计模型：`P(A≻B) = σ(r_A - r_B)` |
| **轨迹** | Trajectory | Agent 与环境完整交互的路径记录 = S₀→A₀→R₁→...→S_T。单轮 RL 中是完整生成序列，多轮 Agent RL 中是多步工具调用链。GRPO 在群组内比较轨迹的优劣 |
| **步** | Step | 轨迹中的单次动作。Token 级 RL 中是生成一个 token，Agent RL 中是一次工具调用。ARPO 在工具调用后的高熵步上增加探索 |
| **群组** | Group | 同一 prompt 下生成的多条轨迹集合（GRPO 中 G=16 或更大）。群组归一化是 GRPO 省掉 Critic 的关键——只需组内相对优劣 |

| **信任域** | Trust Region | 策略每次更新允许的范围——防止一步走太远毁了整个策略 |
| **拒绝采样** | Rejection Sampling | 生成多个候选，只保留"正确的"来训练，是 DeepSeek-R1 的关键步骤 |

---

## 8. 概念关系图

```
                      LLM 后训练全景
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
    对齐方法            指令微调           知识蒸馏
        │               (SFT)          (大模型→小模型)
        │
  ┌─────┴─────┬──────────┬──────────┐
  │           │          │          │
 RLHF        DPO       GRPO     其他方法
(2017-22)  (2023)    (2024)    │
  │           │          │      ├── 宪法 AI
  ├─ 训练RM   ├─ 隐式RM   ├─ 群组相对 │    (Anthropic)
  ├─ PPO优化  ├─ 分类loss  │   优势   ├── RLAIF
  └─ KL惩罚   └─ off-pol  ├─ 无Critic ├── SPIN
                icy       ├─ R1推    └── Self-Reward
                          │   理涌现       ing LM
                          └─ think
                              token

关键联系：
• RLHF教会我们"用RM+PPO做对齐"，但管线和计算开销大
• DPO用数学技巧跳过RM和PPO，但受限于静态数据
• GRPO回到RL路线，但用群组技巧省掉Critic
  → 使纯RL推理涌现成为可能（DeepSeek-R1的革命性突破）
• DPO变体（ORPO、SimPO等）继续向更简单更高效的方向演化
• Agent RL将动作空间从"生成token"扩展到"使用工具"
  → 连接了LLM对齐和Agentic AI两个领域
• ARPO (2025) 将GRPO的群组相对优势扩展到多轮Agent场景，用熵驱动自适应Rollout解决工具调用后的探索-利用平衡
```

---

## 更新记录

- 2026-06-14：初始创建，覆盖 RLHF、DPO、GRPO、Agent RL、后训练技术全景

- 2026-06-19：新增 5.6 Trajectory 详解 + 术语表补充轨迹/步/群组概念
