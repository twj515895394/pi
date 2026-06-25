# Session Tree 与 Compaction

> 状态：0005 已重组讲解，待理解检查后升级为已完成参考资料。  
> 可记忆核心句：**Session Tree 保存“发生过什么、现在在哪条路径、下一轮该带什么上下文”；Compaction 负责把旧路径折叠成检查点；Branch Summary 负责把离开的分支变成移交说明。**

## 0. 本课先建立一个心智模型

不要先把 Session 理解成“聊天记录表”。在 Pi 里，它更像一个 **Agent Runtime 的运行账本 + 历史版本树 + 上下文生成器**。

```text
普通聊天系统：
messages[]
  只回答：用户和助手说过什么？

Pi Session Tree：
entries(id, parentId, type, payload)
  回答：
  1. 这条 Agent 运行路径上发生过什么？
  2. 当前 active leaf 在哪里？
  3. 当前路径应该恢复哪个 model / thinkingLevel / activeTools？
  4. 给下一轮 LLM 的上下文应该是哪些消息？
  5. 历史太长时，哪些内容被 summary 代替，哪些 recent messages 必须原样保留？
  6. 从一个 branch 跳到另一个 branch 时，离开的 branch 有哪些结论要带走？
```

## 1. 先看总览：三条运行链路

```text
┌────────────────────────────────────────────────────────────────────┐
│                         AgentHarness                               │
│                                                                    │
│  普通下一轮：                                                        │
│  session.getBranch(leaf) -> buildSessionContext() -> AgentLoop       │
│                                                                    │
│  长上下文：                                                          │
│  prepareCompaction() -> compact() -> appendCompaction()              │
│                                                                    │
│  分支切换：                                                          │
│  collectEntriesForBranchSummary() -> generateBranchSummary()         │
│       -> session.moveTo(newLeaf, summary?)                           │
└────────────────────────────────────────────────────────────────────┘
```

这三条链路说明一个关键边界：

```text
AgentLoop 只关心“当前这轮 messages 如何推理和调用工具”；
AgentHarness 负责“长期运行历史如何保存、恢复、压缩、切换和审计”。
```

## 2. 职责边界图

```text
UI / CLI / SDK
  - 发起 prompt / compact / navigateTree
  - 展示 message、tool、tree、summary 事件
        |
        v
AgentHarness
  - 决定当前 phase: idle / turn / compaction / branch_summary
  - 读取 Session branch
  - 创建 TurnState
  - 调用 AgentLoop
  - 触发 session_before_compact / session_before_tree hook
  - 写入 compaction / branch_summary / leaf
        |
        v
Session
  - appendMessage / appendModelChange / appendCompaction
  - getBranch(leafId)
  - buildContext()
  - moveTo(entryId, summary?)
        |
        v
SessionStorage / JsonlSessionStorage
  - JSONL append-only
  - byId map
  - currentLeafId
  - getPathToRoot()
        |
        v
JSONL file
  - header
  - SessionTreeEntry lines
```

职责边界：

| 层 | 应该负责 | 不应该负责 |
|---|---|---|
| AgentLoop | 当前 run 的模型请求、工具调用、toolResult 回灌 | 长期会话树、分支导航、上下文压缩 |
| AgentHarness | Runtime 编排、Session 读写、Compaction、Branch Summary、Hook、事件 | 具体 JSONL 读写细节 |
| Session | 面向 Harness 的会话操作 API：append、moveTo、getBranch、buildContext | 模型调用、摘要模型选择 |
| JsonlSessionStorage | append-only 持久化、leaf 指针、path 回溯 | 判断什么内容该进 LLM |
| Compaction 模块 | 切点选择、摘要生成、fileOps 汇总 | UI 展示、用户交互 |

## 3. 数据模型：为什么是 Entry Tree

源码中 `SessionTreeEntryBase` 有四个基础字段：

```text
SessionTreeEntryBase
  - type
  - id
  - parentId
  - timestamp
```

所有节点都挂在这套树结构上：

```text
SessionTreeEntry =
  message                  // 对话消息
  thinking_level_change    // 推理等级变化
  model_change             // 模型变化
  active_tools_change      // 可用工具变化
  compaction               // 长上下文摘要检查点
  branch_summary           // 分支切换摘要
  custom                   // Extension 私有状态
  custom_message           // Extension 注入上下文
  label                    // 用户标记
  session_info             // 会话展示信息
  leaf                     // 当前指针变化记录
```

