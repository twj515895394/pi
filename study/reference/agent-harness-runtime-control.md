# AgentHarness 是 Runtime 控制层

本参考资料沉淀第四课 `0004：AgentHarness 是 Runtime 控制层` 的核心结论。

## 一句话结论

`AgentLoop` 是执行内核，`Agent` 是带状态的调用门面，`AgentHarness` 是把执行内核接入 Session、模型、工具、资源、权限、事件、Provider 请求、队列、Compaction、Branch Summary 和 UI/CLI/SDK 的 Runtime 控制层。

## 三层职责边界

| 层级 | 类比 Java 后端 | 核心职责 |
|---|---|---|
| `AgentLoop` | 核心工作流函数 | 当前 run 内的模型调用、toolCall 检测、工具执行、toolResult 回灌 |
| `Agent` | 有状态业务 Service | 保存 messages / tools / streaming 状态，并调用 AgentLoop |
| `AgentHarness` | 应用运行容器 / Runtime 控制层 | 管 Session、模型、工具、资源、队列、Hook、Provider、Compaction、Branch Summary 和事件 |

核心理解：

```text
AgentLoop 只负责这一轮怎么跑；
AgentHarness 负责这个 Agent 应用怎么长期、稳定、可治理地运行。
```

## Runtime 控制层负责什么

```text
入口控制：用户请求怎么进入 Agent。
上下文控制：本轮带哪些历史消息和业务上下文。
模型控制：用哪个 model、thinking level、provider options。
工具控制：注册哪些工具，本轮启用哪些工具。
权限控制：toolCall 执行前是否允许。
结果控制：toolResult 是否需要脱敏、压缩、错误标准化。
会话控制：消息如何落库，什么时候 save point。
运行控制：运行中能否 steer / followUp / abort。
长期上下文控制：什么时候 compact，分支跳转是否生成 branch summary。
事件控制：UI / CLI / SDK 如何感知运行状态。
```

## TurnState 与 AgentContext

`createTurnState()` 负责从 Harness 内部状态组装本轮运行快照：

```text
session context
resources
session metadata
tools / activeTools
systemPrompt
model
thinkingLevel
streamOptions
```

`createContext()` 再把富状态 TurnState 转成 AgentLoop 需要的最小上下文：

```text
systemPrompt
messages
tools
```

## createLoopConfig 的架构意义

`createLoopConfig()` 不是普通配置对象，而是 Harness 到 AgentLoop 的适配器 / 注入器。

| AgentLoopConfig 字段 | Harness 侧能力 |
|---|---|
| `convertToLlm` | 使用 Harness 消息转换器。 |
| `transformContext` | 触发 `context` hook，做上下文治理。 |
| `beforeToolCall` | 触发 `tool_call` hook，做执行前权限、审批和风控。 |
| `afterToolCall` | 触发 `tool_result` hook，做执行后脱敏、压缩、错误标准化。 |
| `prepareNextTurn` | flush session writes，重新 createTurnState。 |
| `getSteeringMessages` | 从 steerQueue 拉取运行中插话。 |
| `getFollowUpMessages` | 从 followUpQueue 拉取后续消息。 |

核心理解：

```text
AgentLoop 保持小而纯；
Harness 通过 createLoopConfig 把企业级治理能力注入进去。
```

## Harness 事件体系

Harness 事件 = AgentLoop 事件 + Harness 自己的 Runtime 事件。

| 事件类型 | 用途 |
|---|---|
| `message_start/update/end` | UI 展示消息流。 |
| `tool_execution_start/update/end` | UI 展示工具执行过程。 |
| `queue_update` | 展示 steer/followUp/nextTurn 队列。 |
| `save_point` | 会话已保存。 |
| `settled` | 本轮运行真正稳定结束。 |
| `before_agent_start` | 运行前注入消息或系统提示词。 |
| `context` | 上下文治理。 |
| `tool_call` | 工具执行前权限/风控。 |
| `tool_result` | 工具执行后结果治理。 |
| `session_compact` | 上下文压缩完成。 |
| `session_tree` | 会话树跳转完成。 |
| `model_update/tools_update/resources_update` | Runtime 配置变更。 |

## Session 不只是聊天记录

Pi 的 Session Tree Entry 包含：

```text
message
thinking_level_change
model_change
active_tools_change
compaction
branch_summary
custom
custom_message
label
session_info
leaf
```

这说明 Session 是 Agent 运行历史树，不只是聊天 transcript。

## 企业清关 Agent Harness 示例

一个企业级清关 Agent Harness 应放置：

```text
入口控制：用户请求合法性、租户、用户身份。
上下文控制：历史消息、当前清关单、业务上下文。
模型控制：模型选择、thinkingLevel、provider 参数。
工具控制：当前角色可用哪些清关工具。
权限控制：tool_call hook 执行前审批、权限、风控。
结果控制：tool_result hook 执行后脱敏、压缩、错误标准化。
会话控制：消息落库、save point、恢复上下文。
审计控制：请求日志、工具调用日志、provider 调用记录。
UI 控制：message/tool/queue/save_point/settled 事件推送。
长期上下文控制：compaction 与 branch summary。
```

## 本课关键理解

1. `AgentLoop` 是执行内核，不应该承载长期状态和企业治理能力。
2. `AgentHarness` 负责把 AgentLoop 接入真实应用运行环境。
3. `createTurnState()` 组装本轮运行快照。
4. `createContext()` 把 TurnState 压成 AgentLoop 所需的最小上下文。
5. `createLoopConfig()` 是 Harness 到 AgentLoop 的适配器和能力注入器。
6. 企业级权限、审计、Provider 治理、UI 事件、Session 持久化都更适合放在 Harness 层。
