---
title: "主流 Agent 构建框架对比（2026）"
created: 2026-06-27
updated: 2026-06-27
sources:
  - https://github.com/huggingface/smolagents
  - https://github.com/openai/openai-agents-python
  - https://github.com/crewAIInc/crewAI
  - https://github.com/microsoft/autogen
  - https://github.com/rllm-org/rllm
tags: [Agent, Agent-Framework, smolagents, CrewAI, AutoGen, OpenAI-Agents-SDK, rLLM]
category: development
---

# 主流 Agent 构建框架对比（2026）

## 摘要

Agent 构建框架分为两个赛道：**Agent 构建/编排**（定义 Agent 行为、工具调用、多 Agent 协作）和 **Agent RL 训练**（用强化学习训练 Agent 的策略）。本文覆盖两大赛道 5 个主流框架。

---

## 一、Agent 构建 / 编排框架

### smolagents（Hugging Face）
**28.1k⭐ | Apache 2.0 | [github](https://github.com/huggingface/smolagents)**

核心理念：**Agent 用 Python 代码思考**（Code-as-Action），核心代码 <1000 行。

```python
from smolagents import CodeAgent, WebSearchTool
agent = CodeAgent(tools=[WebSearchTool()], model=model)
agent.run("How many seconds for a leopard to run through Pont des Arts?")
```

- **CodeAgent**：LLM 输出 Python 代码片段作为 Action，比 JSON 工具调用少 30% 步骤
- **模型无关**：OpenAI、Anthropic、HF Hub、本地 transformers/ollama
- **工具无关**：MCP server、LangChain 工具、Hub Space
- **沙箱支持**：E2B、Modal、Docker

### OpenAI Agents SDK
**27.5k⭐ | MIT | [github](https://github.com/openai/openai-agents-python)**

轻量级多 Agent 编排框架，核心概念：Agents + Handoffs + Guardrails + Tracing。

```python
from agents import Agent, Runner
agent = Agent(name="Assistant", instructions="You are helpful", model="gpt-4.1")
result = Runner.run_sync(agent, "Hello World!")
```

- **Sandbox Agents**（v0.14+）：Agent 在沙箱容器中执行长时间任务
- **Handoffs**：Agent 之间委托任务
- **Guardrails**：输入/输出安全检查
- **Sessions**：自动对话历史管理
- **Tracing**：内置运行追踪和调试

### CrewAI
**54.5k⭐ | MIT | [github](https://github.com/crewAIInc/crewAI)**

角色扮演式多 Agent 编排，提供 Crews（自主协作）和 Flows（事件驱动工作流）两套范式。

```yaml
# agents.yaml
researcher:
  role: "Senior Data Researcher"
  goal: "Uncover cutting-edge developments in {topic}"
```

- **Crews**：基于角色的自主 Agent 协作
- **Flows**：事件驱动、状态管理、条件分支
- **10 万+ 认证开发者**，企业级 AMP 套件
- 强项：生产级多 Agent 自动化

### AutoGen（Microsoft）⚠️ 维护模式
**59.3k⭐ | CC-BY-4.0/MIT | [github](https://github.com/microsoft/autogen)**

多 Agent 框架先驱，现已进入维护模式，**官方推荐迁移到 [Microsoft Agent Framework](https://github.com/microsoft/agent-framework)**。

- 分层架构：Core API → AgentChat API → Extensions API
- AutoGen Studio：无代码 GUI
- Magentic-One：多 Agent 团队实现

---

## 二、Agent RL 训练框架

### rLLM（UC Berkeley）
**5.7k⭐ | Apache 2.0 | [github](https://github.com/rllm-org/rllm)**

**不改 Agent 代码，直接做 RL 训练**的框架。核心理念：Agent 代码零改动，rLLM 自动采集轨迹→ 打分→ 更新模型。

```
你的 Agent → rLLM Model Gateway 自动采集轨迹 → 你的 Reward 函数 → GRPO/REINFORCE 更新
```

- **任意 harness**：Claude Code、Codex、Terminus-2 等 10+ CLI 工具
- **多训练后端**：verl（分布式多 GPU）、tinker（单机）、fireworks，一键切换
- **60+ 基准**：AIME、SWE-bench、Terminal-Bench 2.0
- **明星成果**：DeepScaleR-1.5B（超 O1-preview）、DeepCoder-14B（O3-mini 级）

> 知识库中已有 **AReaL**（蚂蚁 RL 系统）、**OpenRLHF**、**VeRL** 等同类 RL 训练框架，详见 [Agent RL 框架生态](../ai/agent-rl-frameworks-ecosystem.md)。

---

## 三、全景对比

### Agent 构建框架

| | smolagents | OpenAI Agents SDK | CrewAI | AutoGen |
|---|---|---|---|---|
| **开发者** | Hugging Face | OpenAI | CrewAI Inc | Microsoft |
| **Stars** | 28.1k | 27.5k | 54.5k | 59.3k |
| **核心范式** | Code-as-Action | Handoffs + Guardrails | Crews + Flows | Core + AgentChat |
| **代码量** | <1000 行 | 中等 | 中等 | 大 |
| **沙箱** | E2B/Modal/Docker | ✅ Sandbox Agents | ❌ | ✅ |
| **MCP 支持** | ✅ | ✅ | ✅ | ✅ |
| **状态** | 活跃 | 活跃 | 活跃 | ⚠️ 维护模式 |

### Agent RL 训练框架 vs Agent 构建框架

| 赛道 | 代表框架 | 核心问题 |
|------|---------|---------|
| **Agent 构建** | smolagents, CrewAI, OpenAI SDK | 如何定义 Agent 的行为？ |
| **Agent RL 训练** | rLLM, AReaL, VeRL, OpenRLHF | 如何让 Agent 自己变强？ |

两者互补：Agent 构建框架定义 Agent → Agent RL 训练框架用 RL 优化 Agent 策略。

---

## 更新记录

- 2026-06-27：初始创建，覆盖 smolagents、OpenAI Agents SDK、CrewAI、AutoGen、rLLM
