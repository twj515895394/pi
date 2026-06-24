# 当前学习状态

本文件是新会话续接学习时的第一入口。任何 Agent 进入 `study/` 工作区后，应先阅读本文件，再阅读最新 handoff。

## 当前课程

`0005`：Session Tree 与 Compaction

## 当前状态

未开始。

第一课 `0001：Pi Agent Loop 源码精读` 已完成。
第二课 `0002：AgentMessage vs LLM Message` 已完成。
第三课 `0003：Tool Call 执行链路` 已完成。
第四课 `0004：AgentHarness 是 Runtime 控制层` 已完成。

## 最新 Handoff

```text
study/handoffs/0004-agent-harness-runtime-control.md
```

## 已确认理解

用户已经理解：

```text
AgentLoop 是执行内核；
Agent 是带状态的调用门面；
AgentHarness 是把执行内核接入 session、模型、工具、权限、事件、队列、长期上下文和 UI/CLI/SDK 的 Runtime 控制层。
```

用户已能解释：

- 为什么 AgentLoop 应保持小而纯。
- 为什么 Session、Compaction、Branch Summary 属于 Harness 层。
- 为什么 AgentHarness 适合承载企业权限、审计、provider 请求治理和 UI 事件。
- 为什么 createLoopConfig 是 Harness 到 AgentLoop 的适配器 / 注入器。
- 企业清关 Agent Harness 应承载哪些能力。
- 为什么 tool_call hook 比直接改 executeToolCalls 更低耦合。

## 教学偏好提醒

用户明确提出：对于类似 AgentHarness、Session、Compaction 这类内容多、新概念多、环节多的课程，后续应默认使用结构化内容输出，包括：

```text
分层表格
流程图
职责边界图
设计图
小结卡片
必要时生成更美观的图片辅助理解
```

该偏好已记录到：

```text
study/NOTES.md
```

## 下一步

进入第五课：

```text
study/course-design/0005-session-tree-and-compaction.md
```

第五课要解决的问题：

```text
Pi 的 Session 为什么不是简单聊天记录，而是一个包含 message、model_change、active_tools_change、compaction、branch_summary 等节点的运行历史树？Compaction 如何在长上下文中保持模型可继续推理？
```

## 新会话续接方式

新会话中请先阅读：

1. `study/README.md`
2. `study/MISSION.md`
3. `study/CURRENT.md`
4. `study/PROGRESS.md`
5. `study/handoffs/0004-agent-harness-runtime-control.md`

然后从第五课开始。