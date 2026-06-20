---
title: "主流 Agent CLI 工具深度对比分析（2026）"
nav_order: 20
has_toc: true
created: 2026-06-18
updated: 2026-06-18
sources:
  - https://www.marktechpost.com/2026/06/14/claude-code-guide-2026-25-features-with-examples-demo/
  - https://mcp.directory/blog/claude-skills-vs-mcp-vs-subagents-vs-cli-2026-decision-matrix
  - https://alexop.dev/posts/understanding-claude-code-full-stack/
  - https://www.firecrawl.dev/blog/best-codex-skills
  - https://openai.com/codex/
  - https://github.com/opencode-ai
  - https://github.com/agentskills/agentskills
  - https://openhands.dev/
  - https://github.com/OpenHands/openhands
  - https://vibecoding.app/blog/openhands-review
tags: [Agent-CLI, Claude-Code, Codex, Opencode, Cursor, MCP, Skills, Subagents, Tools]
             OpenHands
             (Agent 聚合平台)

category: development
---

# 主流 Agent CLI 工具深度对比分析（2026）

## 摘要

2026 年 AI 编程 Agent 已从单一对话工具演化为分层架构的智能体系统。本文深度对比 Claude Code、Codex、Opencode 等主流 Agent CLI 的架构设计、扩展机制（Skills/MCP/Subagents/Hooks/Plugins）、工具系统和 Agent 团队协作能力。
             OpenHands
             (Agent 聚合平台)


---

## 1. 整体格局

| 工具 | 厂商 | 开源 | 核心模型 | 发布 |
|------|------|:---:|---------|------|
| **Claude Code** | Anthropic | ❌ | Claude Opus/Sonnet/Haiku | 2025.02 |
| **Codex** | OpenAI | ❌ | GPT-5.x / o4 | 2025.06 |
| **Opencode** | 社区开源 | ✅ | 多模型（支持 DeepSeek/Claude/GPT 等） | 2025 |
             OpenHands
             (Agent 聚合平台)

