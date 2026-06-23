# 课程 Handoff 协议

本文件定义每节课结束后的交接规则。目标是保证即使新开会话，也能从 `study/` 工作区恢复学习上下文。

## 核心原则

每节课结束后，必须形成一份 handoff 文档，记录：

- 学到哪里了。
- 用户已经理解了什么。
- 用户仍然卡在哪里。
- 读了哪些源码。
- 练习做到哪里。
- 更新了哪些文档。
- 下节课应该怎么接。

## 每节课结束后的固定更新

```text
study/PROGRESS.md
study/CURRENT.md
study/handoffs/<lesson-id>.md
study/reference/<对应参考资料>.md
```

只有当用户真正展示理解后，才写入：

```text
study/learning-records/<next-id>-<slug>.md
```

## Handoff 命名规则

```text
study/handoffs/0001-pi-agent-loop.md
study/handoffs/0002-agent-message-vs-llm-message.md
```

## 新会话恢复流程

新会话进入学习工作区时，按顺序阅读：

1. `study/README.md`
2. `study/MISSION.md`
3. `study/CURRENT.md`
4. `study/PROGRESS.md`
5. 最新的 `study/handoffs/*.md`

然后再进入当前课程或下一节课。

## 何时写 Learning Record

Handoff 每节课都写；Learning Record 不一定每节课都写。

只有满足以下条件之一，才写 Learning Record：

- 用户能用自己的话解释一个非显然概念。
- 用户完成了练习并通过复盘。
- 用户纠正了一个重要误解。
- 学习使命发生变化。

Learning Record 不是课程日志，也不是流水账。