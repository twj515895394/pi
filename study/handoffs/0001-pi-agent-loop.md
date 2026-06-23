# Handoff：0001 Pi Agent Loop 源码精读

## 本节课状态

- 状态：已完成
- 日期：2026-06-23
- 对应课程：`study/course-design/0001-pi-agent-loop.md`
- 对应课件：`study/lessons/0001-pi-agent-loop.html`
- 对应参考资料：`study/reference/pi-agent-loop-sequence.md`

## 本节课目标

读懂一次用户输入如何进入 Pi 的 Agent Loop，并理解它如何变成模型响应、工具调用、工具结果回灌和下一轮模型调用。

## 实际学习内容

本节课完成了以下源码链路精读：

```text
Agent.prompt(input)
  -> normalizePromptInput(input)
  -> runPromptMessages(messages)
  -> createContextSnapshot()
  -> createLoopConfig()
  -> runAgentLoop(...)
  -> runLoop(...)
  -> streamAssistantResponse(...)
  -> executeToolCalls(...)
  -> createToolResultMessage(...)
```

重点讲解了：

- `Agent` 与 `AgentLoop` 的职责边界。
- `AgentMessage[]` 与 LLM `Message[]` 的区别。
- `currentContext.messages` 与 `newMessages` 的区别。
- `transformContext` 与 `convertToLlm` 的分工。
- tool call 执行链路。
- tool result 为什么要回灌为 `toolResult message`。
- AgentLoop 如何通过事件反向更新 Agent state。

## 已确认理解

用户已经能够用自己的话说明：

1. `Agent.prompt("hello")` 不能直接进入 loop，需要先统一成 `AgentMessage[]`。
2. `currentContext.messages` 是当前 run 的完整上下文基础；`newMessages` 是本次 run 新产生的消息增量。
3. 工具结果需要作为 `toolResult message` 回灌到上下文，让模型继续判断是否继续 tool call 或生成最终回答。
4. `transformContext` 用于上下文压缩、裁剪、注入；`convertToLlm` 用于转成 LLM 需要的格式。
5. `convertToLlm` 必须在每次 LLM 调用前执行，因为 Agent Loop 过程中 messages 会持续变化。
6. `Agent` 统一管理状态更新，`AgentLoop` 通过事件通知外层，避免底层循环直接修改 Agent state。

## 已纠正的问题

本节课纠正了两个小误区：

1. `uiStatus` / notification / UI-only status 这类消息通常应在 `convertToLlm` 阶段过滤，而不是在 `transformContext` 阶段处理。
2. 工具执行事件名应使用 `tool_execution_start`、`tool_execution_update`、`tool_execution_end`，而不是泛称 `toolcallupdate/toolcallend`。

## 源码阅读进度

已阅读：

```text
packages/agent/src/agent.ts
packages/agent/src/agent-loop.ts
packages/agent/src/types.ts
```

关键源码点：

```text
Agent.prompt()
normalizePromptInput()
runPromptMessages()
createContextSnapshot()
createLoopConfig()
runAgentLoop()
runLoop()
streamAssistantResponse()
executeToolCalls()
prepareToolCall()
executePreparedToolCall()
finalizeExecutedToolCall()
createToolResultMessage()
processEvents()
```

## 练习进度

本节课完成了口头理解检查。

时序图练习已通过 `study/reference/pi-agent-loop-sequence.md` 的 ASCII 图沉淀。

## 文档更新情况

已更新：

```text
study/reference/pi-agent-loop-sequence.md
study/lessons/0001-pi-agent-loop.html
study/PROGRESS.md
study/CURRENT.md
```

已新增：

```text
study/handoffs/0001-pi-agent-loop.md
study/learning-records/0001-agent-loop-is-model-tool-observation-cycle.md
```

## 下节课计划

下一节课进入：

```text
0002：AgentMessage vs LLM Message
```

下节课要解决的问题：

```text
为什么 Pi 要在内部保留 AgentMessage，而不是直接使用 LLM Message？
```

建议源码范围：

```text
packages/agent/src/types.ts
packages/agent/src/agent.ts
packages/agent/src/agent-loop.ts
packages/agent/src/harness/messages.ts
```

## 新会话续接提示

如果新开会话，请说：

```text
继续 Pi Agent Harness 学习。请先读取 study/README.md、study/CURRENT.md、study/PROGRESS.md 和 study/handoffs/0001-pi-agent-loop.md，然后从第二课 AgentMessage vs LLM Message 开始。
```
