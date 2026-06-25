# 为什么 Agent 的 Session 不是聊天记录：Session Tree、Compaction 与 Branch Summary

## 开篇问题：为什么这个知识点重要？

我以前很容易把 Agent 的 Session 理解成一组 `messages[]`：用户说了什么、模型回了什么、工具返回了什么，按顺序存下来就行。

但读 Pi 的 Session / Compaction 这部分源码后，会发现这个理解太浅了。对于一个 Coding Agent 或企业级 Agent Harness 来说，Session 不只是聊天记录，而是 Runtime 的运行历史树。它要支持回退、分支、长上下文压缩、运行状态恢复，以及从一条探索路径切到另一条探索路径。

可记忆核心句：

```text
Session Tree 保存“发生过什么、现在在哪条路径、下一轮该带什么上下文”；
Compaction 负责把旧路径折叠成检查点；Branch Summary 负责把离开的分支变成移交说明。
```

## 一句话结论

Pi 的 Session 更像一棵 append-only 的运行历史树，而不是一个线性聊天数组；Compaction 和 Branch Summary 都建立在这棵树上，但解决的问题不同：前者处理当前 branch 太长，后者处理切换 branch 时如何带走旧路径的关键经验。

## 背景铺垫：普通 messages[] 能解决什么，不能解决什么

如果只是一个简单聊天机器人，`messages[]` 很自然：

```text
user -> assistant -> user -> assistant
```

它能回答：用户和助手说过什么？

但 Agent Runtime 需要回答更多问题：

```text
1. 当前运行路径上发生过什么？
2. 当前 active branch 在哪里？
3. 这条路径上用过哪个模型、哪些工具、什么推理等级？
4. 历史太长时，哪些内容应该摘要，哪些 recent messages 必须原样保留？
5. 从一条探索路径切到另一条路径时，旧路径的关键发现要不要带过去？
```

这就是 Session Tree 出现的原因。

## 核心概念：Session Tree 是 Runtime History Tree

Pi 的 Session 不是只保存 message，它保存的是多种 Runtime Entry：

```text
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

其中最关键的结构字段是：

```text
id
parentId
leaf
```

可以把它理解成：

```text
id / parentId 负责历史结构；
leaf 负责当前正在推进的路径；
buildSessionContext() 负责把当前路径转换成 LLM 能看到的上下文。
```

## 为什么不是 messages[]？

| 工程需求 | messages[] 的问题 | Session Tree 的解法 |
|---|---|---|
| 分支探索 | 回到旧点只能截断或复制数组 | `id` / `parentId` 从任意 entry 开新分支 |
| 当前指针 | 不知道当前 active branch | `leaf` 持久化当前位置 |
| Runtime 恢复 | 只能恢复文本 | 模型、工具、推理等级变化也是 entry |
| 长上下文 | 只能粗暴截断旧消息 | `compaction` = summary + `firstKeptEntryId` |
| 分支切换 | 离开的路径结论容易丢 | `branch_summary` 把 old branch 经验带到 new branch |

## 三条核心链路

```text
AgentHarness
  ├─ 普通下一轮：
  │   session.getBranch(leaf) -> buildSessionContext() -> AgentLoop
  │
  ├─ 长上下文：
  │   prepareCompaction() -> compact() -> appendCompaction()
  │
  └─ 分支切换：
      collectEntriesForBranchSummary() -> generateBranchSummary()
        -> session.moveTo(newLeaf, summary?)
```

这张图的核心边界是：

```text
AgentLoop 负责当前一轮模型-工具闭环；
AgentHarness 负责长期运行历史的保存、恢复、压缩、分支切换和审计。
```

## buildSessionContext：完整历史树到当前上下文视图

我对这一课最大的一个修正理解是：Session Tree 是完整历史，但 LLM 每次看到的是一个被构造出来的上下文视图。

简化伪代码：

```text
buildSessionContext(pathEntries):
  扫描 pathEntries
    更新 thinkingLevel / model / activeTools
    记录最后一个 compaction

  if 有 compaction:
    messages = [compactionSummary]
             + messages from firstKeptEntryId
             + messages after compaction
  else:
    messages = path 中的 message / custom_message / branch_summary
