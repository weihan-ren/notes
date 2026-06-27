---
title: 概念字典
nav_order: 99
has_toc: true
---

# 概念字典

快速查阅 AI/ML/Agent 领域核心概念。点击 🔍 搜索知识库中的相关内容。

<script>
function searchKB(term) {
  var input = document.getElementById('search-input');
  if (input) {
    input.value = term;
    input.focus();
    input.dispatchEvent(new Event('input', {bubbles: true}));
    input.dispatchEvent(new Event('focus', {bubbles: true}));
  }
}
</script>

## 强化学习 (RL)

| 术语 | 英文 | 解释 | 搜索 |
|------|------|------|:--:|
| 策略 | Policy (π) | 给定状态，选择动作的决策规则。LLM 本身就是一个策略 | [🔍](javascript:searchKB('策略 Policy')) |
| 轨迹 | Trajectory | S₀→A₀→R₁→S₁→...→S_T，Agent 与环境完整交互路径 | [🔍](javascript:searchKB('轨迹 Trajectory')) |
| 步 | Step | 轨迹中的单次决策。Token 级 RL 中是生成一个 token | [🔍](javascript:searchKB('步 Step 信用分配')) |
| 群组 | Group | 同一 prompt 下的多条轨迹集合（GRPO 中 G=16+） | [🔍](javascript:searchKB('群组 Group GRPO')) |
| 奖励 | Reward | 环境对动作的标量反馈（好=正,差=负） | [🔍](javascript:searchKB('奖励 Reward RLVR')) |
| 优势 | Advantage A(s,a) | "这个动作比平均好/差多少"，RL 核心训练信号 | [🔍](javascript:searchKB('优势 Advantage GAE')) |
| 折扣因子 | Discount Factor γ | 未来奖励的权重衰减（0.99=看重长期） | [🔍](javascript:searchKB('折扣因子')) |
| On-Policy | On-Policy | 用当前策略采样的数据训练，更新后旧数据作废 | [🔍](javascript:searchKB('On-Policy PPO')) |
| Off-Policy | Off-Policy | 用历史策略数据训练，样本效率高但不稳定 | [🔍](javascript:searchKB('Off-Policy DPO')) |
| Actor-Critic | Actor-Critic | Actor 负责决策，Critic 负责评估 | [🔍](javascript:searchKB('Actor-Critic PPO')) |
| 信用分配 | Credit Assignment | 把终端奖励分配到中间步骤 | [🔍](javascript:searchKB('信用分配 Credit Assignment')) |

## RL 算法

| 术语 | 缩写 | 核心思想 | 搜索 |
|------|------|---------|:--:|
| 近端策略优化 | PPO | 裁剪概率比防止策略突变，RLHF 的标准 RL 引擎 | [🔍](javascript:searchKB('PPO 近端策略优化')) |
| 广义优势估计 | GAE | 指数加权平均 TD 误差，平衡偏差和方差 | [🔍](javascript:searchKB('GAE 广义优势估计')) |
| 直接偏好优化 | DPO | 策略即奖励模型，一个分类损失替代 RLHF | [🔍](javascript:searchKB('DPO 直接偏好优化')) |
| 群组相对策略优化 | GRPO | 群组归一化奖励替代 Critic，DeepSeek-R1 核心 | [🔍](javascript:searchKB('GRPO 群组相对策略优化')) |
| 软自适应策略优化 | SAPO | 温度控制平滑门函数替代硬裁剪（阿里） | [🔍](javascript:searchKB('SAPO 软自适应')) |
| 基于价值增强 PPO | VAPO | 重新引入 Critic + 长度感知归一化（字节） | [🔍](javascript:searchKB('VAPO Value-based')) |
| 解耦裁剪动态采样 | DAPO | 非对称裁剪 + 动态过滤（字节） | [🔍](javascript:searchKB('DAPO 解耦裁剪')) |

## 对齐方法

| 术语 | 解释 | 搜索 |
|------|------|:--:|
| RLHF | 基于人类反馈的强化学习：SFT → RM → PPO | [🔍](javascript:searchKB('RLHF')) |
| RLVR | 基于可验证奖励的强化学习 | [🔍](javascript:searchKB('RLVR 可验证奖励')) |
| RLAIF | 基于 AI 反馈的强化学习 | [🔍](javascript:searchKB('RLAIF Constitutional AI')) |
| Constitutional AI | 用书面原则文档替代人类标注（Anthropic） | [🔍](javascript:searchKB('Constitutional AI 宪法')) |
| 对齐税 | 让模型更安全时在合法任务上的性能下降 | [🔍](javascript:searchKB('对齐税')) |
| 奖励欺骗 | 模型学会输出 RM 喜欢但人类不喜欢的内容 | [🔍](javascript:searchKB('奖励欺骗 reward hacking')) |

## LLM 训练

| 术语 | 解释 | 搜索 |
|------|------|:--:|
| 预训练 | 在海量文本上做 next-token 预测 | [🔍](javascript:searchKB('预训练 Pre-training')) |
| SFT | 监督微调——学习对话格式和指令遵循 | [🔍](javascript:searchKB('SFT 监督微调')) |
| 后训练 | SFT + RLHF/DPO/GRPO 的总称 | [🔍](javascript:searchKB('后训练 Post-training')) |
| MoE | Mixture of Experts——稀疏激活架构 | [🔍](javascript:searchKB('MoE Mixture of Experts')) |
| LoRA | 低秩适配——仅训练少量参数 | [🔍](javascript:searchKB('LoRA 低秩适配')) |
| 蒸馏 | 大模型知识迁移给小模型 | [🔍](javascript:searchKB('蒸馏 Distillation')) |

