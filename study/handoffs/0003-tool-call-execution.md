# Handoff：0003 Tool Call 执行链路

## 本节课状态

- 状态：已完成
- 日期：2026-06-24
- 对应课程：`study/course-design/0003-tool-call-execution.md`
- 对应课件：`study/lessons/0003-tool-call-execution.html`
- 对应参考资料：`study/reference/tool-call-lifecycle.md`

## 本节课目标

理解 Pi 中 tool call 从模型生成到工具执行、结果回灌的完整生命周期。

## 实际学习内容

本节课完成了以下内容：

- 解释 assistant message 中的 `toolCall` 如何被筛选出来。
- 阅读 `executeToolCalls()` 如何选择 sequential / parallel。
- 阅读 `prepareToolCall()` 如何完成 find tool、prepare args、validate args、beforeToolCall。
- 阅读 `executePreparedToolCall()` 如何真正调用 `tool.execute()` 并发出 `tool_execution_update`。
- 阅读 `finalizeExecutedToolCall()` 如何通过 `afterToolCall` 改写结果。
- 阅读 `createToolResultMessage()` 如何把工具结果包装成 `toolResult message`。
- 结合企业清关 Agent 讨论 Tool Permission Layer 的最小设计。

## 已确认理解

用户已经能够用自己的话说明：

1. 模型产生的 `toolCall` 不能直接执行，必须先经过参数整理、参数校验、权限审计和风控拦截。
2. `prepareArguments` 是对模型参数做整理、兼容和修改；`validateToolArguments` 是基于处理后的参数做 schema 校验。
3. `beforeToolCall` block 后仍要生成 error toolResult，因为模型需要知道失败原因，从而换工具、给用户解释或停止继续尝试。
4. 并行执行时，工具完成事件顺序和 toolResult message 顺序可以不同；UI 事件面向真实完成顺序，toolResult message 面向稳定上下文顺序。
5. `beforeToolCall` 适合执行前权限、审计、风控校验；`afterToolCall` 适合执行后压缩、脱敏、错误标准化和结果治理。
6. 工具结果中包含手机号、token、内部接口地址时，应在 `afterToolCall` 层处理，避免敏感信息回灌给模型或展示给用户。
7. 业务工具调用被权限系统拒绝时，应返回 error toolResult 给模型，而不是直接抛异常终止 Agent。
8. 企业级 Tool Permission Layer 至少应检查用户权限、读写操作、资源归属、绕过校验风险、黑白名单和风控策略。

## 已纠正和补充的问题

本节补充了两个边界：

1. Tool Permission Layer 的检查项除了用户权限和风控，还应覆盖资源范围、操作类型、数据归属、审批状态、上下文一致性、审计 trace 信息。
2. 工具失败不是异常终点，而是模型可观察的结果；因此失败、拒绝和拦截都应消息化为 error toolResult。

## 源码阅读进度

已阅读：

```text
study/course-design/0003-tool-call-execution.md
packages/agent/src/agent-loop.ts
packages/agent/src/types.ts
packages/coding-agent/src/core/tools/index.ts
packages/coding-agent/src/core/tools/read.ts
```

关键源码点：

```text
executeToolCalls()
executeToolCallsSequential()
executeToolCallsParallel()
prepareToolCall()
prepareToolCallArguments()
validateToolArguments()
executePreparedToolCall()
finalizeExecutedToolCall()
createErrorToolResult()
emitToolExecutionEnd()
createToolResultMessage()
emitToolResultMessage()
BeforeToolCallResult
AfterToolCallResult
AgentTool.execute
```

## 练习进度

已完成口头理解检查。

Tool Call 生命周期图已沉淀到：

```text
study/reference/tool-call-lifecycle.md
```

## 文档更新情况

已更新：

```text
study/reference/tool-call-lifecycle.md
study/GLOSSARY.md
study/PROGRESS.md
study/CURRENT.md
```

已新增：

```text
study/lessons/0003-tool-call-execution.html
study/handoffs/0003-tool-call-execution.md
study/learning-records/0003-tool-call-is-controlled-runtime-pipeline.md
```

## 下节课计划

下一节课进入：

```text
0004：AgentHarness 是 Runtime 控制层
```

下节课要解决的问题：

```text
AgentHarness 为什么不是简单封装 Agent，而是把模型、工具、消息、权限、事件、上下文和 UI/CLI/SDK 连接起来的 Runtime 控制层？
```

建议源码范围：

```text
packages/agent/src/harness/*
packages/agent/src/agent.ts
packages/agent/src/types.ts
packages/coding-agent/src/core/*
```

## 新会话续接提示

如果新开会话，请说：

```text
继续 Pi Agent Harness 学习。请先读取 study/README.md、study/CURRENT.md、study/PROGRESS.md 和 study/handoffs/0003-tool-call-execution.md，然后从第四课 AgentHarness 是 Runtime 控制层开始。
```
