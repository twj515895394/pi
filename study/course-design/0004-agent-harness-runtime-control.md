# 0004：AgentHarness 是 Runtime 控制层

## 本课目标

理解 Pi 中 `AgentHarness` 为什么不是简单封装 `Agent`，而是把模型、工具、消息、权限、事件、上下文、Session、Compaction、Branch Summary 和 UI/CLI/SDK 连接起来的 Runtime 控制层。

## 源码范围

```text
packages/agent/src/index.ts
packages/agent/src/harness/agent-harness.ts
packages/agent/src/harness/types.ts
packages/agent/src/harness/messages.ts
packages/agent/src/harness/session/session.ts
packages/agent/src/harness/compaction/compaction.ts
packages/agent/src/harness/compaction/branch-summarization.ts
```

## 核心问题

- `AgentLoop`、`Agent`、`AgentHarness` 三者的职责边界是什么？
- 为什么 Session 写入、Compaction、Branch Summary 不应该放进 `AgentLoop`？
- `createTurnState()` 如何组装一次 turn 的运行快照？
- `createLoopConfig()` 如何把 Harness 的 Runtime 能力注入 AgentLoop？
- Harness 如何承载企业权限、审计、Provider 请求治理和 UI/CLI/SDK 事件？

## 小练习

基于源码画出 Runtime 分层图，并标出：

```text
User / UI / CLI / SDK
AgentHarness
AgentLoop
LLM Provider
Tools
Session
Hooks
```

## 复盘产出

```text
study/reference/agent-harness-runtime-control.md
```

## 教学方式

本课属于高概念密度课程，应使用：

- 分层表格
- Runtime 总览图
- 职责边界图
- 源码锚点
- 企业 Agent 设计类比
- 简化理解检查
