# 当前学习状态

本文件是新会话续接学习时的第一入口。任何 Agent 进入 `study/` 工作区后，应先阅读本文件，再阅读最新 handoff。

## 当前课程

`0002`：AgentMessage vs LLM Message

## 当前状态

进行中。

第一课 `0001：Pi Agent Loop 源码精读` 已完成。

## 最新 Handoff

```text
study/handoffs/0001-pi-agent-loop.md
```

## 已确认理解

用户已经理解：

```text
外部输入 -> AgentMessage[] -> Agent Loop -> LLM Message[] -> assistant/toolCall -> toolResult message -> 下一轮 LLM
```

用户已能解释：

- `normalizePromptInput` 为什么存在。
- `currentContext.messages` 与 `newMessages` 的区别。
- tool result 为什么要回灌成 `toolResult message`。
- `transformContext` 与 `convertToLlm` 的区别。
- AgentLoop 为什么通过事件更新 Agent state。

## 已纠正误区

- UI-only status 通常应在 `convertToLlm` 阶段过滤。
- 工具执行事件名是 `tool_execution_start/update/end`。

## 当前问题

第二课要解决的问题：

```text
为什么 Pi 内部要保留 AgentMessage，而不是直接使用 LLM Message？
```

## 当前源码范围

```text
study/course-design/0002-agent-message-vs-llm-message.md
packages/agent/src/types.ts
packages/agent/src/agent.ts
packages/agent/src/agent-loop.ts
packages/agent/src/harness/messages.ts
```

## 新会话续接方式

新会话中请先阅读：

1. `study/README.md`
2. `study/MISSION.md`
3. `study/CURRENT.md`
4. `study/PROGRESS.md`
5. `study/handoffs/0001-pi-agent-loop.md`

然后继续第二课。