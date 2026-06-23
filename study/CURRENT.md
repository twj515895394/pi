# 当前学习状态

本文件是新会话续接学习时的第一入口。任何 Agent 进入 `study/` 工作区后，应先阅读本文件，再阅读最新 handoff。

## 当前课程

`0003`：Tool Call 执行链路

## 当前状态

未开始。

第一课 `0001：Pi Agent Loop 源码精读` 已完成。
第二课 `0002：AgentMessage vs LLM Message` 已完成。

## 最新 Handoff

```text
study/handoffs/0002-agent-message-vs-llm-message.md
```

## 已确认理解

用户已经理解：

```text
AgentMessage = Agent 内部完整消息账本
LLM Message  = LLM Provider 接口 DTO
convertToLlm = 内部消息到模型协议的边界转换器
```

用户已能解释：

- `AgentMessage` 为什么比 `LLM Message` 范围更大。
- `branchSummary` / `compactionSummary` 为什么转换后进入 LLM。
- `bashExecution.excludeFromContext` 的作用。
- `AuditEventMessage` 默认不应进入 LLM。
- `ClearanceStatusMessage` 应按需转换后进入 LLM。
- `PermissionDecisionMessage` 默认不进 LLM，只在安全结论影响下一步行为时转换给模型。

## 下一步

进入第三课：

```text
study/course-design/0003-tool-call-execution.md
```

第三课要解决的问题：

```text
一次 assistant message 中的 toolCall 是如何被校验、拦截、执行、改写，并最终变成 toolResult message 的？
```

## 新会话续接方式

新会话中请先阅读：

1. `study/README.md`
2. `study/MISSION.md`
3. `study/CURRENT.md`
4. `study/PROGRESS.md`
5. `study/handoffs/0002-agent-message-vs-llm-message.md`

然后从第三课开始。