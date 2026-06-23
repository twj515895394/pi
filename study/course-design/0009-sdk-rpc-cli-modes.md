# 0009：SDK / RPC / CLI 模式

## 本课目标

理解 Pi 如何把同一套 Agent 能力暴露给 interactive、print、json、rpc、sdk 等不同使用方式。

## 源码范围

```text
packages/coding-agent/src/core/sdk.ts
packages/coding-agent/src/core/agent-session-runtime.ts
packages/coding-agent/src/main.ts
packages/coding-agent/src/modes/interactive/*
packages/coding-agent/src/modes/json.ts
packages/coding-agent/src/modes/rpc.ts
packages/coding-agent/src/modes/print.ts
packages/coding-agent/docs/sdk.md
packages/coding-agent/docs/rpc.md
packages/coding-agent/docs/json.md
```

## 核心问题

- 模式层和 AgentSession 的边界是什么？
- SDK 适合什么集成场景？
- RPC 适合什么集成场景？
- 如果 Java 平台要接入 Pi 类能力，应该选择哪种方式？

## 小练习

设计一个 Java 后端调用 Pi RPC / SDK 能力的接入方案。

## 复盘产出

```text
study/reference/sdk-rpc-cli-modes.md
```
