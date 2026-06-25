---
title: "Agent RL 后训练 2026 行业全景：各大厂研究路线与演化关系"
created: 2026-06-25
updated: 2026-06-25
sources:
  - https://www.latent.space/p/state-of-post-training-from-gpt-41
  - https://www.alibabacloud.com/blog/sapo-a-stable-and-performant-reinforcement-learning-method-for-training-large-language-models_602734
  - https://arxiv.org/abs/2511.20347
  - https://deepseek.ai/blog/deepseek-guide-2026
  - https://seed.bytedance.com/blog/seed-2-0-official-launch
  - https://qwen.ai/blog?id=gspo
  - https://arxiv.org/abs/2402.03300
  - https://arxiv.org/abs/2501.12948
has_toc: true
tags: [Agent-RL, OpenAI, Anthropic, DeepSeek, Alibaba, ByteDance, Qwen, PostTraining, Industry-Landscape]
category: ai
---

# Agent RL 后训练 2026 行业全景

## 摘要

2026 年，Agent RL 后训练已从学术探索变为大厂军备竞赛的核心战场。OpenAI 从 GPT-4.1→5.1 完成 RLVR 转型，Anthropic 用 Constitutional AI + 动态 Workflow 重新定义对齐，DeepSeek 押注 V4 + R2 推理双轨发展，阿里 Qwen 推出 SAPO/GSPO 算法创新，字节 Seed 将 VeRL 打造成社区标准。本文梳理各厂路线、与知识库经典概念的关联、以及技术演化关系。

---

## 1. OpenAI — RLVR 时代的引领者

### 1.1 核心路线：RLVR + Token 效率

OpenAI 后训练负责人 Josh McGrath（NeurIPS 2025 访谈）明确：

> "RLHF 和 RLVR 本质上都是策略梯度方法——区别不在数学公式，而在于**输入数据的质量和信号可信度**。"

**关键观点**：
- RLHF（人类偏好）→ 信号噪声大、昂贵
- RLVR（可验证奖励）→ 信号可信、可规模化
- GRPO 被低估：不仅是优化技巧，更是**对可靠奖励信号的系统性转向**（数学答案可验证，而非人类偏好不可验证）

### 1.2 GPT-4.1 → 5.1 演化

| 版本 | 后训练方法 | 关键突破 |
|------|-----------|---------|
| GPT-4.1 | RLHF (PPO) | 基础对齐 |
| GPT-4o | RLHF + 部分 RLVR | 多模态对齐 |
| o1/o3 | 推理 RL (类 GRPO) | 链式思维涌现 |
| GPT-5 | 混合 RLVR | thinking/non-thinking 路由 |
| GPT-5.1 | 纯 RLVR | token 效率革命（提分 + 砍 token） |

### 1.3 与知识库概念关联

| OpenAI 实践 | 知识库概念 |
|------------|-----------|
| GPT-5.1 token 效率 | GRPO 群组归一化（群内比较不依赖绝对分数） |
| thinking/non-thinking 路由 | ARPO 熵驱动自适应（高熵→深度思考，低熵→快速回答） |
| shopping model | Agent RL 中的多步交互 |
| Codex Workflow | Agent CLI 工具生态 |

---

## 2. Anthropic — 宪法 AI + Agent 生态

### 2.1 核心路线：Constitutional AI 持续演进

Anthropic 的对齐路线独树一帜——**不靠人类标注，靠宪法（原则文档）**：

| 版本 | 年份 | 宪法规模 | 关键事件 |
|------|------|---------|---------|
| CAI v1 | 2022 | 75 条准则 | 首次提出 RLAIF |
| Claude 2 | 2023 | ~200 条 | 实用性验证 |
| Claude 3 | 2024 | ~5000 字 | Opus/Sonnet/Haiku 分层 |
| Claude 4 (Opus 4.8) | 2026.05 | 23,000 字 | Dynamic Workflows + 1000 Subagent 上限 |

### 2.2 Claude Code SKILL 生态

Anthropic 在 Agent 执行层面领先：
- **Skills**：10 月 2025 发布，已成为跨工具开放标准（agentskills/agentskills）
- **Subagents**：context: fork 隔离 + worktree 并行
- **Dynamic Workflows**：分类-行动 / 扇出-综合 模式
- **Auto Mode**：Sonnet 4.6 安全分类器驱动的自主模式

### 2.3 与知识库概念关联

| Anthropic 实践 | 知识库概念 |
|---------------|-----------|
| Constitutional AI | RLAIF/RLAIF 章节 |
| Skills 渐进式加载 | Token 效率（与 GPT-5.1 同理念） |
| Subagent 上下文隔离 | 黑盒 Agent 概念 |
| Dynamic Workflows | ARPO 多轮 Agent RL |

---

