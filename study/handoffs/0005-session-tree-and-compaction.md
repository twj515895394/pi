# Handoff：0005 Session Tree 与 Compaction

## 本节课状态

- 状态：进行中，已完成源码阅读与课程讲解，待用户完成理解检查
- 日期：2026-06-25
- 对应课程：`study/course-design/0005-session-tree-and-compaction.md`
- 对应课件：`study/lessons/0005-session-tree-and-compaction.html`
- 对应参考资料：`study/reference/session-tree-and-compaction.md`

## 本节课目标

理解 Pi 的 Session 为什么不是简单聊天记录，而是包含 message、model_change、thinking_level_change、active_tools_change、compaction、branch_summary、custom、custom_message、label、session_info、leaf 等节点的运行历史树；理解 Compaction 和 Branch Summary 如何服务长上下文与分支会话。

## 实际学习内容

本节课已讲解以下内容：

1. Session Tree 总览：`id` / `parentId` 形成树，`leaf` 表示当前 active branch。
2. `SessionTreeEntry` 类型联合：message、runtime 状态变化、compaction、branch_summary、extension 状态、label、leaf。
3. JSONL 存储模型：append-only session 文件；entry 追加时更新 current leaf；`setLeafId()` 通过 leaf entry 持久化当前指针。
4. `buildSessionContext()`：从当前 branch 还原 LLM messages、thinkingLevel、model、activeToolNames。
5. Compaction：通过 `prepareCompaction()` 找切点、保留 recent messages、生成 summary、写入 `CompactionEntry`。
6. Cut point 规则：不能切在 toolResult 上；必要时支持 split turn。
7. Repeated compaction：通过 previousSummary 和 firstKeptEntryId 迭代更新历史摘要。
8. Branch Summary：`navigateTree()` 切换 branch 时，摘要 old leaf 到 common ancestor 的 abandoned branch。
9. Harness 扩展点：`session_before_compact`、`session_compact`、`session_before_tree`、`session_tree`。
10. 金融行业 Agent 类比：信贷审批 Agent 中的审批过程审计树、检查点摘要、分支移交说明。

## 本课核心结论

```text
Session Tree = Agent Runtime 的可导航运行历史树
Compaction = 长上下文检查点摘要
Branch Summary = 分支切换移交说明
Leaf = 当前 active branch 指针
```

## 源码阅读进度

已阅读：

```text
study/README.md
study/MASTER-PLAN.md
study/MISSION.md
study/CURRENT.md
study/PROGRESS.md
study/handoffs/0004-agent-harness-runtime-control.md
study/course-design/0005-session-tree-and-compaction.md
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

## 待理解检查

用户需要用自己的话回答：

1. 为什么 Pi 的 Session 不能只设计成 `messages[]`？至少说出两个工程原因。
2. Compaction 为什么要保存 `firstKeptEntryId`，而不是只保存一个 summary？
3. 在金融行业 Agent 里，如果从“拒绝授信方案”切到“补充材料方案”，Branch Summary 应该保留哪些信息？

## 文档更新情况

已新增：

```text
study/lessons/0005-session-tree-and-compaction.html
study/handoffs/0005-session-tree-and-compaction.md
```

已更新：

```text
study/reference/session-tree-and-compaction.md
study/PROGRESS.md
study/CURRENT.md
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
继续 Pi Agent Harness 学习。请读取 study/CURRENT.md、study/PROGRESS.md 和最新 handoff study/handoffs/0005-session-tree-and-compaction.md。0005 已讲解但待理解检查，请先让我回答三个检查问题；通过后再写 learning-record 和 0005 博文草稿，然后进入 0006 Extension System。
```