这就是它不是 `messages[]` 的第一层原因：**它要记录的是 Agent Runtime 历史，而不只是自然语言对话。**

| Entry 类型 | 是否进入 LLM Context | Runtime 价值 |
|---|---:|---|
| `message` | 是 | 用户输入、assistant 输出、tool result、bash 结果等。 |
| `custom_message` | 是 | Extension 注入给模型的上下文。 |
| `branch_summary` | 是 | 把离开的分支摘要成可继续参考的上下文。 |
| `compaction` | 间接是 | 转成 `compactionSummary`，替代旧历史。 |
| `model_change` | 否 | 让恢复后的 branch 知道当时用什么模型。 |
| `thinking_level_change` | 否 | 让恢复后的 branch 知道当时推理强度。 |
| `active_tools_change` | 否 | 让恢复后的 branch 知道哪些工具可用。 |
| `custom` | 否 | Extension 自己的持久化状态，不暴露给模型。 |
| `label` | 否 | bookmark / checkpoint。 |
| `session_info` | 否 | 会话名称等展示信息。 |
| `leaf` | 否 | 记录当前 active branch 指针变化。 |

## 4. 普通链路：Session Tree 如何变成 LLM Context

`buildSessionContext(pathEntries)` 是本课最关键的函数之一。

```text
输入：当前 leaf 到 root 的路径 entries

第一遍扫描：
  - 遇到 thinking_level_change -> 更新 thinkingLevel
  - 遇到 model_change -> 更新 model
  - 遇到 assistant message -> 可从 message.provider / message.model 恢复 model
  - 遇到 active_tools_change -> 更新 activeToolNames
  - 遇到 compaction -> 记录最后一个 compaction

第二步生成 messages：
  如果没有 compaction：
    append message / custom_message / branch_summary

  如果有 compaction：
    先放 compactionSummary
    再从 firstKeptEntryId 开始保留消息
    再放 compaction 后面的新消息

输出：
  SessionContext {
    messages,
    thinkingLevel,
    model,
    activeToolNames
  }
```

这个函数的工程意义：

```text
Session Tree 是完整历史；
buildSessionContext 是“给模型看的当前视图”。
```

这也是理解 Compaction 的关键：**旧 entry 还在 JSONL 文件里，但不一定继续原样塞给 LLM。**

## 5. JSONL 存储：为什么 append-only 很重要

`JsonlSessionStorage` 的核心动作是：

```text
appendEntry(entry):
  append JSON line to file
  entries.push(entry)
  byId.set(entry.id, entry)
  currentLeafId = entry.type === "leaf" ? entry.targetId : entry.id

setLeafId(leafId):
  append a leaf entry
  currentLeafId = leafId

getPathToRoot(leafId):
  current = byId.get(leafId)
  while current:
    path.unshift(current)
    current = byId.get(current.parentId)
```

append-only 的价值：

| 价值 | 说明 |
|---|---|
| 可审计 | 不需要覆盖旧历史，历史路径可追溯。 |
| 可分支 | 新节点只需要指向旧 parentId，不破坏原路径。 |
| 可恢复 | leaf entry 可以恢复“当前在哪条分支”。 |
| 可扩展 | Extension 可以追加 custom / custom_message，不侵入核心 message。 |

## 6. 为什么不是 `messages[]`：五个工程原因

| 工程需求 | `messages[]` 的问题 | Session Tree 的解法 |
|---|---|---|
| 分支探索 | 回到旧点会截断或复制数组 | `id` / `parentId` 从任意 entry 开新分支 |
| 当前指针 | 不知道当前 active branch | `leaf` 持久化当前指针 |
| Runtime 恢复 | 只能恢复文本，难恢复模型/工具/推理等级 | `model_change` / `active_tools_change` / `thinking_level_change` 也是 entry |
| 长上下文 | 只能粗暴截断旧消息 | `compaction` = summary + firstKeptEntryId |
| 分支切换 | 离开的路径结论丢失 | `branch_summary` 把 abandoned branch 移交给新路径 |

## 7. Compaction：不是“压缩字符串”，而是“上下文视图重写”

触发条件可以抽象成：

