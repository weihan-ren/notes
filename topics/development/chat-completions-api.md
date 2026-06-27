---
title: "Chat Completions API：大模型对话接口标准"
created: 2026-06-26
updated: 2026-06-26
has_toc: true
tags: [Chat-Completions, OpenAI, API, Agent, LLM]
category: development
---

# Chat Completions API

## 摘要

Chat Completions API 是 OpenAI 定义、已被全行业采用的大模型对话接口标准。它将 LLM 交互建模为**消息列表**（而非纯文本续写），支持 system/user/assistant/tool 四种角色，是 Claude Code、Codex、Opencode 等所有 Agent CLI 工具的底层通信协议。

---

## 1. 与传统 Completions 的区别

```
传统 Completions API (2020):
  输入: "法国的首都是"
  输出: "巴黎。"
  → 纯文本续写，无上下文结构

Chat Completions API (2023):
  输入: [
    {"role": "system",   "content": "你是一个地理专家"},
    {"role": "user",     "content": "法国的首都是哪里？"}
  ]
  输出: {"role": "assistant", "content": "法国的首都是巴黎。"}
  → 结构化消息，有角色标识
```

---

## 2. 四种消息角色

| role | 谁发送 | 作用 |
|------|--------|------|
| `system` | 开发者 | 定义模型行为、角色、输出格式、安全规则 |
| `user` | 用户 | 提问、指令、对话输入 |
| `assistant` | 模型 | 回答文本、工具调用请求 |
| `tool` | 工具执行结果 | 返回工具调用的结果 |

```
一次完整对话的消息流：

system:  "你是编程助手，用中文回答，代码加注释"
user:    "写一个快排函数"
assistant: "这是快速排序实现..."           ← 文本回答
user:    "改成降序"
assistant: 调用工具 → read_file("sort.py")  ← 工具调用
tool:    "文件内容: def quicksort..."      ← 工具返回
assistant: "已将排序改为降序..."           ← 最终回答
```

---

## 3. API 调用示例

### 基础调用

```
POST https://api.openai.com/v1/chat/completions

{
  "model": "gpt-4o",
  "messages": [
    {"role": "system", "content": "回复简洁，不超过20字"},
    {"role": "user", "content": "什么是黑洞？"}
  ],
  "temperature": 0.7,
  "max_tokens": 100
}

响应:
{
  "id": "chatcmpl-xxx",
  "object": "chat.completion",
  "choices": [{
    "index": 0,
    "message": {
      "role": "assistant",
      "content": "引力极强、光也无法逃逸的天体。"
    },
    "finish_reason": "stop"
  }],
  "usage": {
    "prompt_tokens": 25,
    "completion_tokens": 12,
    "total_tokens": 37
  }
}
```

### 流式调用

```
{ ..., "stream": true }

响应（逐 token）:
data: {"choices":[{"delta":{"content":"引"}}]}
data: {"choices":[{"delta":{"content":"力"}}]}
data: {"choices":[{"delta":{"content":"极"}}]}
...
data: [DONE]
```

### 工具调用

```
{
  "messages": [{"role": "user", "content": "北京天气？"}],
  "tools": [{
    "type": "function",
    "function": {
      "name": "get_weather",
      "description": "获取指定城市天气",
      "parameters": {
        "type": "object",
        "properties": {
          "city": {"type": "string"}
        }
      }
    }
  }]
}

响应:
"message": {
  "role": "assistant",
  "tool_calls": [{
    "function": {
      "name": "get_weather",
      "arguments": "{\"city\": \"北京\"}"
    }
  }]
}
```

---

## 4. 与 Agent 系统的关系

Chat Completions API 是所有 Agent CLI 工具的**底层协议**：

```
Agent 主循环：
  ┌──────────────────────────────────────┐
  │  1. 构造 messages (system + history)  │
  │  2. POST /chat/completions           │
  │  3. 解析响应 → 文本回答或工具调用     │
  │  4. 执行工具 → 结果写入 messages     │
  │  5. 回到步骤 2                       │
  └──────────────────────────────────────┘
```

| Agent 系统 | 底层 API |
|-----------|---------|
| Claude Code | Anthropic Messages API（同概念，不同路径） |
| Codex | OpenAI Chat Completions |
| Opencode | 多模型适配（Claude/GPT/DeepSeek API） |

---

## 5. 关键参数

| 参数 | 默认 | 作用 |
|------|------|------|
| `model` | 必填 | 模型 ID（gpt-4o, claude-sonnet-4-20250514 等） |
| `messages` | 必填 | 消息数组，多轮对话的核心 |
| `temperature` | 1.0 | 0=确定，1=随机；推理任务用 0 |
| `max_tokens` | 无限 | 限制输出长度 |
| `stream` | false | true=逐 token 流式返回 |
| `tools` | [] | 定义可调用的函数 |
| `response_format` | text | json_object / json_schema（结构化输出） |

---

## 6. 行业兼容性

| 厂商 | 端点 | 兼容度 |
|------|------|:-----:|
| OpenAI | `/v1/chat/completions` | 标准 |
| Anthropic | `/v1/messages` | 相似但消息结构不同 |
| Google Gemini | `/v1beta/models/...:generateContent` | 需适配 |
| DeepSeek | `/v1/chat/completions` | 完全兼容 |
| Qwen (DashScope) | `/compatible-mode/v1/chat/completions` | 完全兼容 |
| Groq / Together / Fireworks | `/v1/chat/completions` | 完全兼容 |

---

## 更新记录

- 2026-06-26：初始创建
