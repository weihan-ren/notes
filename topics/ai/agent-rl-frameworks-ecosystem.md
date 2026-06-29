---
title: "Agent RL 训练框架与工具生态全景（2026）"
has_toc: true
created: 2026-06-18
updated: 2026-06-29
sources:
  - https://github.com/verl-project/verl
  - https://arxiv.org/abs/2409.19256
  - https://github.com/OpenRLHF/OpenRLHF
  - https://github.com/huggingface/trl
  - https://github.com/OpenPipe/ART
  - https://blog.dailydoseofds.com/p/how-top-ai-labs-are-building-rl-agents
  - https://github.com/hiyouga/EasyR1
tags: [Agent-RL, VeRL, OpenRLHF, TRL, GRPO, PPO, RLHF, PostTraining, ART, NeMo, AReaL, rLLM, ASystem]
category: ai
---

# Agent RL 训练框架与工具生态全景（2026）

## 摘要

2025-2026 年 LLM 后训练 RL 框架经历了爆发式增长。从 DeepSeek-R1 用 GRPO+RLVR 让推理能力涌现开始，RL 训练框架从"只有大厂用得起"变成开源社区的标配。本文深度剖析 VeRL、OpenRLHF、TRL、ART(RULER)、NeMo-Aligner、DeepSpeed-Chat、AReaL（蚂蚁异步 RL）、rLLM（UC Berkeley Agent RL）、ASystem（蚂蚁 Agent 平台）等框架的架构设计、核心差异和生态位。

---

## 1. 核心概念区分

| 概念 | 是什么 | 不是 |
|------|--------|------|
| **RLHF** | 用人类偏好训练 RM + PPO 的对齐范式 | 不是具体框架 |
| **GRPO** | 用群组相对优势替代 Critic 的 RL 算法 | 不是训练框架 |
| **VeRL** | 字节跳动的大规模 LLM RL 框架 | 不是算法 |
| **OpenRLHF** | 基于 Ray 的易用 RLHF 框架 | 不是算法 |
| **TRL** | HuggingFace 的 Transformers RL 库 | 不是算法 |
| **ART+RULER** | Agent 场景 RL 框架 + LLM-as-Judge 通用奖励 | RULER 不是框架 |
| **OmniRL** | In-Context RL 模型（推理时学习，无需参数更新） | 不是训练框架 |
| **AReaL** | 蚂蚁/清华完全异步 RL 训练系统，5.3K⭐ | 不是算法 |
| **rLLM** | UC Berkeley Agent RL 框架，不改 Agent 代码做 RL | 不是算法 |

---

## 2. VeRL — 工业级大规模 RL 训练

### 2.1 基本信息