## 3. DeepSeek — 推理先锋，R2 待发

### 3.1 核心路线：V4 旗舰 + R2 推理双轨

| 模型 | 状态 | 定位 |
|------|------|------|
| DeepSeek-V4 | ✅ 已发布 (2026.04) | 编码旗舰，1M token 上下文 |
| DeepSeek-R2 | ⏳ 推迟 | 推理专用，CEO 不满进度 |
| DeepSeek-V3.2 | ✅ 已发布 | 基座模型升级 |

### 3.2 R1 的革命性遗产

DeepSeek-R1 (2025.01) 用 GRPO + 纯 RL（无 SFT）让模型自发涌现推理能力——这一突破重塑了整个行业的后训练范式。OpenAI 的 o-series、阿里的 Qwen-thinking、字节的 Seed-Thinking 都直接受其影响。

### 3.3 与知识库概念关联

| DeepSeek 贡献 | 知识库概念 |
|--------------|-----------|
| DeepSeek-R1 推理涌现 | GRPO 专题（Aha Moment） |
| DeepSeekMath | GRPO 算法起源 |
| R1 训练流程 | 拒绝采样 + 蒸馏 |
| 开源战略 | VeRL 生态（TinyZero 等复现） |

---

## 4. 阿里巴巴 / Qwen — 算法创新引领

### 4.1 核心路线：SAPO + GSPO + IB-TPO

阿里是 2025-2026 年 **RL 算法创新最活跃**的大厂：

| 算法 | 论文 | 核心创新 |
|------|------|---------|
| **SAPO** | 2511.20347 | 软自适应策略优化，连续信任域替代硬裁剪 |
| **GSPO** | Qwen 官方 | 序列级裁剪（Group Sampling Policy Optimization） |
| **IB-TPO** | 2605.28109 (ICML 2026) | 信息瓶颈驱动树策略优化 |
| **Retrieval-GRPO** | 2511.13885 | 多目标 RL（检索增强） |
| **Qwen-AgentWorld** | 最新 | 语言世界模型训练通用 Agent |

### 4.2 SAPO 核心创新

SAPO 解决了 GRPO/GSPO 硬裁剪的根本问题：

```
GRPO:  token在裁剪带内 → 全梯度 | token在裁剪带外 → 零梯度
GSPO:  序列在裁剪带内 → 全梯度 | 序列在裁剪带外 → 零梯度
SAPO:  温度控制平滑衰减 → 保留有用梯度 + 抑制噪声更新
```

**关键设计**：
- 连续信任域（无硬截止）
- 序列级一致性（类 GSPO 但不丢弃整序列）
- Token 级自适应（选择性抑制问题 token）
- 非对称温度（负面优势 token 衰减更快→更稳定）

### 4.3 与知识库概念关联

| 阿里/Qwen 实践 | 知识库概念 |
|---------------|-----------|
| SAPO | GRPO 专题（软裁剪 vs 硬裁剪） |
| GSPO | GRPO 专题（序列级 vs Token 级） |
| IB-TPO | Tree-GRPO 专题（信息瓶颈） |
| Qwen3-VL RL | Agent RL 框架（多模态 RL） |

---

## 5. 字节跳动 / Seed — VeRL 帝国 + Agentic Model

### 5.1 核心路线：VeRL 社区 + Seed 模型矩阵

字节的策略是"**框架开源 + 模型闭源**"：

| 层次 | 产品 | 说明 |
|------|------|------|
| 框架 | **VeRL** (22K⭐) | 社区标准 RL 训练框架 |
| 框架扩展 | uni-agent, VeRL-Omni | Agent/多模态 RL |
| 模型 | **Seed 1.8** | 通用 Agentic Model |
| 模型 | **Seed 2.0 (Doubao)** | 旗舰对话模型 |
| 模型 | **Seed-Thinking-v1.5** | 推理模型 (AIME 86.7) |

### 5.2 VeRL 的社区统治力

VeRL 已成为 RL 后训练的"PyTorch"：
- 22K GitHub Stars
- 20+ 社区项目基于 VeRL（TinyZero, DAPO, Easy-R1, Search-R1...）
- NVIDIA GTC26 两场官方演讲
- 被 Qwen、Kimi、StepFun 等多家采用

### 5.3 Seed 1.8 Agentic Model

2026 年初发布的 Seed 1.8 定位为"通用 Agent 模型"——直接在训练阶段集成工具调用能力，而非后期通过 prompt engineering。

### 5.4 与知识库概念关联

| 字节/Seed 实践 | 知识库概念 |
|---------------|-----------|
| VeRL (HybridFlow) | Agent RL 框架生态 |
| Seed-Thinking | GRPO + DAPO/VAPO |
| Seed 1.8 Agentic | ARPO（Agent 原生 RL） |
| Doubao-1.5-pro | RLVR + 可验证奖励 |

