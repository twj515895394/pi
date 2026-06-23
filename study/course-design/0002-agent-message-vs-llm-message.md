# 0002：AgentMessage vs LLM Message

## 本课目标

理解 Pi 为什么要区分内部消息模型和 LLM 消息模型，并掌握 `convertToLlm` 的边界意义。

## 源码范围

```text
packages/agent/src/types.ts
packages/agent/src/agent.ts
packages/agent/src/agent-loop.ts
packages/agent/src/harness/messages.ts
packages/coding-agent/src/core/messages.ts
```

## 核心问题

- 为什么 Agent 内部消息不能全部直接发给模型？
- UI 消息、custom message、branch summary、compaction summary 是否应该进入 LLM？
- `transformContext` 和 `convertToLlm` 的职责边界是什么？
- 企业 Agent 中审计事件、业务状态、内部提示应该如何处理？

## 小练习

画出一张消息模型对照表：

```text
AgentMessage 类型
  -> 是否写入 session
  -> 是否显示给用户
  -> 是否进入 LLM
  -> 是否影响工具调用
```

## 复盘产出

```text
study/reference/agent-message-vs-llm-message.md
```
