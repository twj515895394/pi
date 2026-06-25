# Agent Loop：一个 Agent 到底是怎么跑起来的？

> 系列：从 Dify 到 Agent Runtime：我的 Pi Agent Harness 源码学习笔记  
> 对应课程：0001 Pi Agent Loop 源码精读  
> 状态：草稿

## 开篇：我以前对 Agent 的理解太“应用层”了

在使用 Dify、Coze 或各种 Agent 平台时，我们很容易把 Agent 理解成：

```text
用户输入 -> 大模型回复 -> 必要时调用工具 -> 返回结果
```

这个理解没错，但它太粗了。真正读源码时会发现，一个 Agent 并不是简单调用一次 LLM，也不是简单把工具结果返回给用户。

更准确地说，Agent 是一个运行闭环：

```text
用户输入
  -> 内部消息
  -> 模型响应
  -> 工具调用
  -> 工具结果回灌
  -> 模型继续推理
  -> 最终回答
```

这就是我学习 Pi Agent Harness 第一课时最重要的收获：**Agent Loop 是模型和工具之间的运行闭环，而不是一次普通聊天请求。**

## 一句话结论

```text
Agent Loop 的核心不是“调用模型”，而是持续维护模型、工具和上下文之间的闭环。
```

## 一次 prompt 大概经历了什么？

从学习结果看，Pi 里一次用户输入大致会经过这条链路：

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
  -> push toolResult back to messages
  -> next LLM turn or agent_end
```

这条链路看起来长，但其实职责很清晰：

| 步骤 | 作用 |
|---|---|
| `Agent.prompt()` | 外部入口，接收用户输入 |
| `normalizePromptInput()` | 把 string / message / messages 统一成内部消息格式 |
| `createContextSnapshot()` | 从 Agent 当前状态里生成本轮运行快照 |
| `createLoopConfig()` | 注入模型、工具、消息转换、Hook 等配置 |
| `runAgentLoop()` | 真正进入底层运行循环 |
| `streamAssistantResponse()` | 调用 LLM，接收 assistant message |
| `executeToolCalls()` | 如果 assistant message 里有 toolCall，就执行工具 |
| `createToolResultMessage()` | 把工具结果包装成 toolResult message |

这里最关键的是：工具结果不是直接返回给用户，而是回到上下文，继续交给模型推理。

## Agent Loop 的最小闭环图

```text
User
  |
  v
Agent.prompt()
  |
  v
AgentMessage[]
  |
  v
LLM
  |
  v
AssistantMessage
  |
  |-- 无 toolCall -> 最终回答
  |
  |-- 有 toolCall
        |
        v
      Tool Runtime
        |
        v
      ToolResultMessage
        |
        v
      回到 AgentMessage[]
        |
        v
      下一轮 LLM
```

这个图让我意识到，Agent 的关键不是“能不能调工具”，而是“工具结果如何成为模型下一步推理的一部分”。

## 为什么工具结果要回灌给模型？

假设一个金融客服 Agent 收到用户问题：

```text
帮我查一下这笔贷款申请为什么还没审批通过？
```

模型可能会生成一个工具调用：

```text
queryLoanApplicationStatus(applicationId)
```

工具返回结果后，如果系统直接把工具结果展示给用户，那么模型其实没有机会继续判断：

```text
是否需要解释原因？
是否需要调用另一个风控记录查询工具？
是否需要提醒用户补充材料？
是否涉及合规拒绝原因不能直说？
```

所以更合理的方式是：

```text
工具结果 -> toolResult message -> 回到上下文 -> LLM 继续推理 -> 生成最终用户可读回答
```

这就是 Agent Loop 和普通 API 调用最大的差异。

## currentContext.messages 和 newMessages 的区别

第一课里还有一个很重要的概念：`currentContext.messages` 和 `newMessages`。

可以这样理解：

| 名称 | 作用 |
|---|---|
| `currentContext.messages` | 当前 run 的完整上下文，模型每次推理时要看它 |
| `newMessages` | 本次 run 新产生的消息增量，用于事件、状态更新和最终返回 |

类比 Java 后端：

```text
currentContext.messages ≈ 当前事务内的完整业务上下文
newMessages ≈ 本次请求新产生的变更日志
```

这个分离很有价值。因为 Agent Loop 一边要让模型看到完整上下文，一边还要把本次新增的消息通知外层状态管理器。

## 为什么 AgentLoop 不直接修改 Agent 状态？

源码设计里，AgentLoop 不是直接改外层 Agent 的 state，而是通过事件通知外层。

这很像后端里的事件驱动：

```text
AgentLoop 负责执行
Agent 负责消费事件并更新状态
```

这种方式的好处是：

```text
执行逻辑和状态管理解耦
UI 可以订阅事件
外层可以记录日志
运行过程可以流式展示
未来可以接 CLI / SDK / RPC
```

如果 AgentLoop 直接改状态，底层循环会越来越重，也很难复用。

## 少量伪代码理解

可以把 Agent Loop 简化成这样：

```ts
while (true) {
  const assistant = await callLLM(context.messages);
  context.messages.push(assistant);

  const toolCalls = extractToolCalls(assistant);
  if (toolCalls.length === 0) break;

  const toolResults = await executeToolCalls(toolCalls);
  context.messages.push(...toolResults);
}
```

这不是源码，只是帮助理解。

真正复杂的地方在于：

```text
消息要转换
上下文要裁剪
工具要校验
工具执行要发事件
工具失败也要回灌
运行中可能有用户插话
```

所以真实的 Agent Loop 会比这段伪代码复杂得多。

## 常见误区

### 误区 1：Agent 就是 LLM + Tools

不够准确。

更准确是：

```text
Agent = LLM + Tools + Message Context + Runtime Loop + Event Stream
```

### 误区 2：工具结果直接返回给用户

很多简单 Demo 会这么做，但工程化 Agent 不应该这样。工具结果应该先成为模型可观察的上下文，再由模型生成用户可读回答。

### 误区 3：AgentLoop 应该负责所有事情

不应该。AgentLoop 应该保持小而纯，负责当前 run 的模型-工具闭环。Session、权限、审计、长期上下文、UI 事件等更适合放到 Harness 层。

## 我的阶段性理解

第一课之后，我对 Agent 的理解从“会调用工具的聊天机器人”变成了“一个持续运行的模型-工具-上下文闭环”。

这也是为什么企业级 Agent 不能只关心 Prompt 和工具列表。真正难的是：

```text
工具结果如何回灌？
失败如何让模型知道？
上下文如何持续维护？
事件如何通知外层？
状态如何持久化？
```

这些问题，后面会一路引出 AgentMessage、Tool Call Pipeline、AgentHarness、Session Tree 和 Extension System。

## 小结

本文可以压缩成一句话：

```text
Agent Loop 是 Agent Runtime 的执行内核，它让用户输入、模型响应、工具调用和工具结果形成一个可持续推理的闭环。
```

如果你只把 Agent 当成“一次模型调用”，就很难理解真正的 Agent 工程化；但一旦把它看成闭环，后面的消息模型、工具治理和 Harness 控制层就都能串起来了。
