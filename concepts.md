---
title: 概念字典
nav_exclude: false
nav_order: 99
has_toc: true
---

# 概念字典

快速查阅 AI/ML/Agent 领域核心概念。

## 强化学习 (RL)

| 术语 | 英文 | 解释 |
|------|------|------|
| 策略 | Policy (π) | 给定状态，选择动作的决策规则。LLM 本身就是一个策略 |
| 轨迹 | Trajectory | S₀→A₀→R₁→S₁→...→S_T，Agent 与环境的完整交互路径 |
| 步 | Step | 轨迹中的单次决策/动作。Token 级 RL 中是生成一个 token |
| 群组 | Group | 同一 prompt 下的多条轨迹集合（GRPO 中 G=16+） |
| 奖励 | Reward | 环境对 Agent 动作的标量反馈（好=正，差=负） |
| 优势 | Advantage A(s,a) | "这个动作比平均好/差多少"，是 RL 的核心训练信号 |
| 折扣因子 | Discount Factor γ | 未来奖励的权重衰减（0.99=看重长期） |
| On-Policy | On-Policy | 用当前策略采样的数据训练，更新后旧数据作废 |
| Off-Policy | Off-Policy | 用历史策略数据训练，样本效率高但不稳定 |
| Actor-Critic | Actor-Critic | Actor 负责决策（策略），Critic 负责评估（价值函数） |
| 信用分配 | Credit Assignment | 把终端奖励分配到中间步骤——"哪一步做对了？" |

## RL 算法

| 术语 | 缩写 | 核心思想 |
|------|------|---------|
| 近端策略优化 | PPO | 裁剪概率比防止策略突变，RLHF 的标准 RL 引擎 |
| 广义优势估计 | GAE | 指数加权平均 TD 误差，平衡偏差和方差 |
| 直接偏好优化 | DPO | 策略即奖励模型，一个分类损失替代 RLHF 三步流程 |
| 群组相对策略优化 | GRPO | 群组归一化奖励替代 Critic，DeepSeek-R1 的核心算法 |
| 软自适应策略优化 | SAPO | 温度控制平滑门函数替代硬裁剪（阿里 2025） |
| 基于价值增强 PPO | VAPO | 重新引入 Critic，长度感知价值归一化（字节 2025） |
| 解耦裁剪动态采样 | DAPO | 非对称裁剪 + 动态过滤（字节 2025） |

## 对齐方法

| 术语 | 解释 |
|------|------|
| RLHF | 基于人类反馈的强化学习：SFT → RM → PPO 三步流程 |
| RLVR | 基于可验证奖励的强化学习：用规则/编译器替代 RM 打分 |
| RLAIF | 基于 AI 反馈的强化学习：用 LLM 替代人类提供偏好判断 |
| Constitutional AI | 宪法 AI：用书面原则文档替代人类标注（Anthropic） |
| 对齐税 | 让模型更安全时在合法任务上的性能下降 |
| 奖励欺骗 | 模型学会输出 RM 喜欢但人类不喜欢的内容 |

## LLM 训练

| 术语 | 解释 |
|------|------|
| 预训练 Pre-training | 在海量文本上做下一个 token 预测，学习语言和世界知识 |
| SFT | 监督微调——在高质量问答对上学习对话格式和指令遵循 |
| 后训练 Post-training | SFT + RLHF/DPO/GRPO 的总称 |
| Mixture of Experts | MoE——部分参数激活的稀疏架构（如 Qwen3-MoE） |
| LoRA | 低秩适配——仅训练少量参数的高效微调方法 |
| Distillation 蒸馏 | 大模型的知识（如推理链）迁移给小模型 |

## Agent 系统

| 术语 | 解释 |
|------|------|
| Agent | 能自主感知环境、做出决策、执行动作的 AI 系统 |
| MCP | Model Context Protocol——AI 连接外部系统的开放标准（"USB-C for AI"） |
| Skill | 可复用的领域知识包（SKILL.md），跨 Claude Code/Codex/Opencode |
| Subagent | 独立上下文窗口的专用工作进程，主子 Agent 间摘要通信 |
| Hook | 生命周期事件驱动的自动化脚本（PreToolUse/PostToolUse） |
| Plugin | Skills + Agents + Hooks 的版本化打包 |
| Tool Calling | 模型在生成中调用外部函数（API/搜索/代码执行） |
| 黑盒 Agent | 只暴露 I/O 接口、隐藏内部决策的 Agent。用于上下文隔离 |
| AGENTS.md | 项目级 Agent 指令文件，跨 Claude Code/Cursor/Codex |

## 协议与接口

| 术语 | 解释 |
|------|------|
| Chat Completions API | OpenAI 定义的对话接口标准（/v1/chat/completions） |
| System Prompt | 定义 Agent 角色和行为的初始指令 |
| Token | 模型处理的最小文本单元（≈0.75 英文单词） |
| Hallucination 幻觉 | 模型生成看似合理但事实错误的内容 |
| Temperature 温度 | 控制输出随机性（0=确定，1=随机） |

## 评价指标

| 术语 | 解释 |
|------|------|
| AIME | 美国数学邀请赛（AI 推理能力的标准基准） |
| SWE-bench | 真实 GitHub Issue 修复能力评测 |
| Pass@k | 生成 k 个答案中至少一个正确的概率 |
| HumanEval | 代码生成功能正确性评测 |
| τ²-bench | Agent 任务完成质量评测 |
| GSM8K | 小学数学应用题数据集 |
| MATH-500 | 竞赛级数学推理数据集 |

---

## 论文速查

| 论文 | arXiv |
|------|-------|
| InstructGPT (RLHF) | [2203.02155](https://arxiv.org/abs/2203.02155) |
| PPO | [1707.06347](https://arxiv.org/abs/1707.06347) |
| DPO | [2305.18290](https://arxiv.org/abs/2305.18290) |
| GRPO (DeepSeekMath) | [2402.03300](https://arxiv.org/abs/2402.03300) |
| DeepSeek-R1 | [2501.12948](https://arxiv.org/abs/2501.12948) |
| ARPO | [2507.19849](https://arxiv.org/abs/2507.19849) |
| DAPO | [2503.14476](https://arxiv.org/abs/2503.14476) |
| VAPO | [2504.05118](https://arxiv.org/abs/2504.05118) |
| SAPO | [2511.20347](https://arxiv.org/abs/2511.20347) |
| AReaL | [2505.24298](https://arxiv.org/abs/2505.24298) |
| VeRL (EuroSys 2025) | [2409.19256](https://arxiv.org/abs/2409.19256) |
| Constitutional AI | [2212.08073](https://arxiv.org/abs/2212.08073) |
