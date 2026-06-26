---
title: "AReaL：蚂蚁/清华异步 RL 训练系统"
created: 2026-06-26
updated: 2026-06-26
sources:
  - https://arxiv.org/abs/2505.24298
  - https://github.com/areal-project/AReaL
  - https://areal-ai.io
has_toc: true
tags: [AReaL, Asynchronous-RL, Ant-Group, Tsinghua, RL-System, Agent-Training]
category: ai
---

# AReaL：大规模异步强化学习系统

## 摘要

AReaL（Asynchronous Reinforcement Learning for Language）是蚂蚁集团 + 清华 IIIS 联合开发的大规模异步 RL 训练系统。核心创新：**完全异步架构**——生成（Rollout）和训练（Training）彻底解耦，比同步系统快 **2.77 倍**。支持 13 种 RL 算法、3 种训练后端、2 种推理引擎。5.3K GitHub Stars。

---

## 1. 背景：同步 RL 的效率瓶颈

### 1.1 现有同步系统的问题

VeRL、OpenRLHF 等主流框架都是**同步（Synchronous）架构**：

```
同步 RL（VeRL/OpenRLHF）:
  Batch N:
    → 等待所有 G 条轨迹生成完毕（最快的等最慢的）
    → 等最长序列的最后一个 token
    → 训练模型
  Batch N+1:
    → 重复...

问题：
  GPU 空闲时间 > 50%
  最长序列拖慢整个 batch
  生成和训练串行，无法重叠
```

### 1.2 AReaL 的异步架构

```
异步 RL（AReaL）:
  Rollout Workers:  持续生成 → 持续产出数据
  Training Workers:  有数据就训练 → 不等任何人
  ┌─────────────┐    ┌─────────────┐
  │  生成 Worker  │───→│  训练 Worker  │
  │  (不停产数据)  │    │  (不停消费数据) │
  └─────────────┘    └─────────────┘
         ↑                  │
         └──────────────────┘
          模型参数异步更新

优势：
  GPU 利用率 > 90%
  生成和训练完全重叠
  2.77× 加速（同 GPU 数量）
```

---

## 2. 核心技术：Staleness-Enhanced PPO

### 2.1 异步 RL 的挑战：数据陈旧性

异步架构的根本问题——训练时用的样本是"旧模型"生成的：

```
时间轴：
  t=0: 模型 M₀ 生成样本 S₁
  t=1: 模型 M₀ → M₁ (训练 S₁)
  t=2: 模型 M₁ 生成样本 S₂，但 S₁ 仍在队列中 → 用 M₁ 训练 M₀ 生成的 S₁

S₁ 是 "stale 样本" — 数据陈旧
```

### 2.2 Staleness-Enhanced PPO

AReaL 不是简单地忍受数据陈旧，而是**主动利用陈旧度信息**：

```
标准 PPO:  clip(ratio, 1-ε, 1+ε) · advantage

AReaL PPO: clip(ratio / staleness_weight, 1-ε, 1+ε) · advantage

staleness_weight = f(当前步数 - 生成步数)
  → 旧样本的 ratio 被下调 → 降低陈旧数据的影响
  → 新样本的 ratio 保持原值 → 充分信任新数据
```

**工作负载平衡**：AReaL 动态调整 Rollout Worker 和 Training Worker 的 GPU 配比，控制最大陈旧度。

---

## 3. 算法与后端支持

### 3.1 支持的 13 种 RL 算法

| 算法 | 来源 |
|------|------|
| PPO | OpenAI 2017 |
| GRPO | DeepSeek 2024 |
| GSPO | Alibaba 2025 |
| DAPO | ByteDance 2025 |
| SAPO | Alibaba 2025 |
| REINFORCE++ | 社区 |
| RLOO | 社区 |
| Dr.GRPO | 社区 |
| LitePPO | 社区 |
| IcePop | AReaL 原创 |
| KPop | AReaL 原创 (2026.06) |
| M2PO | 社区 |
| DPO | Stanford 2023 |

### 3.2 训练 & 推理后端

| 后端 | 类型 | 特点 |
|------|------|------|
| Megatron-LM | 训练 | TP/PP/CP/EP 全支持，MoE 优化 |
| PyTorch FSDP2 | 训练 | 易用，HuggingFace 兼容 |
| PyTorch Archon | 训练 | FSDP2 + PP + EP 组合 |
| vLLM | 推理 | TP + PP |
| SGLang | 推理 | TP + DP Attention + EP |

---

## 4. AReaL-lite：轻量版

AReaL-lite 是专为研究者设计的轻量版本：

| | AReaL 完整版 | AReaL-lite |
|---|---|---|
| **代码量** | 100% | **20%** (少 80%) |
| **性能** | 100% | **90%** |
| **API 设计** | 生产级 | **算法优先** |
| **异步 Agent RL** | ✅ | ✅ |
| **适用** | 大规模训练 | 快速原型 & 算法研究 |

---

## 5. Agent RL 应用

AReaL 特别强调 Agent 训练场景：

| 应用 | 详情 |
|------|------|
| **AReaL-SEA** | 自进化数据合成引擎，235B MoE 超 GPT-5，匹敌 Gemini 3.0 Pro (τ²-bench) |
| **ASearcher** | 端到端搜索 Agent，异步 RL 训练 |
| **OpenClaw 训练** | 替换 base_url 即可训练任何 Agent |
| **CAMEL-AI / SETA** | 终端 Agent RL 训练 |
| **Scaffoldings** | NVIDIA TensorRT-LLM 集成 |
| **OpenAI Agents SDK** | 原生集成 |

---

## 6. AReaL vs VeRL vs OpenRLHF

| | VeRL | OpenRLHF | **AReaL** |
|---|---|---|---|
| **架构** | 同步 | 同步 + 可选异步 | **完全异步** |
| **速度** | 1× | 1× | **2.77×** |
| **GPU 利用率** | ~50% | ~50% | **>90%** |
| **算法数** | 10+ | 6 | **13** |
| **Agent RL** | uni-agent | Multi-turn Agent | **Agent RL Bridge（原生）** |
| **训练后端** | FSDP/FSDP2/Megatron | DeepSpeed | **Megatron/FSDP2/Archon** |
| **Stars** | 22K | 9.7K | 5.3K |
| **发起方** | ByteDance | 社区 | **Ant Group + 清华 IIIS** |

---

## 更新记录

- 2026-06-26：初始创建
