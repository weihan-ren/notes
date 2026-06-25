---
title: "Tree-GRPO 与基于树搜索的策略优化"
created: 2026-06-19
updated: 2026-06-19
sources:
  - https://arxiv.org/abs/2606.08346
  - https://arxiv.org/abs/2605.28109
  - https://arxiv.org/abs/2605.05262
  - https://arxiv.org/abs/2603.02216
  - https://arxiv.org/abs/2603.09551
has_toc: true
tags: [Tree-GRPO, CATPO, IB-TPO, InfoTree, ATPO, Tree-Search, RL, LLM, PostTraining]
category: ai
---

# Tree-GRPO 与基于树搜索的策略优化

## 摘要

2026 年 RL 后训练的最大趋势之一：从**扁平轨迹采样**转向**树结构搜索**。Tree-GRPO 将传统 GRPO 的 G 条独立轨迹扩展为在中间步骤分叉的树结构，从而获取密集的步级奖励信号。CATPO、IB-TPO、InfoTree、ATPO 等变体分别从树质量筛选、信息瓶颈、子模块优化、自适应预算等角度推进了这一范式。

---

## 1. 从 Flat GRPO 到 Tree Search

### 1.1 Flat GRPO 的局限

```
传统 GRPO（扁平采样）:
  Prompt → [轨迹A: 完整生成] [轨迹B: 完整生成] ... [轨迹G]
       → 群组归一化奖励 → 更新

问题：
  - 所有轨迹从头生成到尾，中间步骤没有信号
  - G 条轨迹中可能全是成功/全是失败 → 零梯度
  - 无法定位 "具体哪一步出错了"
```

### 1.2 Tree Search 的核心思想

```
Tree-GRPO（树结构采样）:
                    Prompt
                      │
              ┌───────┼───────┐
            步骤1a  步骤1b  步骤1c    ← 在中间步骤分叉
              │       │       │
          ┌───┴───┐   │   ┌───┴───┐
        步骤2a  步骤2b  │ 步骤2c  步骤2d
          │       │   │   │       │
         ...     ...  ... ...     ...

关键优势：
  - 分支点自然形成 "步级比较" → 省掉 PRM
  - 更高效利用 token 预算（共享前缀）
  - 树中既有成功也有失败的分支 → 稳定梯度
```

---

## 2. 主要变体

### 2.1 Tree-GRPO / TreeRPO（基线）

**核心思路**：在生成过程中的中间步骤进行分叉

- 对每个 prompt，在某中间步骤生成 k 个后续分支
- 不同分支共享前缀 token（KV Cache 复用）
- 同一分支点下的子节点天然形成群组比较
- **优势**：获得步级信号，无需单独训练 PRM（Process Reward Model）
- **局限**：并非所有树都有信息量——全成功/全失败的树贡献零梯度

### 2.2 CATPO（Critique-Augmented Tree Policy Optimization）

**论文**：arXiv:2606.08346（2026.06）

**核心创新**：诊断和修复 "无效树" 问题

```
CATPO 三步:

1. 树信息量评分 F(T)
   = 叶子结果多样性 × 策略-奖励去相关性
   → 识别哪些树值得训练、哪些在浪费计算

2. 死树救治（Critique-Guided Healing）
   对于全分支失败的树:
   → 定位最浅的失败点
   → 生成自然语言批评 → 嫁接修正后的续写
   → 恢复训练信号

3. 信息量加权损失
   → 按 F(T) 缩放每棵树的梯度贡献
   → 保留有效树的主导地位，弱化无效树
```

**结果**：Qwen2.5-Math-1.5B，MATH 数据集，比 TreeRPO 高 1.9%，比 GRPO 高 4.8%

### 2.3 IB-TPO（Information Bottleneck Tree Policy Optimization）

**论文**：arXiv:2605.28109（ICML 2026）
**机构**：阿里

**核心创新**：用信息瓶颈理论解决探索-利用失衡

```
IB-Score 指标:
  衡量 "步级推理多样性" vs "与正确答案的互信息"

  → GRPO + 常见正则化器无法在训练中保持平衡
  → IB-TPO 将 IB-Score 作为优化目标
  → IB-guided 树采样策略：相同 token 预算下多 50% 轨迹
  → 树结构被复用为 IB-Score 的蒙特卡洛估计
```