```text
contextTokens > contextWindow - reserveTokens
```

Compaction 的真实目标不是让文件变小，而是让下一轮 LLM 看到：

```text
systemPrompt
+ compactionSummary(旧历史摘要)
+ kept recent messages(近期原文)
+ new messages after compaction
```

### 7.1 Compaction 流程图

```text
AgentHarness.compact()
  |
  v
session.getBranch()
  |
  v
prepareCompaction(pathEntries, settings)
  |-- estimateContextTokens(buildSessionContext(pathEntries).messages)
  |-- 找到上一次 compaction
  |-- findCutPoint(keepRecentTokens)
  |-- messagesToSummarize = 旧历史
  |-- turnPrefixMessages = 被 split 的单轮前缀
  |-- fileOps = 读写文件轨迹
  v
session_before_compact hook
  |-- 可取消
  |-- 可提供自定义 summary
  v
compact(preparation, models, model)
  |-- generateSummary()
  |-- split turn 时 generateTurnPrefixSummary()
  |-- 追加 readFiles / modifiedFiles
  v
session.appendCompaction(summary, firstKeptEntryId, tokensBefore, details)
  |
  v
session_compact event
```

### 7.2 `firstKeptEntryId` 为什么重要

如果只保存 summary，会出现两个问题：

```text
问题 1：模型失去近期原始证据
  旧历史可以摘要，但最近工具结果、最近 assistant 判断、最近用户要求最好原样保留。

问题 2：下一次 buildSessionContext 不知道从哪里开始拼接 recent messages
  summary 只能代表“过去”，firstKeptEntryId 才是“原文保留区”的起点。
```

所以 `CompactionEntry` 的核心不是 `summary` 一个字段，而是：

```text
summary            = 被折叠历史的语义检查点
firstKeptEntryId   = 保留原文上下文的起点
tokensBefore       = 压缩前上下文规模
details            = readFiles / modifiedFiles 等实现细节
```

## 8. Cut Point：为什么不能切在 toolResult 上

```text
assistant message:
  toolCall(id="risk_check_001", name="riskRuleCheck")

下一条 toolResult:
  toolCallId="risk_check_001"
  content="命中规则 R-17，需要补充收入证明"
```

如果切点落在 `toolResult` 上，模型可能看到：

```text
一个孤立的 toolResult，但不知道它对应哪个 toolCall、为什么调用、参数是什么。
```

所以 Pi 的切点规则会跳过 `toolResult`。这不是小细节，而是 **Agent 工具语义完整性** 的要求。

| 可作为切点 | 不作为切点 |
|---|---|
| user message | toolResult |
| assistant message | 纯 runtime state entry |
| bashExecution | label / leaf |
| custom_message / branch_summary | session_info |

## 9. Split Turn：当单轮太大时怎么办

正常情况下，Compaction 最好切在 turn 边界：

```text
user -> assistant -> toolResult -> assistant
```

但如果一轮任务本身就超过 `keepRecentTokens`，例如金融 Agent 一次性读了大量材料、跑了多个风险工具，那就只能：

```text
turn prefix 被摘要
turn suffix 原样保留
```

```text
一个超长 turn：
[user 原始需求] -> [assistant 早期分析] -> [toolResult 大量输出] -> [assistant 最近判断]
        |------------------ turnPrefixMessages ------------------| |-- kept suffix --|
```

这样做的目标是：

```text
保留最近执行现场，同时不让模型忘记这轮最开始的用户目标和早期决策。
```

## 10. Branch Summary：不是压缩历史，而是分支移交

Branch Summary 解决的是另一个问题：**当你从一条分支跳到另一条分支时，刚刚离开的那条路径有什么发现需要带过去？**

```text
Before:

        B -> C -> D   old leaf: 拒绝授信方案
      /
A ---
      
        E -> F       target: 补充材料方案

commonAncestor = A
entriesToSummarize = B,C,D

After:

        B -> C -> D   abandoned branch
      /
A ---
      
        E -> F -> branch_summary(B,C,D)   new context
```

### 10.1 Branch Summary 流程

