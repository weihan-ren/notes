---
title: "ARPO（Agentic Reinforced Policy Optimization）详解"
has_toc: true
created: 2026-06-18
updated: 2026-06-18
sources:
  - https://arxiv.org/abs/2507.19849
  - https://arxiv.org/abs/2505.16282
  - https://arxiv.org/abs/2603.00526
  - https://arxiv.org/abs/2507.19849
  - https://arxiv.org/abs/2505.16282
  - https://arxiv.org/abs/2603.00526
  - https://arxiv.org/abs/2512.01228
  - https://arxiv.org/abs/2308.15550
  - https://github.com/dongguanting/ARPO
  - https://github.com/dvlab-research/ARPO
tags: [ARPO, AgentRL, GRPO, LLM, PostTraining, DPO, GUI-Agent]
category: ai
---

# ARPO（Agentic Reinforced Policy Optimization）详解

## 摘要

ARPO 是 2025 年下半年出现的 LLM 后训练新范式。名称"ARPO"在文献中有多种变体，但核心都围绕将强化学习应用于 **LLM Agent 的多轮交互训练**。最主要的两篇论文分别针对通用 Agent 工具调用和 GUI Agent 操作，核心贡献在于解决了从"单轮推理 RL"到"多轮 Agent RL"的关键瓶颈。

---

## 1. ARPO 的多种含义

在 2025-2026 年的文献中，"ARPO"至少有以下 6 种不同的全称：

| 全称 | 论文 | 时间 | 领域 | 与 LLM 相关性 |
|------|------|------|------|---------------|
| **Agentic Reinforced Policy Optimization** | arXiv:2507.19849 | 2025.07 | LLM Agent 训练 | ⭐⭐⭐⭐⭐ |
| **Agentic Replay Policy Optimization** | arXiv:2505.16282 | 2025.05 | GUI Agent 训练 | ⭐⭐⭐⭐⭐ |
| **Advantage-guided Ranking Preference Optimization** | arXiv:2603.00526 | 2026.02 (CVPR 2026) | 3D Mesh 生成 | ⭐⭐⭐ |
| Adversarially Robust Policy Optimization | arXiv:2512.01228 | 2025.12 | 鲁棒 RL | ⭐⭐ |
| Adversarial Robust Policy Optimization | arXiv:2308.15550 | 2023.08 | 视觉 RL 泛化 | ⭐⭐ |
| Alternating Resolution and Power Optimization | arXiv:2510.10028 | 2025.10 | UAV 网络优化 | ⭐ |

> 本文重点分析前两个与 LLM Agent 训练直接相关的 ARPO 变体。

---

## 2. ARPO 核心版：Agentic Reinforced Policy Optimization

**论文**：Dong et al. (2025.07), *"Agentic Reinforced Policy Optimization"*
**机构**：中国人民大学 + 快手 + 美团
**代码**：https://github.com/dongguanting/ARPO

### 2.1 要解决什么问题

现有的 RLVR（Reinforcement Learning with Verifiable Rewards，如 GRPO 训练 DeepSeek-R1）主要为 **单轮推理任务** 设计——模型拿到一个问题，生成推理链，判断答案是否正确。

但在真实场景中，LLM Agent 需要 **多轮交互**：调用工具 → 读取结果 → 继续推理 → 再调用工具 → ... 往复循环。

**核心挑战**：当前 RL 算法（如 GRPO）在轨迹级别（trajectory-level）进行优化时，无法很好地平衡模型的内在长程推理能力和多轮工具交互能力。

### 2.2 关键观察：工具调用后的"熵增"

ARPO 论文通过初步实验发现了一个重要现象：

> LLM 在与外部工具交互后，生成的 token 熵分布会显著增加——即模型在工具调用后表现出**高度不确定性**。

这一观察是 ARPO 设计的核心动机：
- 工具调用引入了"信息注入"——新的环境反馈改变了模型对任务的理解
- 模型需要"重新校准"以决定下一步该做什么
- 在熵高的步骤上，探索空间更大，需要有更丰富的采样

### 2.3 核心机制：熵驱动的自适应 Rollout

ARPO 的核心创新是一个**基于熵的自适应 Rollout 机制**：

**传统 GRPO**：对每个提示生成 G 个完整的轨迹，用群组内相对优势更新。

**ARPO**：
1. 监控每个生成步骤的 token 熵值
2. 在工具调用后的高熵步骤上，触发**步级别采样（step-level sampling）**，生成多个后续路径
3. 在低熵步骤上，使用**轨迹级别采样（trajectory-level sampling）**
4. 动态平衡全局轨迹探索和局部步骤探索

**效果**：
- 在工具调用后（高熵区域）提供更多探索
- 在稳定推理阶段（低熵区域）保持高效
- **仅使用一半的工具调用预算**就达到或超越现有方法

### 2.4 优势归因估计（Advantage Attribution Estimation）

除了自适应 Rollout，ARPO 还引入了优势归因估计：

- 将轨迹级别的优势信号归因到每一步的工具调用决策上
- 让 LLM 内化**步级别的优势差异**——理解"在此时调用这个工具到底贡献了多少"
- 解决了多轮 Agent RL 中信用分配（credit assignment）的关键难题

### 2.5 实验结果

在 13 个具有挑战性的基准测试上评估，覆盖：
- **计算推理**：数学、编程
- **知识推理**：问答、事实核查
- **深度搜索**：信息检索、多步查询

**主要结果**：
- ARPO 全面优于轨迹级别 RL 算法（如基础 GRPO）
- 仅需现有方法一半的工具调用预算即可达到更好的性能
- 在多轮 Agent 场景中收敛更稳定

---

## 3. ARPO 第二版：Agentic Replay Policy Optimization（GUI Agent）

