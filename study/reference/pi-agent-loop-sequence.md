# Pi Agent Loop 时序图

本参考资料沉淀第一课 `0001：Pi Agent Loop 源码精读` 的核心结论。

## 一句话结论

Pi 的 Agent Loop 是一个模型与工具之间的运行闭环：用户输入被统一成 `AgentMessage`，模型输出 `assistant message`，如果包含 `toolCall` 就执行工具，工具结果再包装成 `toolResult message` 回到上下文，模型据此继续推理或生成最终回答。

## 主调用链

```text
Agent.prompt(input)
  -> normalizePromptInput(input)
  -> runPromptMessages(messages)
  -> runWithLifecycle(signal => ...)
  -> createContextSnapshot()
  -> createLoopConfig()
  -> runAgentLoop(prompts, context, config, emit, signal, streamFn)
  -> runLoop(currentContext, newMessages, config, signal, emit, streamFn)
  -> streamAssistantResponse(currentContext, config, signal, emit, streamFn)
  -> executeToolCalls(currentContext, assistantMessage, config, signal, emit)
  -> createToolResultMessage(finalized)
  -> push toolResult back to currentContext.messages
  -> next LLM turn or agent_end
```

## ASCII 时序图

```text
User
  |
  v
Agent.prompt(input)
  |
  v
normalizePromptInput()
  |  string -> [{ role: "user", content, timestamp }]
  v
runPromptMessages()
  |
  |-- createContextSnapshot()
  |-- createLoopConfig()
  v
runAgentLoop()
  |
  |-- emit agent_start
  |-- emit turn_start
  |-- emit user message_start/message_end
  v
runLoop()
  |
  v
streamAssistantResponse()
  |
  |-- transformContext(AgentMessage[])
  |-- convertToLlm(AgentMessage[] -> Message[])
  |-- streamFunction(model, llmContext)
  |-- emit assistant message_start/update/end
  v
assistant message
  |
  |-- no toolCall
  |     -> emit turn_end
  |     -> emit agent_end
  |
  |-- has toolCall
        |
        v
      executeToolCalls()
        |
        |-- prepareToolCall()
        |     |-- find tool
        |     |-- prepareArguments
        |     |-- validateToolArguments
        |     |-- beforeToolCall
        |
        |-- tool.execute()
        |     |-- optional tool_execution_update
        |
        |-- afterToolCall()
        |
        |-- createToolResultMessage()
        v
      toolResult message
        |
        v
      currentContext.messages.push(toolResult)
        |
        v
      next LLM turn
```

## 关键函数列表

| 函数 | 文件 | 作用 |
|---|---|---|
| `Agent.prompt()` | `packages/agent/src/agent.ts` | 外部输入入口，防止同一 Agent 并发 prompt。 |
| `normalizePromptInput()` | `packages/agent/src/agent.ts` | 把 string / AgentMessage / AgentMessage[] 统一成 AgentMessage[]。 |
| `runPromptMessages()` | `packages/agent/src/agent.ts` | 创建生命周期包装，并调用 `runAgentLoop()`。 |
| `createContextSnapshot()` | `packages/agent/src/agent.ts` | 从 Agent state 中提取 systemPrompt、messages、tools 的运行快照。 |
| `createLoopConfig()` | `packages/agent/src/agent.ts` | 把 model、tool hooks、context transform、message converter、queue reader 等能力注入 loop。 |
| `runAgentLoop()` | `packages/agent/src/agent-loop.ts` | 把 prompts 接入上下文，发出起始事件，然后进入 `runLoop()`。 |
| `runLoop()` | `packages/agent/src/agent-loop.ts` | 主循环，处理 assistant response、tool calls、steering、follow-up 和结束条件。 |
| `streamAssistantResponse()` | `packages/agent/src/agent-loop.ts` | 真正调用 LLM 的地方，调用前执行 `transformContext` 和 `convertToLlm`。 |
| `executeToolCalls()` | `packages/agent/src/agent-loop.ts` | 根据配置和工具声明选择并行或顺序执行。 |
| `prepareToolCall()` | `packages/agent/src/agent-loop.ts` | 工具执行前的查找、参数处理、参数校验和 `beforeToolCall` 拦截。 |
| `executePreparedToolCall()` | `packages/agent/src/agent-loop.ts` | 真正调用 `tool.execute()`。 |
| `finalizeExecutedToolCall()` | `packages/agent/src/agent-loop.ts` | 执行 `afterToolCall`，允许修改工具结果。 |
| `createToolResultMessage()` | `packages/agent/src/agent-loop.ts` | 把工具结果包装成 `role: "toolResult"` 的消息。 |
| `processEvents()` | `packages/agent/src/agent.ts` | Agent 消费 loop 事件并更新自身状态。 |

## 事件触发位置

| 事件 | 触发含义 |
|---|---|
| `agent_start` | 一次 Agent run 开始。 |
| `turn_start` | 一轮模型调用流程开始。 |
| `message_start` | 某条消息开始进入事件流。 |
| `message_update` | assistant streaming 过程中出现增量更新。 |
| `message_end` | 某条消息最终定稿并可写入 state。 |
| `tool_execution_start` | 工具执行开始，UI 可据此显示工具运行中。 |
| `tool_execution_update` | 工具执行过程中的部分结果或进度更新。 |
| `tool_execution_end` | 工具执行结束，UI 可据此移除运行中状态。 |
| `turn_end` | 一轮 assistant response 与相关工具执行结束。 |
| `agent_end` | 本次 Agent run 结束。 |

## 三个关键理解

1. `Agent` 是有状态封装，`AgentLoop` 是底层运行循环。Loop 不直接改 Agent state，而是 emit 事件，由 `Agent.processEvents()` 统一消费和更新状态。
2. Pi 内部流转的是 `AgentMessage[]`；真正调用 LLM 前，才通过 `transformContext()` 和 `convertToLlm()` 生成模型可理解的 `Message[]`。
3. 工具结果不会直接返回给用户，而是包装成 `toolResult message` 回到上下文，让模型基于工具结果继续推理、继续调用工具或生成最终回答。

## 已纠正的易错点

- `uiStatus` / notification / UI-only status 这类消息通常应该在 `convertToLlm` 阶段过滤，因为它们属于内部状态，不是 LLM 的推理上下文。
- UI 展示工具执行状态时，应关注 `tool_execution_start`、`tool_execution_update`、`tool_execution_end`，而不是泛称为 `toolcallupdate/toolcallend`。
