# 当前学习状态

本文件是新会话续接学习时的第一入口。任何 Agent 进入 `study/` 工作区后，应先阅读本文件，再阅读最新 handoff。

## 当前课程

`0005`：Session Tree 与 Compaction

## 当前状态

已完成源码阅读与课程讲解，待理解检查。

第一课 `0001：Pi Agent Loop 源码精读` 已完成。
第二课 `0002：AgentMessage vs LLM Message` 已完成。
第三课 `0003：Tool Call 执行链路` 已完成。
第四课 `0004：AgentHarness 是 Runtime 控制层` 已完成。
第五课 `0005：Session Tree 与 Compaction` 已进入课程讲解，尚未写 learning-record。

## 主计划

```text
study/MASTER-PLAN.md
```

`MASTER-PLAN.md` 是唯一权威学习主计划。后续课程、补课、小课和阶段练习都应服务该主计划。

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
```

用户已能解释：

- 为什么 AgentLoop 应保持小而纯。
- 为什么 Session、Compaction、Branch Summary 属于 Harness 层。
- 为什么 AgentHarness 适合承载企业权限、审计、provider 请求治理和 UI 事件。
- 为什么 createLoopConfig 是 Harness 到 AgentLoop 的适配器 / 注入器。
- 为什么 tool_call hook 比直接改 executeToolCalls 更低耦合。

## 0005 已讲解内容

本课已经覆盖：

```text
SessionTreeEntry 类型联合
id / parentId / leaf 如何形成可导航历史树
JSONL append-only session 存储
buildSessionContext 如何从 current leaf 重建 LLM context
Compaction 如何选择 cut point、生成 summary、保留 firstKeptEntryId
为什么不能切在 toolResult 上
split turn 如何处理超长单轮
Branch Summary 如何在 navigateTree 时保留离开分支的上下文
session_before_compact / session_before_tree 等 Harness hook
金融行业 Agent 类比：信贷审批 Agent 的审计树、检查点摘要、分支移交说明
```

对应材料：

```text
study/course-design/0005-session-tree-and-compaction.md
study/lessons/0005-session-tree-and-compaction.html
study/reference/session-tree-and-compaction.md
study/handoffs/0005-session-tree-and-compaction.md
```

## 待理解检查

请用户用自己的话回答：

1. 为什么 Pi 的 Session 不能只设计成 `messages[]`？至少说出两个工程原因。
2. Compaction 为什么要保存 `firstKeptEntryId`，而不是只保存一个 summary？
3. 在金融行业 Agent 里，如果从“拒绝授信方案”切到“补充材料方案”，Branch Summary 应该保留哪些信息？

通过后再执行：

```text
1. 将 PROGRESS.md 中 0005 标记为已完成。
2. 写入 learning-records/<next-id>-session-tree-is-runtime-history-tree.md。
3. 按 study/blogs/WRITING-GUIDELINES.md 创建 0005 博文草稿。
4. 进入 0006：Extension System。
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
```

## 新会话续接方式

新会话中请先阅读：

1. `study/README.md`
2. `study/MASTER-PLAN.md`
3. `study/MISSION.md`
4. `study/CURRENT.md`
5. `study/PROGRESS.md`
6. `study/handoffs/0005-session-tree-and-compaction.md`

然后先完成第五课理解检查。理解检查通过后，再写 learning-record 和博客草稿。
