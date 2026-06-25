# Session Tree 与 Compaction

> 状态：0005 已讲解，待理解检查后升级为已完成参考资料。  
> 核心句：Pi 的 Session 不是聊天记录数组，而是 Agent Runtime 的“可导航运行历史树”；Compaction 不是删除历史，而是把旧上下文折叠成可继续推理的摘要节点。

## 1. 总览图

```text
User / CLI / UI
    |
    v
AgentHarness
    |
    | 读取当前 leaf
    v
Session Tree (JSONL append-only)
    |
    | getBranch(leafId): leaf -> root -> chronological path
    v
buildSessionContext(pathEntries)
    |
    | 输出 messages + model + thinkingLevel + activeTools
    v
AgentLoop / LLM Context

长上下文治理：

Session Tree path
    |
    | estimate tokens / choose cut point
    v
CompactionEntry(summary, firstKeptEntryId, tokensBefore)
    |
    | 下次 build context: summary + kept recent messages
    v
LLM can continue

分支治理：

old leaf ---- abandoned branch
    |
    | navigateTree(targetId, summarize=true)
    v
BranchSummaryEntry(summary of abandoned branch)
    |
    | 接到新路径上作为上下文
    v
new branch can understand what was explored before
```

## 2. 源码锚点

| 主题 | 源码锚点 | 关键理解 |
|---|---|---|
| Session entry 类型 | `packages/agent/src/harness/types.ts`：`SessionTreeEntryBase`、`MessageEntry`、`CompactionEntry`、`BranchSummaryEntry`、`LeafEntry`、`SessionTreeEntry` | Session 是多种运行历史节点的联合，不只是 message。 |
| 当前路径重建上下文 | `packages/agent/src/harness/session/session.ts`：`buildSessionContext()` | 从当前 branch 提炼 LLM messages、model、thinkingLevel、activeTools。 |
| JSONL 持久化 | `packages/agent/src/harness/session/jsonl-storage.ts` | 每个 entry 追加到 JSONL；`id`/`parentId` 形成树；`leaf` 记录当前指针。 |
| Session repo / fork | `packages/agent/src/harness/session/jsonl-repo.ts` | session 文件按 cwd 隔离，可 fork 到新 session 文件。 |
| Compaction 触发与切点 | `packages/agent/src/harness/compaction/compaction.ts`：`shouldCompact()`、`findCutPoint()`、`prepareCompaction()` | 超上下文阈值后选择保留 recent tokens 的安全切点。 |
| Branch summary | `packages/agent/src/harness/compaction/branch-summarization.ts` | 切换树分支前，摘要将离开的 branch，避免丢失探索结论。 |
| Harness 调用入口 | `packages/agent/src/harness/agent-harness.ts`：`compact()`、`navigateTree()` | Compaction / Branch Summary 是 Harness Runtime 行为，不是 AgentLoop 行为。 |

## 3. Session Entry 数据模型图

```text
SessionTreeEntryBase
  - type: string
  - id: string
  - parentId: string | null
  - timestamp: string

SessionTreeEntry =
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

| Entry 类型 | 是否进 LLM Context | 作用 |
|---|---:|---|
| `message` | 是 | 用户、助手、工具结果、bash 等核心对话消息。 |
| `custom_message` | 是 | Extension 注入给模型看的上下文。 |
| `branch_summary` | 是 | 从别的 branch 带回来的摘要上下文。 |
| `compaction` | 间接是 | 自身不是普通 message，但会被转换为 `compactionSummary` 放入上下文。 |
| `model_change` | 否 | 恢复当前 branch 的模型选择。 |
| `thinking_level_change` | 否 | 恢复当前 branch 的推理强度。 |
| `active_tools_change` | 否 | 恢复当前 branch 的可用工具集合。 |
| `custom` | 否 | Extension 私有状态，不给模型。 |
| `label` | 否 | 对某个 entry 做书签或命名。 |
| `session_info` | 否 | session 展示名等元信息。 |
| `leaf` | 否 | 持久化“当前光标指向哪里”。 |

## 4. 为什么是 Tree，不是数组

| 如果用数组 | Pi 用 Tree 的原因 |
|---|---|
| 只能顺序追加，回到旧点通常要复制或截断。 | `id`/`parentId` 能从任意 entry 分叉，不破坏旧历史。 |
| 无法表达“当前正在看哪条路径”。 | `leaf` 指针明确当前 active branch。 |
| 只能保存 messages，难保存模型/工具/思考等级变化。 | entry union 可以把运行时状态变化也挂到历史树上。 |
| 切换分支后，离开的探索上下文容易丢。 | `branch_summary` 把离开的分支压成上下文接到新分支。 |
| 长会话只能删历史或截断。 | `compaction` 用 summary + firstKeptEntryId 保留可继续推理的上下文。 |

## 5. `buildSessionContext()` 的职责

```text
input: 当前 branch 上的 pathEntries

for entry in pathEntries:
  if thinking_level_change -> 更新 thinkingLevel
  if model_change -> 更新 model
  if assistant message -> 也可恢复 message.provider / message.model
  if active_tools_change -> 更新 activeToolNames
  if compaction -> 记录最后一个 compaction

