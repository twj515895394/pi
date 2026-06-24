# Tool Call 生命周期

本参考资料沉淀第三课 `0003：Tool Call 执行链路` 的核心结论。

## 一句话结论

LLM 只能提出 `toolCall`；Agent Runtime 才真正决定工具能不能执行、如何执行、执行结果如何治理，以及如何包装成 `toolResult message` 回灌给模型。

## 主生命周期

```text
assistant message contains toolCall
  -> filter content blocks where type === "toolCall"
  -> executeToolCalls()
  -> choose sequential / parallel
  -> emit tool_execution_start
  -> prepareToolCall()
      -> find tool by name
      -> prepareArguments
      -> validateToolArguments
      -> beforeToolCall
  -> executePreparedToolCall()
      -> tool.execute(toolCallId, args, signal, onUpdate)
      -> optional tool_execution_update
  -> finalizeExecutedToolCall()
      -> afterToolCall
  -> emit tool_execution_end
  -> createToolResultMessage()
  -> emit message_start/message_end for toolResult
  -> append toolResult to currentContext.messages
  -> next LLM turn or agent_end
```

## 关键函数列表

| 函数 | 职责 |
|---|---|
| `executeToolCalls()` | 从 assistant message 中取出 toolCall，并选择顺序或并行执行。 |
| `executeToolCallsSequential()` | 每个 toolCall 完整准备、执行、finalize、生成 toolResult 后再处理下一个。 |
| `executeToolCallsParallel()` | 先逐个 prepare，再并发执行允许并发的工具，最后按 assistant 原始顺序生成 toolResult message。 |
| `prepareToolCall()` | 工具执行前的核心安全关口：find tool、prepare args、validate args、beforeToolCall。 |
| `prepareToolCallArguments()` | 调用工具自定义 `prepareArguments`，对模型参数做兼容和修正。 |
| `validateToolArguments()` | 按工具 schema 做正式参数校验。 |
| `executePreparedToolCall()` | 真正调用 `tool.execute()`，并处理工具执行更新和异常。 |
| `finalizeExecutedToolCall()` | 调用 `afterToolCall`，允许对结果做脱敏、压缩、标准化、错误改写和 terminate 控制。 |
| `createErrorToolResult()` | 把错误包装成工具结果，避免工具失败直接打断模型观察链。 |
| `createToolResultMessage()` | 把最终工具结果包装成 `role: "toolResult"` 的 AgentMessage。 |

## sequential 与 parallel 差异

| 模式 | 行为 | 适用场景 |
|---|---|---|
| sequential | 一个工具完整完成后再执行下一个 | 文件写入、编辑、状态修改、有顺序依赖的业务操作 |
| parallel | 先逐个 prepare，再并发执行，toolResult message 按 assistant 原始顺序输出 | 多个只读查询、互不依赖的读取或检索操作 |

并行模式下，`tool_execution_end` 可以按真实完成顺序发出，便于 UI 及时展示；但 `toolResult message` 应按 assistant 原始 toolCall 顺序回灌，保持 LLM 上下文稳定。

## prepareArguments 与 validateToolArguments

```text
prepareArguments：
  对模型输出的 raw arguments 做兼容、修正、标准化。

validateToolArguments：
  按工具 schema 做正式校验，不合法就失败。
```

类比 Java：

```text
prepareArguments = 参数适配器 / 参数清洗
validateToolArguments = Bean Validation / Schema 校验
```

## beforeToolCall 与 afterToolCall

| Hook | 时机 | 适合做什么 |
|---|---|---|
| `beforeToolCall` | 工具执行前，参数已校验后 | 权限、审批、风控、工具白名单、命令安全检查、业务状态校验 |
| `afterToolCall` | 工具执行后，生成 toolResult 前 | 脱敏、压缩、错误标准化、审计补充、结果改写、强制 terminate |

简化原则：

```text
beforeToolCall 决定能不能执行。
afterToolCall 决定结果如何给模型和用户看。
```

## 工具失败为什么也要变成 toolResult

工具失败可能来自：

```text
- 工具不存在
- 参数不合法
- beforeToolCall 拦截
- tool.execute 抛异常
- afterToolCall 处理失败
```

这些失败都不应该只是写日志或静默跳过，而应该转成 error toolResult，让模型知道：

```text
- 工具没有成功
- 失败原因是什么
- 是否可以换工具
- 是否应该向用户解释失败
- 是否应该停止继续尝试
```

## 企业级 Tool Permission Layer 检查清单

最少应检查：

```text
用户身份：
  userId、角色、部门、租户、岗位、数据权限范围。

工具权限：
  当前用户是否能调用 read / write / edit / bash / 业务修改工具。

操作类型：
  读、写、删除、审批、状态流转、外部调用、命令执行。

资源范围：
  文件路径、运单号、订单号、客户、仓库、国家、业务线。

数据归属：
  当前用户是否有权访问该资源或该业务对象。

参数风险：
  是否越权、是否绕过校验、是否危险命令、是否大范围修改。

上下文一致性：
  模型请求是否符合用户意图，是否出现越权推断或隐式扩大范围。

审批状态：
  是否需要人工确认、是否已批准、是否被拒绝。

风控策略：
  黑白名单、频率限制、敏感字段、危险操作策略。

审计信息：
  traceId、toolCallId、sessionId、riskLevel、decisionReason。
```

## 清关 Agent 示例

模型请求：

```text
toolCall:
  name: updateClearanceStatus
  args:
    waybillNo: "BUFFALO123"
    status: "CLEARED"
```

执行前：

```text
prepareArguments：标准化 waybillNo 和 status。
validateToolArguments：校验 waybillNo 格式和 status 枚举。
beforeToolCall：检查用户是否有状态修改权限、当前节点是否允许流转、是否需要审批。
```

执行后：

```text
afterToolCall：脱敏内部接口信息，压缩结果，标准化业务错误，必要时设置 terminate。
createToolResultMessage：把执行结果或拒绝原因回灌给模型。
```

## 本课关键理解

1. toolCall 是 assistant message content 中的一个 block，不是可信命令。
2. Agent Runtime 通过 `prepareToolCall` 把 toolCall 变成可控、可校验、可拦截的执行请求。
3. `beforeToolCall` 是执行前治理，`afterToolCall` 是执行后治理。
4. 工具失败、工具被拦截也应该生成 error toolResult，让模型获得可推理的观察结果。
5. 并行执行时，UI 事件可以按真实完成顺序发出，但 toolResult message 应保持 assistant 原始顺序。
