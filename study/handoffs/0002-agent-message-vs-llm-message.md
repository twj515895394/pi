# Handoff：0002 AgentMessage vs LLM Message

## 本节课状态

- 状态：已完成
- 日期：2026-06-23
- 对应课程：`study/course-design/0002-agent-message-vs-llm-message.md`
- 对应课件：`study/lessons/0002-agent-message-vs-llm-message.html`
- 对应参考资料：`study/reference/agent-message-vs-llm-message.md`

## 本节课目标

理解 Pi 为什么要区分内部消息模型和 LLM 消息模型，并掌握 `convertToLlm` 的边界意义。

## 实际学习内容

本节课完成了以下内容：

- 解释 `AgentMessage` 与 `LLM Message` 的职责边界。
- 阅读 `CustomAgentMessages` 与 `AgentMessage` 的定义。
- 阅读 `defaultConvertToLlm` 的默认过滤逻辑。
- 阅读 Harness 层 `bashExecution`、`custom`、`branchSummary`、`compactionSummary` 的自定义消息设计。
- 阅读 Harness 层 `convertToLlm` 如何把内部消息转换成 LLM 可消费消息。
- 结合企业清关 Agent 讨论 `AuditEventMessage`、`PermissionDecisionMessage`、`ClearanceStatusMessage` 是否进入 LLM。

## 已确认理解

用户已经能够用自己的话说明：

1. `AgentMessage` 范围更大，是 Agent 流程链路内部使用的消息模型。
2. `LLM Message` 范围更小，是对接 LLM 外部接口的 DTO 契约。
3. `branchSummary` 和 `compactionSummary` 虽然是内部消息，但它们包含模型继续推理所需的上下文，所以需要转换后进入 LLM。
4. `bashExecution` 需要 `excludeFromContext`，因为有些命令输出模型需要，有些不需要；过滤可以避免上下文臃肿、噪音和安全风险。
5. `AuditEventMessage` 默认不进入 LLM，因为它主要服务审计、风控和 Agent 级处理。
6. `ClearanceStatusMessage` 可以按需进入 LLM，但最好转换成清晰、简洁的上下文，而不是原样塞给模型。
7. `PermissionDecisionMessage` 默认不进入 LLM，只有在安全边界会影响模型下一步行为时，才给模型一个简化结论。

## 已纠正和补充的问题

本节补充了两个边界：

1. `bashExecution.excludeFromContext` 不只是“模型需不需要”的问题，还涉及 token 成本、上下文噪音和安全泄露风险。
2. 审批 / 权限消息默认不进入 LLM；只有“会影响模型下一步行为的结论”才应该转换成 LLM 上下文。

## 源码阅读进度

已阅读：

```text
study/course-design/0002-agent-message-vs-llm-message.md
packages/agent/src/types.ts
packages/agent/src/agent.ts
packages/agent/src/harness/messages.ts
```

关键源码点：

```text
CustomAgentMessages
AgentMessage
AgentLoopConfig.convertToLlm
AgentLoopConfig.transformContext
defaultConvertToLlm
BashExecutionMessage
CustomMessage
BranchSummaryMessage
CompactionSummaryMessage
harness/messages.ts convertToLlm()
```

## 练习进度

已完成口头理解检查。

消息模型对照表已沉淀到：

```text
study/reference/agent-message-vs-llm-message.md
```

## 文档更新情况

已更新：

```text
study/reference/agent-message-vs-llm-message.md
study/GLOSSARY.md
study/PROGRESS.md
study/CURRENT.md
```

已新增：

```text
study/lessons/0002-agent-message-vs-llm-message.html
study/handoffs/0002-agent-message-vs-llm-message.md
study/learning-records/0002-agent-message-is-internal-domain-model.md
```

## 下节课计划

下一节课进入：

```text
0003：Tool Call 执行链路
```

下节课要解决的问题：

```text
一次 assistant message 中的 toolCall 是如何被校验、拦截、执行、改写，并最终变成 toolResult message 的？
```

建议源码范围：

```text
packages/agent/src/agent-loop.ts
packages/agent/src/types.ts
packages/coding-agent/src/core/tools/index.ts
```

## 新会话续接提示

如果新开会话，请说：

```text
继续 Pi Agent Harness 学习。请先读取 study/README.md、study/CURRENT.md、study/PROGRESS.md 和 study/handoffs/0002-agent-message-vs-llm-message.md，然后从第三课 Tool Call 执行链路开始。
```
