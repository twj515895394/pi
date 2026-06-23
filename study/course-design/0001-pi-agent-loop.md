# 0001：Pi Agent Loop 源码精读

## 本课目标

读懂一次用户输入如何进入 Pi 的 Agent Loop，并理解它如何变成模型响应、工具调用、工具结果回灌和下一轮模型调用。

本课结束后，应该能画出：

```text
Agent.prompt()
  -> runAgentLoop()
  -> runLoop()
  -> streamAssistantResponse()
  -> executeToolCalls()
  -> toolResult message
  -> next turn or agent_end
```

## 前置知识

需要知道：

- TypeScript 基本语法。
- async / await。
- EventStream / async iteration 的基本概念。
- LLM tool calling 的基本含义。

不要求提前理解完整 Harness、Extension、Session Tree。

## 源码范围

```text
packages/agent/src/agent.ts
packages/agent/src/agent-loop.ts
packages/agent/src/types.ts
packages/agent/README.md
```

## 核心概念

### Agent Loop

Agent Loop 是 Agent 的最小运行闭环：

```text
messages -> LLM -> assistant message -> tool calls -> tool results -> messages -> LLM
```

它解决的不是“模型怎么生成文字”，而是“模型和外部世界如何通过工具形成多轮闭环”。

### Agent

`Agent` 是带状态的封装。它持有：

```text
systemPrompt
model
thinkingLevel
tools
messages
isStreaming
streamingMessage
pendingToolCalls
errorMessage
```

它对外提供：

```text
prompt()
continue()
steer()
followUp()
subscribe()
abort()
waitForIdle()
```

### AgentLoop

`agentLoop()` 是更底层的循环函数。它不负责 UI，不负责完整产品体验，只负责执行模型调用、工具调用和事件流。

## 源码阅读路径

### 第一步：从 Agent.prompt() 开始

阅读：

```text
packages/agent/src/agent.ts
```

重点看：

```text
prompt()
normalizePromptInput()
runPromptMessages()
createContextSnapshot()
createLoopConfig()
```

问题：

- `prompt("hello")` 如何被包装成 user message？
- 为什么 `Agent` 要创建 context snapshot，而不是直接把内部 state 丢给 loop？
- `createLoopConfig()` 里把哪些能力传给了 AgentLoop？

### 第二步：进入 runAgentLoop()

阅读：

```text
packages/agent/src/agent-loop.ts
```

重点看：

```text
agentLoop()
runAgentLoop()
runLoop()
```

问题：

- `agent_start`、`turn_start`、`message_start`、`message_end` 是何时发出的？
- `newMessages` 和 `currentContext.messages` 有什么区别？
- 外层 while 和内层 while 分别解决什么？

### 第三步：模型响应阶段

重点看：

```text
streamAssistantResponse()
```

问题：

- `transformContext` 在什么时候执行？
- `convertToLlm` 为什么在 LLM 调用边界执行？
- assistant message 为什么要先 partial 后 final？
- `message_update` 事件解决什么问题？

### 第四步：工具执行阶段

重点看：

```text
executeToolCalls()
executeToolCallsSequential()
executeToolCallsParallel()
prepareToolCall()
executePreparedToolCall()
finalizeExecutedToolCall()
createToolResultMessage()
```

问题：

- tool call 是从 assistant message 的哪部分取出来的？
- 工具不存在时如何处理？
- 参数校验在哪里发生？
- `beforeToolCall` 能阻断什么？
- `afterToolCall` 能修改什么？
- 工具错误为什么被包装成 toolResult message？

### 第五步：下一轮或结束

重点看：

```text
prepareNextTurn
shouldStopAfterTurn
getSteeringMessages
getFollowUpMessages
agent_end
```

问题：

- 有 tool result 后为什么通常需要再问一次模型？
- steering 和 follow-up 的区别是什么？
- 什么情况下 loop 会提前结束？

## 检查问题

完成阅读后，尝试回答：

1. `Agent` 和 `agentLoop` 的职责边界是什么？
2. 为什么 tool result 必须进入 messages？
3. 为什么 `convertToLlm` 不是一开始就执行，而是在每次模型调用前执行？
4. `beforeToolCall` 和 `afterToolCall` 分别适合做什么？
5. 如果一次 assistant message 里有多个 tool call，Pi 如何决定并行还是顺序执行？

## 小练习

画一张时序图，标题为：

```text
Pi Agent Loop: prompt -> assistant -> tool -> result -> next turn
```

要求包含：

- user message
- assistant streaming
- tool call extraction
- beforeToolCall
- tool.execute
- afterToolCall
- tool result message
- turn_end
- agent_end

## 复盘产出

完成本课后，应产出：

```text
study/reference/pi-agent-loop-sequence.md
```

内容包括：

- 一张 ASCII 时序图。
- 关键函数列表。
- 每个事件的触发位置。
- 3 个最重要的理解。

如果能正确回答检查问题，再考虑写学习记录：

```text
study/learning-records/0001-agent-loop-is-model-tool-observation-cycle.md
```