**论文**：Lu et al. (2025.05), *"ARPO: End-to-End Policy Optimization for GUI Agents with Experience Replay"*
**机构**：dvlab-research（港中文/商汤）
**代码**：https://github.com/dvlab-research/ARPO.git

### 3.1 要解决什么问题

训练 LLM 作为 GUI Agent（控制图形界面完成复杂任务）面临独特挑战：
- **稀疏奖励**：完成整个任务才有反馈
- **延迟反馈**：一个操作的效果可能在很多步之后才显现
- **高 Rollout 成本**：每个训练样本需要完整的 GUI 交互
- **多模态输入**：需要理解屏幕截图 + 文本指令

### 3.2 核心机制：GRPO + 经验回放（Experience Replay）

ARPO 将标准 GRPO 与经验回放缓冲区结合：

1. **回放缓冲区**：存储历史上成功的交互轨迹
2. **跨迭代复用**：不只在当前 batch 采样，也从回放缓冲区中采样成功经验
3. **任务选择策略**：根据 baseline agent 的表现筛选任务——聚焦于"刚好在能力边缘"的任务

### 3.3 任务筛选策略

ARPO 提出了一种任务选择策略：

- 过滤掉已被"完美解决"的任务（太简单，学了也没用）
- 过滤掉"完全不会"的任务（太难，奖励信号稀疏）
- 保留"刚好能学到东西"的任务——处于能力边缘

**直觉**：就像人类学习，做已经会做的题没进步，做完全不会的题没方向，做"有点难但能做对一部分"的题进步最快。

### 3.4 实验结果

在 **OSWorld** 基准测试（真实操作系统 GUI 任务）上：
- ARPO 在 LLM-based GUI Agent 强化学习中建立了新的基线
- 与离线偏好优化方法（如 DPO）的比较显示了 policy-based 方法在 GUI 环境中的优势
- 验证了 RL（而非 SFT 或偏好对齐）是训练多轮 GUI Agent 的有效路径

---

## 4. ARPO 第三版：Advantage-guided Ranking Preference Optimization（3D 生成）

**论文**：Zhou et al. (2026.02), *"Mesh-Pro"*, CVPR 2026
**领域**：3D 四边形网格生成

### 4.1 核心贡献

虽然不直接与 LLM 相关，但这个 ARPO 变体在技术上有值得关注的创新：

- **异步在线 RL**：第一个用于 3D 网格生成的异步 RL 框架，比同步 RL 快 **3.75 倍**
- **ARPO 算法**：在训练效率和泛化性之间取得比 DPO 和 GRPO 更好的平衡
- **Mesh-Pro 系统**：结合 ARPO + 对角感知的三角-四边形混合 tokenization + 基于射线的几何完整性奖励
- 在艺术风格和密集网格上达到 SOTA

**值得关注**：其异步 RL 框架的设计思路（生成和训练解耦）可能对其他 RL 训练场景（包括 LLM Agent）有借鉴意义。

---

## 5. ARPO vs GRPO vs DPO 对比

| 维度 | DPO | GRPO | ARPO (Agentic) | ARPO (Replay) |
|------|-----|------|----------------|---------------|
| **发布时间** | 2023 | 2024 | 2025.07 | 2025.05 |
| **是否需要 RM** | 隐式 | 需要（规则/模型） | 需要 | 需要 |
| **是否需要 Critic** | N/A | 不需要 | 不需要 | 不需要 |
| **采样策略** | Off-policy（静态） | Trajectory-level On-policy | 混合（熵驱动） | On-policy + 回放 |
| **适用场景** | 单轮偏好对齐 | 单轮推理 RL | 多轮 Agent 工具调用 | 多轮 GUI 操作 |
| **核心创新** | 策略即 RM | 群组相对优势 | 熵驱动自适应 Rollout | GRPO + 经验回放 |
| **工具调用** | 不支持 | 不支持 | 核心场景 | 核心场景 |
| **训练效率** | 高（最简单） | 中（并行采样） | 高（减半工具调用预算） | 中（回放复用） |

---

## 6. 关键引用

> "LLMs tend to exhibit highly uncertain behavior, characterized by an increase in the entropy distribution of generated tokens, immediately following interactions with external tools."
> — Dong et al., Agentic Reinforced Policy Optimization (2025)

> "ARPO achieves improved performance using only half of the tool-use budget required by existing methods, offering a scalable solution for aligning LLM-based agents with real-time dynamic environments."
> — Dong et al., Agentic Reinforced Policy Optimization (2025)

> "We propose Agentic Replay Policy Optimization (ARPO), an end-to-end RL approach that augments Group Relative Policy Optimization (GRPO) with a replay buffer to reuse the successful experience across training iterations."
> — Lu et al., ARPO: End-to-End Policy Optimization for GUI Agents (2025)

---

## 7. 技术演进脉络

```
RLHF (2017-2022)
  └── DPO (2023): 跳过 RM 和 PPO，off-policy
       └── GRPO (2024): 回到 RL，但用群组优势省掉 Critic
            └── ARPO (2025): GRPO 的 Agent 版
                 ├── Agentic ARPO: 熵驱动自适应 Rollout + 步级优势归因
                 │   → 解决多轮工具调用的信用分配和探索平衡
                 └── Replay ARPO: GRPO + 经验回放 + 任务筛选
                     → 解决 GUI Agent 的稀疏奖励和高成本问题
```

**核心趋势**：从"让 LLM 学会说话更好"（RLHF/DPO）→ "让 LLM 学会推理"（GRPO/R1）→ "让 LLM 学会使用工具完成复杂任务"（ARPO）

---

## 更新记录

- 2026-06-18：初始创建，涵盖 ARPO 三种主要变体及其与 GRPO/DPO 的关系