## Agent 系统

| 术语 | 解释 | 搜索 |
|------|------|:--:|
| Agent | 自主感知环境、决策、执行的 AI 系统 | [🔍](javascript:searchKB('Agent')) |
| MCP | Model Context Protocol——"USB-C for AI" | [🔍](javascript:searchKB('MCP Model Context Protocol')) |
| Skill | 可复用领域知识包（SKILL.md） | [🔍](javascript:searchKB('Skill SKILL.md')) |
| Subagent | 独立上下文窗口的专用工作进程 | [🔍](javascript:searchKB('Subagent 子Agent')) |
| Hook | 生命周期事件驱动的自动化脚本 | [🔍](javascript:searchKB('Hook PreToolUse')) |
| Plugin | Skills + Agents + Hooks 的版本化打包 | [🔍](javascript:searchKB('Plugin 插件')) |
| Tool Calling | 模型生成中调用外部函数 | [🔍](javascript:searchKB('Tool Calling 工具调用')) |
| 黑盒 Agent | 只暴露 I/O 接口的 Agent | [🔍](javascript:searchKB('黑盒 Agent')) |
| AGENTS.md | 项目级 Agent 指令文件 | [🔍](javascript:searchKB('AGENTS.md')) |

## 协议与接口

| 术语 | 解释 | 搜索 |
|------|------|:--:|
| Chat Completions API | OpenAI 对话接口标准 | [🔍](javascript:searchKB('Chat Completions API')) |
| System Prompt | 定义 Agent 角色和行为的初始指令 | [🔍](javascript:searchKB('System Prompt')) |
| Token | 模型处理的最小文本单元 | [🔍](javascript:searchKB('Token')) |
| Hallucination | 模型生成看似合理但事实错误的内容 | [🔍](javascript:searchKB('幻觉 Hallucination')) |
| Temperature | 控制输出随机性（0=确定,1=随机） | [🔍](javascript:searchKB('Temperature 温度')) |


## 深度学习框架

LLM 训练和推理的底层通用框架（区别于专门的训练分布式方案和推理引擎）。

| 框架 | 厂商 | 核心技术 | 芯片适配 |
|------|------|---------|---------|
| **PyTorch** | Meta | 动态图，社区最大 | NVIDIA CUDA 原生 |
| **MindSpore** | 华为 | 源码转换自动微分，自动并行 | **Ascend NPU 原生** |
| **JAX** | Google | 函数式+XLA 编译 | TPU 原生 |
| **PaddlePaddle** | 百度 | 动静统一，产业模型库 | 昆仑芯原生 |
| **TensorFlow** | Google | 静态图，TF Serving | TPU/GPU/CPU |

> 层次关系：PyTorch/MindSpore/JAX（DL 框架）→ DeepSpeed/FSDP/Megatron（训练分布式）→ vLLM/SGLang（推理引擎）

## 推理后端

| 引擎 | 开发者 | 核心技术 | 特点 | 搜索 |
|------|--------|---------|------|:--:|
| **vLLM** | UC Berkeley | PagedAttention | 社区标准，Prefix Caching 对 GRPO 关键 | [🔍](javascript:searchKB('vLLM PagedAttention')) |
| **SGLang** | Stanford | RadixAttention | DP Attention，多轮 Agent RL 优化 | [🔍](javascript:searchKB('SGLang RadixAttention')) |
| **TensorRT-LLM** | NVIDIA | TensorRT + FP8/INT4 | H100/B200 性能极致 | [🔍](javascript:searchKB('TensorRT-LLM')) |
| **TGI** | HuggingFace | 简单部署 | HF 生态集成 | [🔍](javascript:searchKB('TGI Text Generation Inference')) |
| **Ollama** | 社区 | llama.cpp 封装 | 本地一键部署 | [🔍](javascript:searchKB('Ollama')) |
| **llama.cpp** | ggerganov | GGUF 量化 | 消费级设备友好 | [🔍](javascript:searchKB('llama.cpp GGUF')) |

## 训练后端

| 后端 | 开发者 | 核心技术 | 最大模型 | 搜索 |
|------|--------|---------|:------:|:--:|
| **DeepSpeed ZeRO** | Microsoft | ZeRO 分片优化 | 70B+ | [🔍](javascript:searchKB('DeepSpeed ZeRO')) |
| **FSDP / FSDP2** | PyTorch | 分片数据并行 | 70B+ | [🔍](javascript:searchKB('FSDP FSDP2')) |
| **Megatron-LM** | NVIDIA | TP+PP+CP+EP | **万亿级** | [🔍](javascript:searchKB('Megatron-LM')) |
| **PyTorch DDP** | PyTorch | 数据并行 | 7B | [🔍](javascript:searchKB('PyTorch DDP')) |
| **PyTorch Archon** | 社区 | FSDP2+PP+EP | 100B+ | [🔍](javascript:searchKB('Archon')) |

## 评价指标

| 术语 | 解释 | 搜索 |
|------|------|:--:|
| AIME | 美国数学邀请赛（推理基准） | [🔍](javascript:searchKB('AIME')) |
| SWE-bench | GitHub Issue 修复评测 | [🔍](javascript:searchKB('SWE-bench')) |
| Pass@k | k 个答案中至少一个正确的概率 | [🔍](javascript:searchKB('Pass@k')) |
| GSM8K | 小学数学应用题数据集 | [🔍](javascript:searchKB('GSM8K')) |

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

---

> 点击任一概念的 🔍 即可搜索知识库中相关内容 · 更新：2026-06-26