**结果**：比 GRPO 高 2.9%-3.6%，超过其他 SOTA 在线 RL 方法。代码：[alibaba/EfficientRL](https://github.com/alibaba/EfficientRL)

### 2.4 InfoTree

**论文**：arXiv:2605.05262（2026.05）

**核心创新**：子模块优化视角 + UUCB 选择器

```
InfoTree 框架:

1. UUCB (Uncertainty-aware Upper Confidence Bound)
   → 将中间状态选择建模为单调子模块最大化
   → 贪心一步选择器享有 1-1/e 近似保证
   → token 级熵 bonus 从经验技巧变为分析结论

2. ABA (Adaptive Budget Allocator)
   → 挽救初始树浪费在统一结果上的 prompt
   → 混合结果比例从 58.1% → 76.3%（<5% 开销）

3. Speculative Expansion
   → 异步预测展开，容忍有界陈旧
   → 墙钟开销从 14.3% → 4.8%
```

**结果**：9 个基准（数学推理 + web搜索 agent + 工具编程 agent），全面超越 flat GRPO、DeepSearch、Tree-GRPO、AT2PO、CW-GRPO、RC-GRPO

### 2.5 ATPO（Adaptive Tree Policy Optimization）

**论文**：arXiv:2603.02216（ICLR 2026）

**核心创新**：自适应预算分配 + 不确定性驱动探索

```
ATPO 将多轮对话建模为层级 MDP:

  → 不确定性量化：Bellman 误差 + 动作值方差
  → 高不确定性状态获得更多 rollout 预算
  → 不确定性引导剪枝 → 最小化开销
  → 异步搜索 + KV Cache 复用 → 最大化吞吐
```

**结果**：Qwen3-8B 超过 GPT-4o（+0.92% 准确率）

### 2.6 GeoSolver（Process-Aware Tree-GRPO）

**论文**：arXiv:2603.09551（2026.05 更新）

**领域**：遥感 VLM 推理

- 构建 Geo-PRM-2M（token 级过程监督数据集，熵引导 MCTS 合成）
- 训练 GeoPRM（token 级过程奖励模型）
- **Process-Aware Tree-GRPO**：树结构探索 + 忠实度加权奖励

---

## 3. 变体全景对比

| 方法 | 论文 | 会议 | 核心创新 | vs GRPO |
|------|------|:--:|---------|:-------:|
| **Tree-GRPO/TreeRPO** | 基线 | - | 中间步分叉，KV Cache 共享前缀 | 基线 |
| **CATPO** | 2606.08346 | - | 树信息量评分 + 死树救治 | +4.8% |
| **IB-TPO** | 2605.28109 | ICML 2026 | 信息瓶颈 + IB-Score + 50% 更多轨迹 | +3.6% |
| **InfoTree** | 2605.05262 | - | UUCB + ABA + 子模块优化 | 全面超越 |
| **ATPO** | 2603.02216 | ICLR 2026 | 自适应预算 + 不确定性驱动 | 超 GPT-4o |
| **GeoSolver** | 2603.09551 | - | PRM + 忠实度加权 Tree-GRPO | - |

---

## 4. 为什么 Tree Search 在 2026 年爆发

| 原因 | 说明 |
|------|------|
| **GRPO 成熟后的自然演进** | 扁平 GRPO 已经到了天花板，步级信用分配是下一个瓶颈 |
| **KV Cache 复用** | 树结构天然共享前缀，降低 token 开销 |
| **PRM 的替代方案** | 训练独立 PRM 成本高，树分叉天然提供步级比较 |
| **异步架构成熟** | veRL/OpenRLHF 支持异步 rollout，树搜索的墙钟开销大幅降低 |
| **Agent 场景的需求** | 多轮工具调用需要更精细的信用分配，树搜索比 flat GRPO 更适合 |

---

## 5. 在 RL 策略全景中的位置

```
GRPO (2024, DeepSeek-R1)
  │  扁平 G 条轨迹，群组归一化
  │
  ├── ARPO (2025)
  │    熵驱动自适应 Rollout（步级探索）
  │
  ├── Tree-GRPO / TreeRPO (2025-2026)
  │    树结构采样，步级比较信号
  │    │
  │    ├── CATPO (2026.06)
  │    │    树质量筛选 + 死树救治
  │    │
  │    ├── IB-TPO (2026.05, ICML)
  │    │    信息瓶颈驱动，探索-利用平衡
  │    │
  │    ├── InfoTree (2026.05)
  │    │    子模块优化 + UUCB + 自适应预算
  │    │
  │    └── ATPO (2026.03, ICLR)
  │         自适应预算 + 不确定性驱动
  │
  └── 关联方法
       ├── RTMC (2026.04): Rollout Tree Monte Carlo 优势估计
       └── TD-Grokking (2026.06): 训练时分解零奖励问题为子树
```

---

## 更新记录

- 2026-06-19：初始创建，覆盖 Tree-GRPO/CATPO/IB-TPO/InfoTree/ATPO/GeoSolver