| 项目 | 详情 |
|------|------|
| **GitHub** | [verl-project/verl](https://github.com/verl-project/verl) |
| **Stars** | 22K+ |
| **论文** | HybridFlow（EuroSys 2025, arXiv:2409.19256） |
| **许可证** | Apache 2.0 |
| **发起方** | ByteDance Seed + 火山引擎 |

### 2.2 HybridFlow 架构

VeRL 的核心创新是 **HybridFlow 编程模型**——将 RL 数据流中的计算和数据依赖解耦：

```
┌──────────────────────────────────────┐
│          VeRL (HybridFlow)           │
│                                      │
│  训练后端        推理后端             │
│  ┌─────────┐    ┌─────────┐         │
│  │ FSDP    │    │ vLLM    │         │
│  │ FSDP2   │    │ SGLang  │         │
│  │ Megatron│    │ HF      │         │
│  └────┬────┘    └────┬────┘         │
│       │              │               │
│  ┌────┴──────────────┴────┐         │
│  │   Hybrid Controller     │         │
│  │   (Ray 分布式调度)       │         │
│  └─────────────────────────┘         │
│                                      │
│  3D-HybridEngine: 训练/推理切换       │
│  零内存冗余, 通信开销大幅减少          │
└──────────────────────────────────────┘
```

**关键优化**：
- **3D-HybridEngine**：Actor 在训练和生成模式间切换时共享权重，消除模型拷贝
- **灵活设备映射**：不同模型放置在不同 GPU 组
- **SOTA 吞吐**：比 OpenRLHF 快 ~1.4x

### 2.3 支持的算法

| 算法 | 用途 |
|------|------|
| PPO | 经典 RLHF |
| GRPO | 群组相对优势（DeepSeek-R1） |
| GSPO | Group Sampling Policy Optimization |
| ReMax | 简化 PPO |
| REINFORCE++ | 无 Critic 的高效 RL |
| RLOO | Leave-One-Out 优势估计 |
| PRIME | 过程奖励隐式估计 |
| DAPO | 动态采样 + 过滤（AIME 50） |
| DrGRPO | 简化 GRPO，移除 /std 归一化 |
| VAPO | Value-based Augmented PPO（AIME 60.4） |

### 2.4 里程碑

| 时间 | 事件 |
|------|------|
| 2025.01 | Doubao-1.5-pro (AIME 70.0) |
| 2025.04 | Seed-Thinking-v1.5 (AIME 86.7) |
| 2025.06 | 支持 DeepSeek-671B MoE |
| 2026.03 | NVIDIA GTC26 两场演讲 |
| 2026.05 | uni-agent（Agent 训练）、VeRL-Omni（多模态 RL） |

### 2.5 基于 VeRL 的生态项目

| 项目 | 用途 |
|------|------|
| **TinyZero** (7K⭐) | DeepSeek R1-Zero 复现（$30, 3B） |
| **Easy-R1** (5K⭐) | 多模态 RL 训练 |
| **DAPO** | SOTA RL 算法 (AIME 50, 32B) |
| **OpenManus-RL** | 多 Agent 环境 RL |
| **Search-R1** | RL + 搜索工具调用 |
| **RAGEN** | 通用推理 Agent 训练 |
| **GUI-R1** | GUI Agent R1 风格训练 |
| **RL-Factory** | Agentic 学习即插即用 |

---

## 3. OpenRLHF — 最易上手的分布式 RLHF

### 3.1 基本信息

| 项目 | 详情 |
|------|------|
| **GitHub** | [OpenRLHF/OpenRLHF](https://github.com/OpenRLHF/OpenRLHF) |
| **Stars** | 9.7K+ |
| **许可证** | Apache 2.0 |
| **发起方** | 社区（hijkzzz 等） |

### 3.2 架构：Ray + vLLM + DeepSpeed

OpenRLHF 是**首个**基于 Ray + vLLM 分布式架构的 RLHF 框架：

```
┌──────────────────────────────────────────┐
│           OpenRLHF (Ray 调度)            │
│                                          │
│  ┌────────┐ ┌────────┐ ┌────────┐       │
│  │ Actor  │ │ Critic │ │  RM    │       │
│  │(DeepSpd│ │(DeepSpd│ │(DeepSpd│       │
│  └───┬────┘ └───┬────┘ └───┬────┘       │
│      │          │          │             │
│  ┌───┴──────────┴──────────┴───┐        │
│  │      vLLM 推理引擎           │        │
│  │   (AutoTP + PP)             │        │
│  └─────────────────────────────┘        │
│                                          │
│  Hybrid Engine: 模型 + vLLM 共享 GPU    │
│  7B 模型单机可训练                        │
└──────────────────────────────────────────┘
```

### 3.3 Agent-Based 执行范式（核心创新）

OpenRLHF 是**首个**实现统一 Agent 执行范式的 RLHF 框架：

```
AgentExecutorBase (Token-in-Token-out)
    │
    ├── SingleTurnExecutor  (标准 RLHF / 自定义 Reward)
    │
    └── MultiTurnExecutor   (多步推理 / 环境交互)
```

**关键设计**：RL 算法（PPO/REINFORCE++/GRPO）与 Agent 执行器**解耦**，任意算法可配合任意执行模式。

### 3.4 支持的算法

| 算法 | 标志 | 特点 |
|------|------|------|
| **PPO** | (默认) | 完整 Critic，最稳定 |
| **REINFORCE++** | `reinforce` | PPO 技巧但无 Critic，更快 |
| **REINFORCE++-baseline** | `reinforce_baseline` | 均值奖励基线，推理任务首选 |
| **GRPO** | `group_norm` | 群组归一化 |
| **RLOO** | `rloo` | Leave-One-Out |
| **Dr. GRPO** | `dr_grpo` | 简化 GRPO，移除局部 /std |

### 3.5 Multi-Turn Agent 支持

```python
class AgentInstance(AgentInstanceBase):
    async def reset(self, states):  # 返回初始观察
        ...
    async def step(self, states):   # 返回 reward, done, feedback
        ...
```

支持 OpenAI-compatible Agent Server，可对接外部工具框架。

### 3.6 独特优势

- **REINFORCE++ 系列**：OpenRLHF 原创算法，比 GRPO 更稳定、比 PPO 更快（被 Magistral/Mistral、ProRL V2/NVIDIA 采用）
- **异步 RL**：`--train.async_enable` 支持生成和训练并行
- **Dynamic Filtering (DAPO)**：生成多个响应后按 reward 过滤
- **VLM RLHF**：视觉语言模型端到端 RL 训练
- **7B 单机可训练**：比 VeRL 硬件门槛更低

---

## 4. TRL — HuggingFace 生态原生 RL

### 4.1 基本信息

| 项目 | 详情 |
|------|------|
| **GitHub** | [huggingface/trl](https://github.com/huggingface/trl) |
| **Stars** | 18.7K+ |
| **许可证** | Apache 2.0 |
| **发起方** | HuggingFace |
| **v1.0 发布** | 2026.04 |

### 4.2 设计理念：Trainer 模式

TRL 的核心设计是**轻量 Trainer 封装**——每个算法对应一个 Trainer 类：

```python
from trl import SFTTrainer, DPOTrainer, GRPOTrainer, RewardTrainer

# SFT
trainer = SFTTrainer(model="Qwen2.5-0.5B", train_dataset=dataset)

# GRPO
trainer = GRPOTrainer(model="Qwen2.5-0.5B-Instruct", 
                       reward_funcs=accuracy_reward, ...)

# DPO
trainer = DPOTrainer(model="Qwen3-0.6B", train_dataset=dataset)
```

### 4.3 架构对比

| 维度 | TRL | VeRL | OpenRLHF |
|------|-----|------|----------|
| **设计模式** | Trainer 封装 | HybridFlow 数据流 | Agent-Based 执行 |
| **API 复杂度** | ⭐ 最低（~5 行代码） | ⭐⭐⭐ 最高 | ⭐⭐ 中等 |
| **分布式** | Accelerate (DDP/DeepSpeed/FSDP) | Ray 原生 | Ray + vLLM + DeepSpeed |
| **模型规模** | <70B（单机为主） | 671B+（多机集群） | 70B+（多机） |
| **HuggingFace 集成** | ⭐⭐⭐ 原生 | ⭐⭐ 兼容 | ⭐⭐ 兼容 |
| **适合用户** | 研究者、快速原型 | 大厂、大规模训练 | 中等规模、需要灵活性的团队 |

### 4.4 独特优势

- **HF 生态无缝**：transformers + datasets + peft + accelerate 全链路
- **CLI 一键训练**：`trl sft` / `trl dpo` / `trl grpo` 
- **Unsloth 加速**：集成优化内核
- **TRL v1.0**：2026.04 发布，统一后训练栈（SFT + RM + DPO + GRPO）
- **内置 Reward 函数**：`accuracy_reward`、`reasoning_accuracy_reward` 等开箱即用

---

## 5. ART + RULER — Agent 场景的通用奖励

### 5.1 基本信息

| 项目 | 详情 |
|------|------|
| **GitHub** | [OpenPipe/ART](https://github.com/OpenPipe/ART) |
| **Stars** | 9K+ |
| **核心创新** | RULER — LLM-as-Judge 替代手写 reward function |

### 5.2 解决的核心问题

**传统 Agent RL 的最大瓶颈**：Reward 函数编写。

- 数学/代码有可验证答案 → 写 `if answer == gold: return 1`
- RAG/客服/摘要等 Agent 任务 → 需要手写复杂的评分逻辑（数天迭代）

**RULER 的解决方案**：用 LLM-as-Judge 替代手写 reward。

```
传统方式:  手写 reward_function() → 调试 → 迭代（数天）
RULER方式:  写一段自然语言 Rubric → 直接训练（数分钟）
```

### 5.3 工作原理

```
1. GRPO 对每个 scenario 生成 4-8 条轨迹
2. RULER 将所有轨迹发给 Judge LLM（o3/Qwen3 32B）
3. Judge 根据 system prompt 中的目标进行相对评分（0-1）
4. GRPO 用群组归一化后的分数更新策略
```

**关键洞察**：LLM 不擅长绝对打分，但擅长"这 4 个回答哪个更好"的比较任务。GRPO 本身也只需要相对排序。

### 5.4 代码示例

```python
# 不需要手写 reward function！
from art.rewards import ruler_score_group

# system prompt 就是隐式 reward 函数
group = art.TrajectoryGroup([traj_a, traj_b, traj_c, traj_d])
judged = await ruler_score_group(group, "openai/o3")

# 可选：自然语言 Rubric
judged = await ruler_score_group(group, "openai/o3", 
    rubric="优先简洁回答，惩罚包含 emoji 的回复")
```

### 5.5 实用建议

- Judge 模型不一定要最贵的：Qwen3 32B 通常够用
- 推荐每组 4-8 条轨迹
- RULER 自动缓存，调试时不会重复调用 API

---

## 6. NeMo-Aligner — NVIDIA GPU 极致优化

| 项目 | 详情 |
|------|------|
| **GitHub** | NVIDIA/NeMo-Aligner |
| **Stars** | 5K+ |
| **后端** | Megatron-LM + TensorRT-LLM |
| **定位** | 企业级 RLHF，万亿参数模型 |

**特点**：
- Megatron-LM 原生支持，最大模型规模
- TensorRT-LLM 推理加速
- 适合已有 NVIDIA GPU 集群的企业

---

## 7. DeepSpeed-Chat — 微软的端到端方案

| 项目 | 详情 |
|------|------|
| **GitHub** | microsoft/DeepSpeed |
| **Stars** | 30K+（DeepSpeed 整体） |
| **定位** | 端到端 RLHF 训练方案 |

**特点**：
- DeepSpeed ZeRO-3 优化，极致内存效率
- 完整的 SFT → RM → PPO 三阶段
- 但更新较慢，社区活跃度不如 VeRL/OpenRLHF


---

## 7.5 AReaL — 蚂蚁/清华异步 RL 系统

| 项目 | 详情 |
|------|------|
| **GitHub** | [areal-project/AReaL](https://github.com/areal-project/AReaL) |
| **Stars** | 5.3K+ |
| **论文** | [arXiv:2505.24298](https://arxiv.org/abs/2505.24298) |
| **定位** | 大规模异步 RL 训练系统，完全解耦 Rollout 和 Train |

**核心创新**：完全异步架构——生成和训练彻底解耦，比同步系统快 **2.77 倍**。

```
传统同步:  Rollout → 等待完成 → Train → Rollout → ...（GPU 空闲等待）
AReaL 异步:  Rollout 流 ──→ 持续产生轨迹
             Train 流  ──→ 持续消费轨迹更新   （GPU 始终繁忙）
```

**特点**：
- 支持 13 种 RL 算法（GRPO/PPO/ARPO 等）、3 种训练后端、2 种推理引擎
- Megatron-LM / FSDP2 / Archon 训练后端可切换
- vLLM / SGLang 推理引擎可切换
- 蚂蚁 + 清华 IIIS 联合开发

> 详见 [AReaL 专题笔记](areal-asynchronous-rl-system-ant-group.md)

---

## 7.6 rLLM — UC Berkeley Agent RL 框架

| 项目 | 详情 |
|------|------|
| **GitHub** | [rllm-org/rllm](https://github.com/rllm-org/rllm) |
| **Stars** | 5.7K+ |
| **定位** | 不改 Agent 代码直接做 RL 训练 |

**核心创新**：Agent 代码零改动，rLLM 的 Model Gateway 自动拦截 LLM 调用，采集轨迹→打分→更新模型。

```
你的 Agent → rLLM Model Gateway（透明代理，自动采集 token IDs + logprobs）
              → 你的 Reward 函数 → GRPO/REINFORCE 更新模型
```

**特点**：
- 任意 harness：Claude Code、Codex、Terminus-2 等 10+ CLI
- 多后端：verl（分布式）、tinker（单机）、fireworks，一键切换
- 60+ 基准集成
- 明星成果：DeepScaleR-1.5B（超 O1-preview）、DeepCoder-14B（O3-mini 级）

> 详见 [Agent 构建框架对比](../development/agent-build-frameworks.md) 中 rLLM 章节

---

## 7.7 ASystem — 蚂蚁 Agent 平台（含 AReaL）

| 项目 | 详情 |
|------|------|
| **GitHub** | [inclusionAI](https://github.com/inclusionAI) |
| **定位** | 蚂蚁 inclusionAI 开源的 Agentic AI 平台 |

ASystem 是 AReaL 的上层平台，核心理念 "Everything as Environment"。旗下包含 AReaL（RL 训练）、AEnvironment（环境平台）、ASearcher（搜索 Agent）、AWorld（世界模拟）等组件。

> ASystem 是平台，不是 RL 训练框架本身。其 RL 训练能力由 AReaL 提供。详见 [ASystem 专题笔记](asystem-ant-group-agent-platform.md)


---

## 8. 其他值得关注的框架

| 框架 | 特色 |
|------|------|
| **verl-agent** | 长程 Agent RL + GiGPO 算法 |
| **OpenManus-RL** | 多 Agent 环境 RL 训练 |
| **RL-Factory** | Agent RL 即插即用，简洁 API |
| **Easy-R1** | 多模态 RL（VLM GRPO 训练） |
| **MARTI** | 基于 OpenRLHF 的多 Agent 系统 RL |
| **LMM-R1** | 多模态 DeepSeek-R1 复现（OpenRLHF fork）|

---

## 9. 选型决策树

```
需要训练什么？
│
├── 数学/代码推理模型 (>70B, 多机)
│   └── VeRL（SOTA 吞吐，工业验证）
│
├── 中等规模模型 (<70B, 单机或小集群)
│   ├── 想要 HuggingFace 生态 → TRL
│   └── 想要 Ray 分布式 + 更多算法 → OpenRLHF
│
├── Agent 场景（RAG/客服/工具调用）
│   ├── 有明确 reward 函数 → OpenRLHF（Agent-Based 执行）
│   └── 没有明确 reward → ART + RULER
│
├── 多模态（VLM）
│   ├── 快速上手 → Easy-R1
│   └── 大规模 → VeRL-Omni / OpenRLHF VLM
│
├── 需要异步 RL 训练（最大化 GPU 利用率）
│   └── AReaL（完全异步架构，2.77× 加速）
│
├── 已有 Agent 代码，想直接 RL 训练
│   └── rLLM（零代码改动接入）
│
└── NVIDIA GPU 集群，万亿参数
    └── NeMo-Aligner
```

---

## 10. 演进趋势

```
2022        2023        2024          2025           2026
 │           │           │             │              │
RLHF        DPO        GRPO         VeRL 爆发     Agent RL 元年
(Instruct)  (Stanford) (DeepSeek)   OpenRLHF      ART/RULER
 │           │           │           TRL v1.0      通用奖励
DeepSpeed   TRL        REINFORCE++  Easy-R1       VeRL-Omni
 Chat       (HF)       (OpenRLHF)   DAPO/VAPO     全模态
```

**关键趋势**：
1. **Critic 消亡**：PPO → REINFORCE++/GRPO，不再需要价值模型
2. **奖励民主化**：从手写代码 → LLM-as-Judge（RULER），Agent RL 的门槛大幅降低
3. **全模态 RL**：从纯文本 → VLM → 全模态（VeRL-Omni）
4. **Agent 原生**：从"对齐人类偏好" → "训练 Agent 完成复杂任务"
5. **$30 复现 R1**：TinyZero 证明了 RL 训练的民主化

---

## 更新记录

- 2026-06-29：新增 AReaL、rLLM、ASystem，更新选型决策树和演进时间线
- 2026-06-18：初始创建，覆盖 VeRL、OpenRLHF、TRL、ART(RULER)、NeMo-Aligner、DeepSpeed-Chat 等主流框架的深度剖析
