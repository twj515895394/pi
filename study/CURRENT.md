# 当前学习状态

本文件是新会话续接学习时的第一入口。任何 Agent 进入 `study/` 工作区后，应先阅读本文件，再阅读最新 handoff。

## 当前课程

`0004`：AgentHarness 是 Runtime 控制层

## 当前状态

进行中。

第一课 `0001：Pi Agent Loop 源码精读` 已完成。
第二课 `0002：AgentMessage vs LLM Message` 已完成。
第三课 `0003：Tool Call 执行链路` 已完成。

## 最新 Handoff

```text
study/handoffs/0003-tool-call-execution.md
```

## 已确认理解

用户已经理解：

```text
LLM 只能提出 toolCall；
Agent Runtime 才真正决定工具能不能执行、如何执行、结果如何治理和回灌。
```

用户已能解释：

- 为什么 toolCall 不能直接执行 `tool.execute()`。
- `prepareArguments` 与 `validateToolArguments` 的区别。
- `beforeToolCall` block 后为什么仍要生成 error toolResult。
- 并行执行时事件完成顺序和 toolResult message 顺序的区别。
- `beforeToolCall` 与 `afterToolCall` 的职责边界。
- 企业级 Tool Permission Layer 的最小检查项。

## 当前问题

第四课要解决的问题：

```text
AgentHarness 为什么不是简单封装 Agent，而是把模型、工具、消息、权限、事件、上下文和 UI/CLI/SDK 连接起来的 Runtime 控制层？
```

## 当前源码范围

```text
study/course-design/0004-agent-harness-runtime-control.md
packages/agent/src/harness/*
packages/agent/src/agent.ts
packages/agent/src/types.ts
packages/coding-agent/src/core/*
```

## 新会话续接方式

新会话中请先阅读：

1. `study/README.md`
2. `study/MISSION.md`
3. `study/CURRENT.md`
4. `study/PROGRESS.md`
5. `study/handoffs/0003-tool-call-execution.md`

然后继续第四课。