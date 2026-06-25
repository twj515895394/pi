# 当前学习状态

本文件是新会话续接学习时的第一入口。任何 Agent 进入 `study/` 工作区后，应先阅读本文件，再阅读最新 handoff。

## 当前课程

`0006`：Extension System

## 当前状态

`0005：Session Tree 与 Compaction` 已完成理解检查，并已写入 learning-record 与 0005 博文草稿。下一步进入 `0006：Extension System`。

第一课 `0001：Pi Agent Loop 源码精读` 已完成。
第二课 `0002：AgentMessage vs LLM Message` 已完成。
第三课 `0003：Tool Call 执行链路` 已完成。
第四课 `0004：AgentHarness 是 Runtime 控制层` 已完成。
第五课 `0005：Session Tree 与 Compaction` 已完成。
第六课 `0006：Extension System` 未开始。

## 主计划

```text
study/MASTER-PLAN.md
```

`MASTER-PLAN.md` 是唯一权威学习主计划。后续课程、补课、小课和阶段练习都应服务该主计划。

## 教学执行协议

```text
study/TEACHING-PROTOCOL.md
```

新会话必须读取该文件，用于恢复固定讲课节奏、结构化讲解方式、理解检查方式和收尾规则。

如果用户只说“继续下一节课”，也不能简单输出摘要；应主动读取学习工作区文件，并按教学协议执行。

## 最新 Handoff

```text
study/handoffs/0005-session-tree-and-compaction.md
```

## 已确认理解

用户已经理解：

```text
AgentLoop 是执行内核；
Agent 是带状态的调用门面；
AgentHarness 是把执行内核接入 session、模型、工具、权限、事件、队列、长期上下文和 UI/CLI/SDK 的 Runtime 控制层。

Session Tree 不是简单 messages[]，而是 Agent Runtime 的运行历史树；
id / parentId / leaf 用于表达 branch path 和当前 active branch；
buildSessionContext() 把完整历史树转换成 LLM 当前上下文视图；
Compaction 处理当前 branch 太长的问题；
Branch Summary 处理从 old branch 切到 new branch 时的路径移交。
```

用户已能解释：

- 为什么 AgentLoop 应保持小而纯。
- 为什么 Session、Compaction、Branch Summary 属于 Harness 层。
- 为什么 AgentHarness 适合承载企业权限、审计、provider 请求治理和 UI 事件。
- 为什么 createLoopConfig 是 Harness 到 AgentLoop 的适配器 / 注入器。
- 为什么 tool_call hook 比直接改 executeToolCalls 更低耦合。
- 为什么 Session Tree 比 `messages[]` 更适合多路径探索、回退、审计和长上下文治理。
- 为什么 Compaction 与 Branch Summary 不是同一个东西。

## 0005 完成材料

```text
study/course-design/0005-session-tree-and-compaction.md
study/lessons/0005-session-tree-and-compaction.html
study/reference/session-tree-and-compaction.md
study/handoffs/0005-session-tree-and-compaction.md
study/learning-records/0005-session-tree-is-runtime-history-tree.md
study/blogs/drafts/0005-session-tree-and-compaction.md
```

## 下一步：0006 Extension System

下一课建议按以下顺序开始：

```text
1. 读取 study/MASTER-PLAN.md 中 0006 相关目标。
2. 读取 Extension System 相关源码入口。
3. 先建立 Extension 的职责边界：它是 Harness 扩展点，不是 AgentLoop 内核。
4. 对照前面课程中出现的 hook：tool_call、session_before_compact、session_before_tree 等。
5. 用金融行业 Agent 泛化例子说明 Extension 如何做权限、审计、上下文注入和策略控制。
```

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

企业案例统一使用泛化的金融行业 Agent，不暴露真实企业行业方向。

该偏好已记录到：

```text
study/NOTES.md
study/TEACHING-PROTOCOL.md
```

## 新会话续接方式

新会话中请先阅读：

1. `study/README.md`
2. `study/MASTER-PLAN.md`
3. `study/TEACHING-PROTOCOL.md`
4. `study/MISSION.md`
5. `study/CURRENT.md`
6. `study/PROGRESS.md`
7. `study/handoffs/0005-session-tree-and-compaction.md`

然后进入第六课 `0006：Extension System`。