---

## 6. 其他重要玩家

### 6.1 Kimi / Moonshot AI

| 方向 | 详情 |
|------|------|
| **Kimi Code CLI** | 终端 AI 编程 Agent，基于 TypeScript |
| **Kimi Claw** | 原生 OpenClaw，5000+ 社区 Skills |
| **RL 后训练** | 使用 VeRL 进行推理模型训练 |

### 6.2 Google / DeepMind

| 方向 | 详情 |
|------|------|
| **Gemini 3 Deep Think** | ARC-AGI-2 84.6%，Humanity's Last Exam 突破 |
| **Aletheia** | 从数学竞赛到自主科研发现的 Agent |
| **Antigravity 2.0** | Agent-first 平台，CLI+SDK+托管执行 |

### 6.3 Meta

| 方向 | 详情 |
|------|------|
| **Llama 4** | 开源旗舰，使用 DPO + 迭代 RL |
| **Self-Rewarding LM** | 自我奖励（ICML 2024） |

### 6.4 Mistral

| 方向 | 详情 |
|------|------|
| **Magistral** | 推理模型，使用 REINFORCE++ 训练 |
| **Mistral Large 3** | RL 后训练提升编码能力 |

---

## 7. 技术演化全景图

```
2022        2023         2024          2025             2026
 │           │            │             │                │
RLHF        DPO         GRPO          RLVR 元年       Agent RL 军备竞赛
(Instruct)  (Stanford)  (DeepSeek)    (DeepSeek-R1)   
                                     ├─ OpenAI: o1→GPT-5.1
                                     │  (RLVR + Token效率)
                                     │
                                     ├─ Anthropic: CAI 23K字
                                     │  (RLAIF + Workflows)
                                     │
                                     ├─ DeepSeek: V4+R2
                                     │  (推理+编码双轨)
                                     │
                                     ├─ 阿里: SAPO+GSPO+IB-TPO
                                     │  (算法创新集群)
                                     │
                                     ├─ 字节: VeRL社区+Seed
                                     │  (框架开源+模型闭源)
                                     │
                                     └─ Tree-GRPO 流派
                                        (CATPO/InfoTree/ATPO)
```

### 与知识库经典概念的演化链

```
RLHF（2017-2022）                   知识库：RLHF 专题
  ↓
DPO（2023）                         知识库：DPO 专题
  ↓                                 数学等价变换，策略即RM
GRPO（2024）                        知识库：GRPO 专题
  ↓                                 群组优势，省掉 Critic
RLVR（2025）                        知识库：概念全景 5.2
  ├─ DeepSeek-R1 (推理涌现)         "可验证奖励" → 推理模型
  └─ OpenAI GPT-5.1 (token效率)     "信号可信度" → 通用模型
  ↓
Agent RL（2025-2026）               知识库：ARPO 专题
  ├─ ARPO (熵驱动 + 优势归因)
  ├─ SAPO (软裁剪 + 非对称温度)     新算法
  └─ Seed 1.8 (Agentic Model)
  ↓
Tree-GRPO（2026）                   知识库：Tree-GRPO 专题
  ├─ CATPO (树质量筛选)
  ├─ IB-TPO (信息瓶颈, ICML 2026)
  └─ InfoTree (子模块优化)
  ↓
未来方向
  ├─ Universal Verifier (OpenAI): 通用奖励信号
  ├─ RULER (OpenPipe): LLM-as-Judge
  └─ Self-Play + Multi-Agent RL (Meta/OpenManus)
```

---

## 8. 各厂战略定位总结

| 厂商 | 核心策略 | 知识库关联 |
|------|---------|-----------|
| **OpenAI** | RLVR + Token 效率革命，不公开算法细节 | GRPO → ARPO → RLVR |
| **Anthropic** | Constitutional AI + Agent 生态 (Skills/Subagents) | RLAIF + Agent CLI |
| **DeepSeek** | 开源推理引领 + V4 编码旗舰 | GRPO 开创者 |
| **阿里/Qwen** | 算法创新最活跃 (SAPO/GSPO/IB-TPO) | GRPO 变体 + Tree-GRPO |
| **字节/Seed** | VeRL 社区标准 + Seed 模型矩阵 | Agent RL 框架生态 |
| **Kimi** | CLI Agent 差异化 + VeRL 训练 | Agent CLI 工具 |
| **Google** | Gemini 推理 + Agent-first 平台 | 多模态 RL |
| **Meta** | 开源 + Self-Reward 路线 | DPO + 自我改进 |

---

## 更新记录

- 2026-06-25：初始创建，覆盖 OpenAI/Anthropic/DeepSeek/阿里/字节/Kimi/Google/Meta 2026 年 Agent RL 后训练全景
