# Handoff：0004 AgentHarness 是 Runtime 控制层

## 本节课状态

- 状态：已完成
- 日期：2026-06-24
- 对应课程：`study/course-design/0004-agent-harness-runtime-control.md`
- 对应课件：`study/lessons/0004-agent-harness-runtime-control.html`
- 对应参考资料：`study/reference/agent-harness-runtime-control.md`

## 本节课目标

理解 Pi 中 `AgentHarness` 为什么不是简单封装 `Agent`，而是把模型、工具、消息、权限、事件、上下文、Session、Compaction、Branch Summary 和 UI/CLI/SDK 连接起来的 Runtime 控制层。

## 实际学习内容

本节课完成了以下内容：

- 解释 `AgentLoop`、`Agent`、`AgentHarness` 三层职责边界。
- 阅读 `AgentHarness` 持有的 Runtime 状态：env、session、models、model、tools、activeToolNames、resources、queues、handlers 等。
- 阅读 `createTurnState()` 如何组装本轮运行快照。
- 阅读 `createContext()` 如何把富状态 TurnState 转为 AgentLoop 的最小 AgentContext。
- 阅读 `createLoopConfig()` 如何把 Harness 的 Runtime 能力注入 AgentLoop。
- 阅读 Harness 事件系统如何把 AgentLoop 事件与 Harness 自有事件统一起来。
- 阅读 `handleAgentEvent()` 如何消费 message/turn/agent 事件并写 session、发 save_point、settled。
- 讨论为什么 Session、Compaction、Branch Summary 属于 Harness 层，而不是 AgentLoop 层。
- 结合企业清关 Agent 讨论 Harness 层应承载哪些能力。

## 已确认理解

用户已经能够用自己的话说明：

1. `AgentLoop` 只是某一轮任务执行，只负责模型-工具闭环及结果消息回灌。
2. Session、Compaction、Branch Summary 属于 Harness 管理，因为它们是跨 turn、跨 run、跨会话树的长期状态治理。
3. `AgentHarness` 负责完整执行流程各阶段的协调，适合承载权限、审计、provider 请求治理、UI 事件和上下文控制。
4. `createLoopConfig()` 不是普通配置对象，而是把 Harness Runtime 能力和 hooks 注入到 AgentLoop 的适配器。
5. 企业清关 Agent Harness 应放置用户请求合法性校验、权限整合、上下文整理、工具整理、模型选择、审计日志、请求日志、安全风控、业务工具集合和配置管理等能力。
6. `tool_call hook` 比直接改 `executeToolCalls()` 更适合企业权限扩展，因为它职责清晰、低耦合、可单独扩展。
7. UI 应更关注 Harness 事件，因为 UI 需要看到 message update、工具过程、队列、save_point、settled 等应用运行状态，而不是只看 AgentLoop 内核。

## 已纠正和补充的问题

本节补充了几个边界：

1. `AgentLoop` 应保持小而纯，只负责当前 run 的局部闭环。
2. `AgentHarness` 是 Agent 应用运行容器，不是简单调用模型的 wrapper。
3. `createLoopConfig()` 的重点不是“配置集合”，而是“适配器 / 注入器”。
4. `Session` 不是简单聊天记录，而是 Agent 运行历史树，包含 message、model_change、active_tools_change、compaction、branch_summary 等。
5. 对高概念密度课程，后续教学应默认使用表格、流程图、设计图、小结卡片，并在必要时生成图片辅助理解。

## 源码阅读进度

已阅读：

```text
packages/agent/src/index.ts
packages/agent/src/harness/agent-harness.ts
packages/agent/src/harness/types.ts
```

关键源码点：

```text
AgentHarness
AgentHarnessTurnState
createTurnState()
createContext()
createStreamFn()
createLoopConfig()
handleAgentEvent()
executeTurn()
skill()
promptFromTemplate()
steer()
followUp()
nextTurn()
appendMessage()
compact()
navigateTree()
setModel()
setThinkingLevel()
setTools()
setActiveTools()
setResources()
AgentHarnessOwnEvent
AgentHarnessEventResultMap
SessionTreeEntry
```

## 练习进度

已完成理解检查。

Runtime 分层图和职责边界已沉淀到：

```text
study/reference/agent-harness-runtime-control.md
```

用户明确提出新的教学偏好：高概念密度课程应更多使用结构化表格、流程图、设计图和小结卡片。该偏好已记录到：

```text
study/NOTES.md
```

## 文档更新情况

已更新：

```text
study/NOTES.md
study/GLOSSARY.md
study/PROGRESS.md
study/CURRENT.md
```

已新增：

```text
study/course-design/0004-agent-harness-runtime-control.md
study/reference/agent-harness-runtime-control.md
study/lessons/0004-agent-harness-runtime-control.html
study/handoffs/0004-agent-harness-runtime-control.md
study/learning-records/0004-agent-harness-is-runtime-control-plane.md
```

## 下节课计划

下一节课进入：

```text
0005：Session Tree 与 Compaction
```

下节课要解决的问题：

```text
Pi 的 Session 为什么不是简单聊天记录，而是一个包含 message、model_change、active_tools_change、compaction、branch_summary 等节点的运行历史树？Compaction 如何在长上下文中保持模型可继续推理？
```

建议源码范围：

```text
packages/agent/src/harness/session/session.ts
packages/agent/src/harness/session/jsonl-repo.ts
packages/agent/src/harness/types.ts
packages/agent/src/harness/compaction/compaction.ts
packages/agent/src/harness/compaction/branch-summarization.ts
```

## 新会话续接提示

如果新开会话，请说：

```text
继续 Pi Agent Harness 学习。请先读取 study/README.md、study/CURRENT.md、study/PROGRESS.md 和 study/handoffs/0004-agent-harness-runtime-control.md，然后从第五课 Session Tree 与 Compaction 开始。以后高概念密度课程请默认使用表格、流程图和设计图辅助理解。
```
