# Handoff：0005 Session Tree 与 Compaction

## 本节课状态

- 状态：进行中，已根据用户反馈重新组织课程讲法，待用户完成理解检查
- 日期：2026-06-25
- 对应课程：`study/course-design/0005-session-tree-and-compaction.md`
- 对应课件：`study/lessons/0005-session-tree-and-compaction.html`
- 对应参考资料：`study/reference/session-tree-and-compaction.md`

## 新会话必须读取的教学协议

新会话继续本课前，必须读取：

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

## 重组后的课程主线

新版本不再按“源码点罗列”讲，而按以下教学路径讲：

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
14. 理解检查
```

## 本节课目标

理解 Pi 的 Session 为什么不是简单聊天记录，而是包含 message、model_change、thinking_level_change、active_tools_change、compaction、branch_summary、custom、custom_message、label、session_info、leaf 等节点的运行历史树；理解 Compaction 和 Branch Summary 如何服务长上下文与分支会话。

## 本课核心结论

```text
Session Tree 保存“发生过什么、现在在哪条路径、下一轮该带什么上下文”；
Compaction 负责把旧路径折叠成检查点；
Branch Summary 负责把离开的分支变成移交说明。
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

## 重组后的关键理解点

1. `messages[]` 只能保存对话；Session Tree 保存 Runtime History。
2. `id` / `parentId` 负责历史结构，`leaf` 负责当前 active branch。
3. `buildSessionContext()` 是从完整历史树到 LLM 当前上下文视图的转换器。
4. Compaction 不删除 JSONL 旧 entry，而是用 summary + `firstKeptEntryId` 改写 LLM 看到的上下文视图。
5. `firstKeptEntryId` 是近期原文保留区的起点，不能只保存 summary。
6. `toolResult` 不能作为 cut point，因为它必须和 assistant toolCall 成对理解。
7. Split turn 用于单轮任务本身过长的场景：摘要 turn prefix，保留 turn suffix。
8. Branch Summary 是分支切换时的移交说明，不是长上下文压缩。
9. Compaction / Branch Summary 都属于 AgentHarness 层，不属于 AgentLoop 层。

## 待理解检查

用户需要用自己的话回答：

1. 为什么 Pi 的 Session 不能只设计成 `messages[]`？至少说出两个工程原因。
2. Compaction 为什么要保存 `firstKeptEntryId`，而不是只保存一个 summary？
3. 在金融行业 Agent 里，如果从“拒绝授信方案”切到“补充材料方案”，Branch Summary 应该保留哪些信息？

## 文档更新情况

已重写 / 更新：

```text
study/reference/session-tree-and-compaction.md
study/lessons/0005-session-tree-and-compaction.html
study/handoffs/0005-session-tree-and-compaction.md
study/CURRENT.md
study/TEACHING-PROTOCOL.md
study/HANDOFF-PROTOCOL.md
study/LEARNING-WORKFLOW.md
```

暂未写入：

```text
study/learning-records/<next-id>-*.md
study/blogs/drafts/0005-session-tree-and-compaction.md
```

原因：根据学习规则，learning-record 需要用户通过理解检查后再写；博客草稿应在完整知识点通过理解检查后再按写作规范沉淀。

## 下节课计划

理解检查通过后：

1. 将 `0005` 标记为已完成。
2. 新增 learning-record，记录用户已掌握 Session Tree / Compaction / Branch Summary 的职责边界。
3. 按 `study/blogs/WRITING-GUIDELINES.md` 创建博文草稿，企业类比继续统一使用金融行业 Agent。
4. 进入 `0006：Extension System`。

## 新会话续接提示

如果新开会话，请说：

```text
继续 Pi Agent Harness 学习。请读取 study/README.md、study/MASTER-PLAN.md、study/TEACHING-PROTOCOL.md、study/CURRENT.md、study/PROGRESS.md 和最新 handoff study/handoffs/0005-session-tree-and-compaction.md。0005 已根据用户反馈重组讲法但仍待理解检查，请先让我回答三个检查问题；通过后再写 learning-record 和 0005 博文草稿，然后进入 0006 Extension System。
```

即使用户使用更短提示词，新会话也必须主动读取 `study/TEACHING-PROTOCOL.md` 并按其中节奏继续。