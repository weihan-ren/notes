---
title: "ASystem：蚂蚁 inclusionAI 开源 Agent 平台"
created: 2026-06-26
updated: 2026-06-26
sources:
  - https://github.com/inclusionAI
  - https://github.com/inclusionAI/AEnvironment
  - https://github.com/areal-project/AReaL
  - https://www.inclusion-ai.org/AEnvironment
has_toc: true
tags: [ASystem, Ant-Group, inclusionAI, Agent-Platform, AEnvironment, AReaL]
category: ai
---

# ASystem：蚂蚁 inclusionAI 开源 Agent 平台

## 摘要

ASystem 是蚂蚁集团 inclusionAI 推出的开源 Agentic AI 平台，核心理念 "Everything as Environment"。旗下包含多个组件：AReaL（RL 训练）、AEnvironment（环境平台）、ASearcher（搜索 Agent）、AWorld（世界模拟）等，覆盖从环境构建到 RL 训练再到 Agent 部署的完整链路。

---

## 1. ASystem 组件全景

```
┌──────────────────────────────────────────────┐
│                  ASystem                      │
│          (蚂蚁 inclusionAI AGI 开源平台)       │
│                                              │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐     │
│  │AReaL    │ │AEnviron- │ │ASearcher │     │
│  │RL 训练   │ │ment (AEnv)│ │搜索 Agent │     │
│  │5.3K⭐   │ │环境平台   │ │端到端RL   │     │
│  └──────────┘ └──────────┘ └──────────┘     │
│                                              │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐     │
│  │AWorld   │ │AReaL-SEA │ │...更多组件│     │
│  │世界模拟   │ │自进化数据 │ │          │     │
│  └──────────┘ └──────────┘ └──────────┘     │
└──────────────────────────────────────────────┘
```

---

## 2. AEnvironment（AEnv）— 统一环境平台

### 2.1 核心哲学："Everything as Environment"

AEnvironment 是 ASystem 第 5 个开源组件，定位为 **Agentic RL 时代的统一环境平台**：

```
传统 RL 环境：
  Gymnasium → 标准化的状态/动作/奖励接口

AEnvironment（扩展后）：
  Everything → 标准化环境接口
    ├── Web 环境（浏览器交互）
    ├── 代码执行环境（沙盒）
    ├── 数据库查询环境
    ├── API 调用环境
    ├── 多 Agent 交互环境
    └── MCP 服务器环境
```

### 2.2 功能特性

| 特性 | 说明 |
|------|------|
| **标准化环境** | 统一的 MCP 协议接口，适合 Agent 训练 |
| **沙盒执行** | 安全的代码/命令执行环境 |
| **Web 交互** | 浏览器自动化 + DOM 解析 |
| **数据合成** | 与训练流水线集成，生成训练数据 |
| **可复用** | 环境定义可跨项目复用 |

---

## 3. 组件间关系

```
AWorld (世界模拟)
  │ 生成任务/场景
  ↓
AEnvironment (AEnv)
  │ 提供标准化的状态/动作/奖励
  ↓
AReaL (异步 RL 训练)
  │ 训练 Agent 策略
  ↓
ASearcher / AReaL-SEA / SETA (部署)
  │ 训练好的 Agent 部署到生产
```

---

## 4. 与知识库关联

| ASystem 概念 | 知识库概念 |
|-------------|-----------|
| AEnvironment | MCP 服务器（环境抽象） |
| AReaL（异步 RL） | VeRL/OpenRLHF（同步 RL 对比） |
| AReaL-SEA（自进化） | RLVR（可验证奖励 → Agent 奖励） |
| ASearcher | Tree-GRPO / ARPO（搜索 Agent RL） |
| "Everything as Environment" | 黑盒 Agent（环境抽象视角） |

---

## 5. 与其他平台对比

| | ASystem | VeRL 生态 | OpenRLHF |
|---|---|---|---|
| **定位** | **端到端 Agent 平台** | RL 训练框架 | RL 训练框架 |
| **环境支持** | ✅ AEnvironment 原生 | ❌ 需外部环境 | ❌ 需外部环境 |
| **搜索 Agent** | ✅ ASearcher 原生 | ❌ 需要社区项目 | ❌ |
| **数据合成** | ✅ AReaL-SEA | ❌ | ❌ |
| **RL 训练** | AReaL（异步） | VeRL（同步） | OpenRLHF（同步+异步） |
| **算法数** | 13 | 10+ | 6 |

---

## 更新记录

- 2026-06-26：初始创建
