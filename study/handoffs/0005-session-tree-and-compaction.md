# Handoff：0005 Session Tree 与 Compaction

## 本节课状态

- 状态：已完成
- 日期：2026-06-25
- 对应课程：`study/course-design/0005-session-tree-and-compaction.md`
- 对应课件：`study/lessons/0005-session-tree-and-compaction.html`
- 对应参考资料：`study/reference/session-tree-and-compaction.md`
- 学习记录：`study/learning-records/0005-session-tree-is-runtime-history-tree.md`
- 博文草稿：`study/blogs/drafts/0005-session-tree-and-compaction.md`

## 新会话必须读取的教学协议

新会话继续后续课程前，必须读取：

```text
study/TEACHING-PROTOCOL.md
```

原因：用户反馈过短提示词会导致新会话讲课风格丢失。后续不能只读取 `CURRENT.md` 和本 handoff 后简单输出摘要，必须按教学协议恢复固定讲课节奏：总览图、分层表格、流程图、源码锚点、金融行业 Agent 类比、理解检查和课程收尾。

## 用户反馈与修正

用户指出第一版第五课“不够好”，没有充分遵守学习偏好。

已按 `study/NOTES.md` 和 `study/TEACHING-PROTOCOL.md` 中的偏好重新组织：

```text
中文为主
源码 + 调用链 + 工程场景
结构化输出：分层表格、流程图、职责边界图、设计图、小结卡片
高概念密度内容先总览，再分模块拆解，最后用企业场景回填
企业案例统一使用泛化金融行业 Agent
```

## 完成后的课程主线

```text
0. 先建立心智模型：Session 不是聊天记录表，而是 Runtime History Tree
1. 三条运行链路总览：普通下一轮 / 长上下文 / 分支切换
2. 职责边界图：UI/CLI -> AgentHarness -> Session -> Storage -> JSONL
3. 数据模型：SessionTreeEntry 类型联合
4. 普通链路：buildSessionContext 如何把树变成 LLM Context
5. 为什么不是 messages[]：五个工程原因
6. Compaction：上下文视图重写，而不是删除历史
7. firstKeptEntryId：summary 与 recent messages 的边界
8. Cut Point / Split Turn：为什么不能切 toolResult
9. Branch Summary：分支切换时的移交说明
10. Compaction vs Branch Summary 对照
11. 金融行业 Agent 回填：信贷审批 Agent 的审计树、检查点摘要、分支移交说明
12. Java / Dify 经验迁移
13. 小结卡片
14. 理解检查与补讲
```

## 本课核心结论

```text
Session Tree 保存“发生过什么、现在在哪条路径、下一轮该带什么上下文”；
Compaction 负责把旧路径折叠成检查点；
Branch Summary 负责把离开的分支变成移交说明。
```

## 已确认理解

用户已通过补充理解检查，确认理解以下边界：

1. `messages[]` 只能保存对话；Session Tree 保存 Runtime History。
2. `id` / `parentId` 负责历史结构，`leaf` 负责当前 active branch。
3. `buildSessionContext()` 是从完整历史树到 LLM 当前上下文视图的转换器。
4. Compaction 不删除 JSONL 旧 entry，而是用 summary + `firstKeptEntryId` 改写 LLM 看到的上下文视图。
5. `firstKeptEntryId` 是近期原文保留区的起点，不能只保存 summary。
6. `toolResult` 不能作为 cut point，因为它必须和 assistant toolCall 成对理解。
7. Split turn 用于单轮任务本身过长的场景：摘要 turn prefix，保留 turn suffix。
8. Branch Summary 是分支切换时的移交说明，不是长上下文压缩。
9. Compaction / Branch Summary 都属于 AgentHarness 层，不属于 AgentLoop 层。
10. branch 是同一个 session tree 内的路径分叉；fork 才是创建新的 session 文件。

## 理解检查证据

用户最终回答：

```text
Compaction处理的是当前 branch 太长问题，不是切换当前branch。 所以我觉得不是一个东西
```

该回答说明用户已能区分：

```text
Compaction = 当前 branch 太长时的上下文治理
Branch Summary = 从 old branch 切到 new branch 时的路径移交
```

此前用户也已经能够说明：

```text
Branch Summary 摘要的是被切换的 branch 的内容，带到新的 branch 里去。
```

## 源码阅读进度

已阅读：

```text
study/README.md
study/MASTER-PLAN.md
study/TEACHING-PROTOCOL.md
study/MISSION.md
study/CURRENT.md
study/PROGRESS.md
study/handoffs/0004-agent-harness-runtime-control.md
study/course-design/0005-session-tree-and-compaction.md
study/NOTES.md
packages/agent/src/harness/types.ts
packages/agent/src/harness/session/session.ts
packages/agent/src/harness/session/jsonl-storage.ts
packages/agent/src/harness/session/jsonl-repo.ts
packages/agent/src/harness/compaction/compaction.ts
packages/agent/src/harness/compaction/branch-summarization.ts
packages/agent/src/harness/agent-harness.ts
packages/coding-agent/docs/session-format.md
packages/coding-agent/docs/compaction.md
study/blogs/WRITING-GUIDELINES.md
```

关键源码点：

```text
SessionTreeEntryBase
MessageEntry
CompactionEntry
BranchSummaryEntry
LeafEntry
SessionTreeEntry
SessionStorage
Session.getBranch()
Session.buildContext()
buildSessionContext()
JsonlSessionStorage.appendEntry()
JsonlSessionStorage.setLeafId()
JsonlSessionStorage.getPathToRoot()
JsonlSessionRepo.fork()
shouldCompact()
findCutPoint()
prepareCompaction()
compact()
generateSummary()
generateTurnPrefixSummary()
collectEntriesForBranchSummary()
prepareBranchEntries()
generateBranchSummary()
AgentHarness.compact()
AgentHarness.navigateTree()
session_before_compact
session_compact
session_before_tree
session_tree
```

## 文档更新情况

已更新：

```text
study/PROGRESS.md
study/CURRENT.md
study/handoffs/0005-session-tree-and-compaction.md
study/learning-records/0005-session-tree-is-runtime-history-tree.md
study/blogs/drafts/0005-session-tree-and-compaction.md
study/blogs/README.md
```

已生成 / 保留：

```text
study/reference/session-tree-and-compaction.md
study/lessons/0005-session-tree-and-compaction.html
```

## 下一节课

进入：

```text
0006：Extension System
```

建议新会话先读取：

```text
study/README.md
study/MASTER-PLAN.md
study/TEACHING-PROTOCOL.md
study/MISSION.md
study/CURRENT.md
study/PROGRESS.md
study/handoffs/0005-session-tree-and-compaction.md
```

然后按教学协议开始第六课。