```

这说明：

```text
Compaction 不删除旧 JSONL entry；
它改变的是下一轮 LLM 看到的上下文视图。
```

## Compaction：处理当前 branch 太长

Compaction 解决的是上下文窗口问题。

它的目标不是让 session 文件变小，而是让下一轮模型看到：

```text
systemPrompt
+ compactionSummary(旧历史摘要)
+ kept recent messages(近期原文)
+ new messages after compaction
```

这里 `firstKeptEntryId` 很关键。

它不是“从哪里开始压缩”的游标，而是：

```text
压缩后仍然保留原文的起点。
```

也就是：

```text
firstKeptEntryId 之前：由 summary 表示
firstKeptEntryId 之后：recent messages 原样保留
```

## Branch Summary：处理切换 branch 时的路径移交

Branch Summary 解决的是另一个问题：从一条 branch 切到另一条 branch 时，被离开的那条路径有什么关键经验要带过去？

```text
Before:
        B -> C -> D   old branch
      /
A ---
      
        E -> F       target branch

commonAncestor = A
entriesToSummarize = B,C,D

After:
        B -> C -> D
      /
A ---
      
        E -> F -> branch_summary(B,C,D)
```

所以我现在会这样区分：

| 维度 | Compaction | Branch Summary |
|---|---|---|
| 解决问题 | 当前 branch 太长 | 从 old branch 切到 new branch 时，旧路径经验如何带过去 |
| 摘要对象 | 当前 branch 的旧历史 | old branch 相对 common ancestor 的新增路径 |
| 写入 entry | `compaction` | `branch_summary` |
| 关键词 | 检查点 | 移交说明 |

这两者不是一个东西。

## 工程类比：金融行业 Agent

以一个泛化的信贷审批 Agent 为例。

同一份申请材料下，Agent 可能探索多个方案：

```text
共同事实：读取申请材料 -> 调用征信工具 -> 调用风险规则工具

分支 A：拒绝授信方案
分支 B：补充材料后重评方案
分支 C：降低额度通过方案
分支 D：转人工复核方案
```

如果都塞在一条 messages[] 里，旧方案的判断可能污染新方案，审计上也很难看清当时探索过哪些路径。

如果用 Session Tree：

```text
共同事实
  ├─ 拒绝授信方案
  ├─ 补充材料方案
  ├─ 降低额度方案
  └─ 人工复核方案
```

每条路径都可以独立推进、回退、对比和审计。

当从“拒绝授信方案”切到“补充材料方案”时，Branch Summary 应该保留：

| 信息 | 作用 |
|---|---|
| 被离开分支的目标 | 说明刚才探索的是拒绝方案 |
| 关键风险证据 | 让新方案继续关注核心风险 |
| 工具结论 | 保留征信、规则、额度测算等关键结果 |
| 已做判断 | 哪些已经确定，哪些仍不确定 |
| 可复用上下文 | 新分支不要重复踩坑 |

## 常见误区

### 误区一：Session 就是聊天记录

不是。Session Tree 记录的是 Runtime History。聊天消息只是其中一种 entry。

### 误区二：Compaction 就是删历史

不是。Compaction 不删除 JSONL 里的旧 entry，它只是改写给 LLM 的上下文视图。

### 误区三：firstKeptEntryId 表示从哪里开始压缩

更准确是：它表示压缩后从哪里开始保留原文。

### 误区四：Branch Summary 和 Compaction 是一回事

不是。Compaction 处理当前 branch 太长；Branch Summary 处理 branch 切换时的路径移交。

### 误区五：branch 一定比线性流程好

也不是。线性业务流程可以继续走同一条 branch。branch 的价值主要在多路径探索、回退、对比、审计和上下文隔离。

## 我的阶段性理解

我现在对 Pi Session 的理解是：

```text
Session Tree 是 Agent Harness 的长期记忆结构；
buildSessionContext 是当前上下文视图生成器；
Compaction 是当前路径的长上下文治理；
Branch Summary 是多路径探索中的经验移交机制。
```

这对后续学习 Extension System 很重要。因为 Extension 往往不是直接改 AgentLoop，而是在 Harness 层通过 hook、custom entry、custom_message、权限检查、审计记录等方式介入 Runtime。

也就是说，理解 Session Tree 后，再看 Extension，才能真正理解：

```text
扩展点不是外挂功能，而是 Runtime 治理能力的一部分。
```

## 小结

这一课最重要的不是记住某个函数，而是换掉一个心智模型：

```text
不要把 Agent Session 当成聊天记录；
要把它当成可以分支、回退、压缩、导航和审计的 Runtime History Tree。
```

当这个模型建立起来，Compaction 和 Branch Summary 就不再是两个孤立概念：

```text
Compaction 让当前 branch 能在长上下文下继续走；
Branch Summary 让 Agent 从 old branch 切到 new branch 时不丢掉关键经验。
```