| **OpenHands** | ✅ | ✅ | ✅ Docker 沙盒 | ✅ | ✅ | ✅ | ✅ MCP | ✅ | ✅ ACP | ✅ | ✅ | ✅ |
| **OpenHands** | `.agents/skills/` | Agentskills 标准 | ✅ | ✅ 支持 | ✅ | 社区 | `npx skills add` |
| **OpenHands** | ACP agents/*.yml | 多 Agent 聚合 | 多种 | Agent Server REST API | ✅ ACP 协议 |
| **OpenHands** | All-Hands-AI | ✅ (MIT) | 多模型 | 2024（OpenDevin → 2025 OpenHands） |
| **Cursor** | Cursor Inc | ❌ | 多模型 | 2024 |
| **Gemini CLI** | Google | ❌ | Gemini 3.x | 2025 |
| **Aider** | 社区开源 | ✅ | 多模型 | 2023 |
| **Cline** | 社区开源 | ✅ | 多模型 | 2024 |
| **Augment Code** | Augment | ❌ | 自有模型 | 2025 |
| **Windsurf** | Codeium | ❌ | 自有模型 | 2024 |
| **GitHub Copilot** | Microsoft | ❌ | GPT/Claude 多模型 | 2024 |

---

## 2. 架构设计对比

### 2.1 Claude Code 架构

```
┌──────────────────────────────────────────┐
│              Claude Code Agent           │
│                                          │
│  ┌────────┐  ┌──────────┐  ┌──────────┐ │
│  │CLAUDE.md│  │ Skills   │  │Subagents │ │
│  │(Memory) │  │(SKILL.md)│  │(.claude/ │ │
│  │         │  │          │  │ agents/) │ │
│  └────────┘  └──────────┘  └──────────┘ │
│                                          │
│  ┌────────┐  ┌──────────┐  ┌──────────┐ │
│  │ Hooks  │  │ Plugins  │  │   MCP    │ │
│  │(Events)│  │(Bundles) │  │ Servers  │ │
│  └────────┘  └──────────┘  └──────────┘ │
│                                          │
│  Core Loop: Read → Think → Tool → Output │
│  Permission: Plan/Accept/Auto modes      │
│  Context: Progressive Compaction         │
└──────────────────────────────────────────┘
```

**核心设计理念**：分层 Agentic 系统

**六层扩展原语**：
1. **CLAUDE.md** — 持久化项目记忆（层级解析：企业 → 用户 → 项目 → 目录）
2. **Skills** — 领域知识 + 可执行脚本（`SKILL.md` 渐进式披露）
3. **Subagents** — 独立上下文窗口的专用工作进程
4. **Hooks** — 生命周期事件驱动（PreToolUse/PostToolUse 等 15+ 事件）
5. **Plugins** — 以上所有原语的版本化打包
6. **MCP** — 连接外部系统的通用协议

**关键特性**：
- 渐进式上下文压缩（`/compact`）
- Auto Mode（Sonnet 4.6 安全分类器）
- Checkpoints（Escape 两次回滚）
- Agent SDK（`query()` 编程接口）
- Headless CLI（`claude -p` 一次性执行）
- Worktree 隔离（子 agent 独立 git worktree）

### 2.2 Codex 架构

```
┌──────────────────────────────────────────┐
│              Codex Agent                 │
│                                          │
│  ┌────────┐  ┌──────────┐  ┌──────────┐ │
│  │AGENTS.md│  │ Skills   │  │  Agents  │ │
│  │(Memory) │  │(.agents/ │  │(.codex/  │ │
│  │         │  │ skills/) │  │ agents/) │ │
│  └────────┘  └──────────┘  └──────────┘ │
│                                          │
│  ┌────────┐  ┌──────────┐  ┌──────────┐ │
│  │ Hooks  │  │ Plugins  │  │   MCP    │ │
│  │(Events)│  │(Bundles) │  │ Servers  │ │
│  └────────┘  └──────────┘  └──────────┘ │
│                                          │
│  Goal Mode: /goal → 持续工作至里程碑      │
│  Worktrees: 并行隔离开发环境              │
│  $skill-creator: 交互式技能构建器         │
└──────────────────────────────────────────┘
```

**核心设计理念**：与 Claude Code 高度对标的分层架构

**配置目录**：
- `$HOME/.agents/skills/` — 个人 Skills
- `.agents/skills/` — 项目 Skills
- `.codex/config.toml` — 项目配置
- `.codex/hooks.json` — 项目 Hooks
- `.codex/agents/` — 项目 Agents

**Codex 独有特性**：
- **Goal Mode**（`/goal`）：设定里程碑，持续工作数小时/数天
- **Worktrees**：并行隔离开发，多 agent 同时工作
- **System Skills**：`skill-creator`、`plugin-creator`、`skill-installer`、`imagegen`、`openai-docs`
- **$ 语法**：`$skill-name` 显式调用（区别于 Claude Code 的 `/skill-name`）
- **跨 Agent 迁移**：`migrate-to-codex` 从 Claude Code 迁移

### 2.3 Opencode 架构
             OpenHands
             (Agent 聚合平台)


```
┌──────────────────────────────────────────┐
│             Opencode Agent               │
             OpenHands
             (Agent 聚合平台)

│                                          │
│  ┌────────────┐  ┌─────────────────────┐ │
│  │opencode.   │  │ agents/             │ │
│  │jsonc       │  │   research.md       │ │
│  │(主配置)     │  │(primary/explore/   │ │
│  │            │  │ general 类型)        │ │
│  └────────────┘  └─────────────────────┘ │
│                                          │
│  ┌────────────┐  ┌─────────────────────┐ │
│  │ skills/    │  │ MCP (本地/远程)      │ │
│  │SKILL.md    │  │ playwright, etc.    │ │
│  │(Markdown+  │  │                     │ │
│  │ YAML)      │  │                     │ │
│  └────────────┘  └─────────────────────┘ │
│                                          │
│  权限：精细路径级（edit/bash/external）    │
│  Shell: 可配置（/bin/zsh, pwsh）          │
│  Task: 子 agent 并行执行                  │
└──────────────────────────────────────────┘
```

**核心设计理念**：开源、可定制、路径级安全控制

**Opencode 独有优势**：
             OpenHands
             (Agent 聚合平台)

1. **完全开源** — 社区驱动，可自托管
2. **多模型支持** — 不绑定单一厂商（DeepSeek、Claude、GPT 等）
3. **精细权限** — path 级别的 edit/bash/external_directory 控制
4. **原生知识库** — Research Agent 内置持久化知识管理
5. **Shell 可配置** — 支持 bash/zsh/pwsh
6. **轻量无依赖** — 独立二进制分发

**Skill 格式**：
```yaml
---
name: skill-name
description: 触发条件描述
---
# 指令内容
```

无需特定的目录结构约定，`skills/` 下任意放置。

---


### 2.4 OpenHands 架构

```
┌──────────────────────────────────────────┐
│          OpenHands Platform              │
│                                          │
│  ┌────────────┐  ┌─────────────────────┐ │
│  │ Agent Canvas│  │ Agent Server (REST) │ │
│  │ (Web GUI)   │  │ - 多 Agent 运行      │ │
│  │ - 会话管理   │  │ - Docker 沙盒隔离    │ │
│  │ - 自动化任务  │  │ - ACP 协议互通      │ │
│  └────────────┘  └─────────────────────┘ │
│                                          │
│  ┌────────────┐  ┌─────────────────────┐ │
│  │ Skills      │  │ Automations         │ │
│  │ (.agents/   │  │ (Slack/GitHub/      │ │
│  │  skills/)   │  │  Schedule/Webhook)  │ │
│  └────────────┘  └─────────────────────┘ │
│                                          │
│  后端：Python (63.5%) + TypeScript (35%) │
│  沙盒：Docker-in-Docker                  │
│  协议：ACP (Agent-Client Protocol)        │
└──────────────────────────────────────────┘
```

**核心设计理念**：Agent 控制中心 + 多 Agent 编排平台

**OpenHands 独有优势**：
1. **Agent 聚合器** — 一个平台运行 OpenHands/Claude Code/Codex/Gemini 等多种 Agent
2. **ACP 协议** — Agent-Client Protocol，统一不同 Agent 的通信接口
3. **Automations** — Slack/GitHub/Linear 触发 + 定时任务 + Webhook，Agent 持续在线
4. **Agent Server** — REST API 架构，Agent 作为服务运行在服务器上（非仅本机 CLI）
5. **77.6K GitHub Stars** — 社区最大的开源 Agent 项目
6. **$18.8M 融资** — 企业级商业化路线清晰
7. **Sandbox 原生** — Docker 隔离执行，Agent 不可触及宿主机
8. **SWE-bench Verified 53%+** — 基准测试与 Devin 持平

**与前三个工具的本质区别**：
- Claude Code / Codex / Opencode 是"你用 Agent"——在终端里和单一 Agent 交互
             OpenHands
             (Agent 聚合平台)

- OpenHands 是"你管理 Agent 团队"——Agent 作为持续运行的服务，被调度、监控和管理

**Skills 支持**：`.agents/skills/` 目录，兼容 Agentskills 开放标准
**MCP 支持**：支持连接外部 MCP 服务器
**Agent 定义**：通过 `AGENTS.md` + Skills + 后端配置
**多 Agent 编排**：Automations 触发多个 Agent 并行执行，Slack/GitHub 作为交互入口

## 3. Agent 特性对比

| 特性 | Claude Code | Codex | Opencode |
             OpenHands
             (Agent 聚合平台)

|------|:-----------:|:-----:|:--------:|
| **Agent 定义** | `.claude/agents/*.md` | `.codex/agents/*.md` | `agents/*.md` |
| **Primary Agent** | ✅ 内置 | ✅ 内置 | ✅ 需配置 mode: primary |
| **Explore Agent** | ✅ 内置类型 | ❌ 无内置 | ✅ explore 类型 |
| **Plan Agent** | ✅ 内置类型 | ✅ Plan Mode | ❌ 无内置 |
| **General Agent** | ✅ 内置类型 | ✅ 默认 | ✅ general 类型 |
| **Agent 工具限制** | ✅ tools 字段 | ✅ 继承主 agent | ✅ permission 字段 |
| **Agent 模型覆盖** | ✅ model 字段 | ✅ 支持 | ❌ 依赖全局配置 |
| **Agent 上下文隔离** | ✅ context: fork | ✅ Worktree | ❌ 共享上下文 |
| **Task 委托** | ✅ Auto-delegate | ✅ $ 显式调用 | ✅ task 工具 |

### Agent 定义示例对比

**Claude Code**:
```yaml
---
name: security-auditor
description: Analyzes code for security vulnerabilities
tools: Read, Grep, Bash
model: sonnet
---
You are a security-focused code auditor...
```

**Codex**:
```yaml
---
name: security-auditor
description: Analyzes code for security vulnerabilities
---
You are a security-focused code auditor...
```

**Opencode**:
             OpenHands
             (Agent 聚合平台)

```yaml
---
description: Security analysis agent
mode: general
permission:
  edit: "*" deny
  bash: allow
---
You are a security-focused code auditor...
```

---

## 4. Skill 机制对比

| 特性 | Claude Code | Codex | Opencode |
             OpenHands
             (Agent 聚合平台)

|------|:-----------:|:-----:|:--------:|
| **Skill 文件** | `SKILL.md` | `SKILL.md` | `SKILL.md` |
| **安装位置** | `.claude/skills/` | `.agents/skills/` | `skills/` |
| **YAML 前置** | name + description | name + description | name + description |
| **自动调用** | ✅ 描述匹配 | ✅ 描述匹配 | ✅ 描述匹配 |
| **显式调用** | `/skill-name` | `$skill-name` | 加载到上下文 |
| **渐进式加载** | ✅ 只加载 frontmatter | ✅ capped budget | ❌ 全量加载 |
| **子 agent 运行** | ✅ context: fork | ✅ 支持 | ❌ 不支持 |
| **路径过滤** | ✅ paths glob | ✅ 支持 | ❌ 不支持 |
| **禁用自动调用** | ✅ disable-model-invocation | ✅ 支持 | ❌ 不支持 |
| **跨 Agent 移植** | ✅ 开放标准 | ✅ 开放标准 | ✅ 兼容格式 |
| **官方 Skills 仓库** | anthropics/skills | openai/skills（19.3k⭐） | 社区贡献 |
| **生态安装 CLI** | `npx skills add` | `npx skills add` | 手动 |

---

## 5. MCP 支持对比

| 特性 | Claude Code | Codex | Opencode |
             OpenHands
             (Agent 聚合平台)

|------|:-----------:|:-----:|:--------:|
| **MCP 服务器** | ✅ `.mcp.json` | ✅ `.codex/config.toml` | ✅ `opencode.jsonc` |
| **本地 stdio** | ✅ | ✅ | ✅ |
| **远程 HTTP/SSE** | ✅ OAuth 2.1 | ✅ | ⚠️ 有限 |
| **Tool Search（延迟加载）** | ✅ 减少上下文 | ✅ 支持 | ❌ |
| **Code Mode（99.9% token 节省）** | ✅ Cloudflare | ❌ | ❌ |
| **内置 MCP** | Playwright, Filesystem | Playwright, GitHub | Playwright |

---

## 6. Subagents / Agent Team 对比

| 特性 | Claude Code | Codex | Opencode |
             OpenHands
             (Agent 聚合平台)

|------|:-----------:|:-----:|:--------:|
| **Subagents** | ✅ `.claude/agents/` | ✅ `.codex/agents/` | ⚠️ task 工具 |
| **Agent Teams（对等协作）** | ✅ 多 session | ✅ Goal Mode + Worktree | ❌ |
| **并行执行** | ✅ context: fork | ✅ Worktrees | ⚠️ task 并行 |
| **工作树隔离** | ✅ worktree isolation | ✅ Worktrees | ❌ |
| **子 agent 上下文隔离** | ✅ 独立窗口 | ✅ 独立 Worktree | ❌ 共享 |
| **调度/定时任务** | ✅ `/schedule` + Triggers | ❌ | ❌ |
| **后台任务** | ✅ run_in_background | ✅ 支持 | ❌ |

---

## 7. Tools / 工具系统对比

| 工具 | Claude Code | Codex | Opencode |
             OpenHands
             (Agent 聚合平台)

|------|:-----------:|:-----:|:--------:|
| **Read** | ✅ | ✅ | ✅ |
| **Write/Edit** | ✅ | ✅ | ✅ |
| **Bash/Shell** | ✅ 含沙盒 | ✅ 含沙盒 | ✅ 可配置 |
| **Grep** | ✅ 内置 | ✅ 内置 | ✅ 内置 |
| **Glob** | ✅ | ✅ | ✅ |
| **WebFetch** | ✅ | ✅ | ✅ |
| **WebSearch** | ✅ | ⚠️ 缓存搜索 | ⚠️ |
| **Playwright Browser** | ✅ MCP | ✅ MCP | ✅ MCP |
| **Task（子 agent）** | ✅ Auto-delegate | ✅ $ 调用 | ✅ task 工具 |
| **TodoWrite** | ✅ | ✅ | ✅ |
| **Question（用户交互）** | ✅ | ✅ | ✅ |
| **Skill 加载** | ✅ 自动 | ✅ 自动 | ✅ 自动 |

---

## 8. Hooks 系统对比

| 特性 | Claude Code | Codex | Opencode |
             OpenHands
             (Agent 聚合平台)

|------|:-----------:|:-----:|:--------:|
| **PreToolUse** | ✅ | ✅ | ❌ |
| **PostToolUse** | ✅ | ✅ | ❌ |
| **SessionStart/Stop** | ✅ | ✅ | ❌ |
| **PreCompact/PostCompact** | ✅ | ❌ | ❌ |
| **Notification** | ✅ 推送通知 | ❌ | ❌ |
| **Hook 类型** | Command/Prompt/HTTP | Command/Prompt | ❌ |

---

## 9. 安全与权限对比

| 特性 | Claude Code | Codex | Opencode |
             OpenHands
             (Agent 聚合平台)

|------|:-----------:|:-----:|:--------:|
| **权限模式** | Plan/Accept/Auto | 类似 | 路径级 deny/allow/ask |
| **Checkpoints** | ✅ Escape 回滚 | ✅ | ❌ |
| **沙盒 Bash** | ✅ OS 级隔离 | ✅ | ❌ |
| **Auto Mode 安全分类器** | ✅ Sonnet 4.6 | ❌ | ❌ |
| **PreToolUse 钩子** | ✅ 阻止危险命令 | ✅ | ❌ |

---

## 10. 生态与社区

| 维度 | Claude Code | Codex | Opencode |
             OpenHands
             (Agent 聚合平台)

|------|:-----------:|:-----:|:--------:|
| **Skills 生态** | MCP.Directory + anthropics/skills | openai/skills（19.3k⭐） | 社区 |
| **Skills 市场** | skills.sh（Vercel） | skills.sh（Vercel） | 无 |
| **MCP 生态** | 1653+ 发布者 | 共享 MCP 生态 | 共享 MCP 生态 |
| **插件生态** | ✅ /plugin 安装 | ✅ plugin-creator | ❌ |
| **CI/CD 集成** | ✅ GitHub Action | ✅ | ❌ |
| **多平台** | CLI/IDE/Desktop/Web/iOS | CLI/IDE/Desktop | CLI |
| **Agent SDK** | ✅ query() API | ✅ | ❌ |

---

## 11. 选型建议

| 场景 | 推荐 | 理由 |
|------|------|------|
| **全栈开发，复杂项目** | Claude Code | 最成熟的 subagent/context 管理 |
| **长时间持续工作** | Codex | Goal Mode 支持小时/天级持续工作 |
| **开源/自托管需求** | Opencode | 完全开源，无厂商锁定 |
             OpenHands
             (Agent 聚合平台)

| **多 Agent 并行** | Claude Code / Codex | Worktree 隔离 + 并行执行 |
| **注重隐私/本地** | Opencode / Aider | 开源，可本地部署 |
             OpenHands
             (Agent 聚合平台)

| **简单快速上手** | Aider / Cline | 轻量，开箱即用 |
| **VS Code 深度集成** | Cursor / Copilot | IDE 原生体验 |
| **预算敏感** | Opencode / Cline | 开源，支持多模型选择 |
             OpenHands
             (Agent 聚合平台)


---

## 12. 架构演进趋势

```
2023          2024              2025              2026
 │              │                 │                 │
Copilot      Cursor           Claude Code      统一 Skill 标准
(Autocomplete) (Agent Tab)     Codex CLI        (agentskills)
             Aider             Opencode         MCP 2.0 (OAuth)
             OpenHands
             (Agent 聚合平台)

             Cline             Gemini CLI       延迟 Tool Loading
                               Windsurf         Agent Teams
                                                Goal Mode
                                                Triggers/Scheduling
```

**关键趋势**：
1. **Skill 标准化**：`SKILL.md` 成为跨工具开放标准（Agentskills spec，Apache 2.0）
2. **MCP 演进**：从 stdio 到远程 HTTP + OAuth 2.1，延迟工具加载解决 token 膨胀
3. **Agent 分层**：主 Agent → Subagent → Agent Team 的协作层级
4. **上下文工程**：渐进式加载、Compaction、Worktree 隔离成为标配
5. **安全深化**：Sandbox、Checkpoint、安全分类器、PreToolUse 检查

---

## 更新记录

- 2026-06-18：初始创建，覆盖 Claude Code/Codex/Opencode 三大工具的战略对比
             OpenHands
             (Agent 聚合平台)

