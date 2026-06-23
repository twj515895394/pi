# 0004：AgentHarness 是 Runtime 控制层

## 本课目标

理解 AgentHarness 为什么不是 AgentLoop 的重复封装，而是负责运行时控制、资源编排、事件扩展和会话治理的核心层。

## 源码范围

```text
packages/agent/src/harness/agent-harness.ts
packages/agent/src/harness/types.ts
packages/agent/src/harness/messages.ts
packages/agent/src/harness/system-prompt.ts
```

## 核心问题

- `AgentHarness` 持有哪些状态？
- `createTurnState()` 为什么每轮都要重建状态？
- `createLoopConfig()` 如何把 Harness hook 接入 AgentLoop？
- `pendingSessionWrites` 解决什么问题？
- `phase` 为什么需要区分 idle / turn / compaction / branch_summary？

## 小练习

画出 AgentLoop、Agent、AgentHarness、CodingAgent 四层职责图。

## 复盘产出

```text
study/reference/agent-harness-responsibility-map.md
```