```text
AgentHarness.navigateTree(targetId, summarize=true)
  |
  v
oldLeafId = session.getLeafId()
  |
  v
collectEntriesForBranchSummary(session, oldLeafId, targetId)
  |-- oldPath = getBranch(oldLeafId)
  |-- targetPath = getBranch(targetId)
  |-- commonAncestor = deepest shared entry
  |-- entriesToSummarize = old leaf 回溯到 commonAncestor
  v
session_before_tree hook
  |-- 可取消导航
  |-- 可提供自定义 summary
  v
generateBranchSummary(entries)
  |
  v
session.moveTo(newLeafId, summary?)
  |-- append leaf
  |-- append branch_summary when summary exists
  v
session_tree event
```

### 10.2 Compaction vs Branch Summary

| 维度 | Compaction | Branch Summary |
|---|---|---|
| 解决问题 | 上下文窗口不够 | 切换分支时丢失离开路径的探索结论 |
| 摘要对象 | 当前 branch 的旧历史 | old leaf 到 common ancestor 的 abandoned branch |
| 写入节点 | `compaction` | `branch_summary` |
| 进入上下文方式 | 转成 `compactionSummary` + 保留 recent messages | 转成 `branchSummary` message 接入新路径 |
| 关键词 | 检查点 | 移交说明 |

## 11. 企业金融 Agent 回填

以“信贷审批 Agent”为泛化例子：

```text
A: 用户要求评估一笔信贷申请
  -> B: Agent 读取申请材料
  -> C: 调用征信摘要工具
  -> D: 调用风险规则工具
  -> E: 分支 1：拒绝授信方案
  -> F: 分支 2：补充材料后重评方案
```

### 11.1 如果用 `messages[]`

```text
要么覆盖拒绝方案路径；
要么复制整段消息生成另一份会话；
要么靠人工总结刚才发生了什么。
```

### 11.2 如果用 Session Tree

```text
A -> B -> C -> D
              |-> E 拒绝授信方案
              |-> F 补充材料方案
```

每个方案都是同一棵树上的 branch，审计、恢复和对比都更自然。

### 11.3 Branch Summary 应该保留什么

从“拒绝授信方案”切到“补充材料方案”时，Branch Summary 不应该写成泛泛总结，而应保留：

| 信息 | 示例 |
|---|---|
| 分支目标 | 刚才探索的是拒绝授信方案。 |
| 关键证据 | 命中了哪些风险信号、缺了哪些材料。 |
| 工具结论 | 征信摘要、规则工具、额度测算工具给出的关键结果。 |
| 已做判断 | 为什么拒绝方案成立，哪些点仍不确定。 |
| 可复用上下文 | 补充材料方案需要继续关注哪些风险点。 |
| 文件 / 资料轨迹 | 哪些材料被读取、哪些分析结果被修改。 |

## 12. Java / Dify 经验迁移

| 你已有经验 | Pi 这一课对应的底层设计 |
|---|---|
| Dify 会话历史 | Pi Session Tree 更强调可分支、可恢复、可压缩的 Runtime history。 |
| Java 审计日志表 | JSONL Session 像 append-only 审计日志，但多了 `parentId` 树结构。 |
| 工作流节点状态 | `model_change` / `active_tools_change` / `custom` 类似运行状态事件。 |
| 长上下文总结节点 | Compaction 是系统级上下文视图重写，不只是让模型“总结一下”。 |
| 人工方案切换记录 | Branch Summary 是分支跳转时的自动移交说明。 |

## 13. 小结卡片

```text
1. Session Tree 不是 messages[]，而是 Runtime History Tree。
2. id / parentId 负责历史结构，leaf 负责当前位置。
3. buildSessionContext 把完整历史树转换成“当前给 LLM 看的上下文视图”。
4. Compaction 不删历史，只把旧历史换成 summary，并从 firstKeptEntryId 保留近期原文。
5. toolResult 不能作为切点，因为它必须和 toolCall 成对理解。
6. Branch Summary 是分支切换时的上下文移交，不是长上下文压缩。
7. 这些能力都属于 Harness 层，不属于 AgentLoop 层。
```

## 14. 本课理解检查

请用自己的话回答：

1. 为什么 Pi 的 Session 不能只设计成 `messages[]`？至少说出两个工程原因。
2. Compaction 为什么要保存 `firstKeptEntryId`，而不是只保存一个 summary？
3. 在金融行业 Agent 里，如果从“拒绝授信方案”切到“补充材料方案”，Branch Summary 应该保留哪些信息？