如果存在 compaction:
  1. 先插入 compactionSummary(summary, tokensBefore)
  2. 找到 firstKeptEntryId
  3. 追加 firstKeptEntryId 到 compaction 前的保留消息
  4. 追加 compaction 之后的新消息
否则:
  直接追加路径上的 message / custom_message / branch_summary

output:
  SessionContext {
    messages,
    thinkingLevel,
    model,
    activeToolNames
  }
```

关键点：`compaction` 不删除 JSONL 里的旧 entry。它改变的是“下次给 LLM 的上下文视图”。

## 6. Compaction 流程图

```text
AgentHarness.compact()
    |
    v
session.getBranch()
    |
    v
prepareCompaction(pathEntries, settings)
    |
    |-- estimateContextTokens(buildSessionContext(pathEntries).messages)
    |-- find previous compaction
    |-- findCutPoint(keepRecentTokens)
    |-- collect messagesToSummarize
    |-- collect turnPrefixMessages if split turn
    |-- extract fileOps
    v
session_before_compact hook
    |
    |-- cancel? stop
    |-- custom compaction? use hook result
    v
compact(preparation, models, model)
    |
    |-- generateSummary()
    |-- if split turn: generateTurnPrefixSummary()
    |-- append file operation list
    v
session.appendCompaction(summary, firstKeptEntryId, tokensBefore, details)
    |
    v
emit session_compact
```

## 7. Compaction 切点规则

| 规则 | 原因 |
|---|---|
| 默认保留最近 `keepRecentTokens` 的上下文。 | 旧内容可以摘要，最近工作最好原样保留。 |
| `reserveTokens` 用于给摘要 prompt 和模型输出留空间。 | 不能把 context window 塞满，否则模型没空间回答。 |
| 不切在 `toolResult` 上。 | tool result 必须和触发它的 tool call 保持语义配对。 |
| 如果一个 turn 太大，允许 split turn。 | 单轮超长时只能摘要 turn prefix，保留后缀继续工作。 |
| 重复 compaction 会携带 previousSummary 更新。 | 防止多次压缩后丢掉更早历史。 |

## 8. Branch Summary 流程图

```text
AgentHarness.navigateTree(targetId, summarize=true)
    |
    v
oldLeafId = session.getLeafId()
    |
    v
collectEntriesForBranchSummary(session, oldLeafId, targetId)
    |
    |-- oldPath = path(oldLeafId)
    |-- targetPath = path(targetId)
    |-- commonAncestor = deepest shared entry
    |-- entriesToSummarize = old leaf back to common ancestor
    v
session_before_tree hook
    |
    |-- cancel? stop navigation
    |-- custom summary? use hook result
    v
optional generateBranchSummary(entries)
    |
    v
session.moveTo(newLeafId, summary?)
    |
    |-- append leaf entry
    |-- if summary: append branch_summary at navigation point
    v
emit session_tree
```

## 9. Compaction vs Branch Summary

| 维度 | Compaction | Branch Summary |
|---|---|---|
| 解决问题 | 上下文窗口不够。 | 切换分支时，离开的探索路径不丢。 |
| 触发 | 自动阈值或 `/compact` / Harness `compact()`。 | `/tree` / Harness `navigateTree()`。 |
| 摘要对象 | 当前 branch 的旧历史。 | 从 old leaf 到 common ancestor 的 abandoned branch。 |
| 写入 entry | `compaction` | `branch_summary` |
| 下次上下文 | `compactionSummary + firstKeptEntryId 之后消息` | branch summary 作为 message 进入新路径上下文。 |
| 企业类比 | 风控 Agent 把旧审批讨论压缩成“审批检查点”。 | 信贷 Agent 从“拒绝方案”切到“补资料方案”时，保留刚才拒绝方案的关键发现。 |

## 10. 金融行业 Agent 类比

设想一个“信贷审批 Agent”：

```text
客户申请材料 -> Agent 分析征信 -> 调用规则工具 -> 调用额度测算工具
              -> 发现材料缺失 -> 分支 A：拒绝
                                -> 分支 B：补充材料后重评
```

- 如果只是 messages 数组，分支 A / B 的探索路径会互相覆盖或需要复制会话。
- 如果用 Session Tree，分支 A 和 B 都能挂在同一个共同祖先后面。
- 如果会话很长，Compaction 把旧审批背景压成 summary，但保留最近工具结果和模型判断。
- 如果从分支 A 切回分支 B，Branch Summary 可以把“分支 A 已经确认的风险点”带回新路径。

企业级启发：

```text
Session Tree = 审批过程审计树
Compaction = 长审批链路检查点摘要
Branch Summary = 方案切换时的分支移交说明
Leaf = 当前正在推进的审批路径
```

## 11. 本课理解检查

请用自己的话回答：

1. 为什么 Pi 的 Session 不能只设计成 `messages[]`？至少说出两个工程原因。
2. Compaction 为什么要保存 `firstKeptEntryId`，而不是只保存一个 summary？
3. 在金融行业 Agent 里，如果从“拒绝授信方案”切到“补充材料方案”，Branch Summary 应该保留哪些信息？
