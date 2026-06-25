# 课程 Handoff 协议

本文件定义每节课结束后的交接规则。目标是保证即使新开会话，也能从 `study/` 工作区恢复学习上下文，并保持一致的讲课风格和推进节奏。

## 核心原则

每节课结束后，必须形成一份 handoff 文档，记录：

- 学到哪里了。
- 用户已经理解了什么。
- 用户仍然卡在哪里。
- 读了哪些源码。
- 练习做到哪里。
- 更新了哪些文档。
- 下节课应该怎么接。
- 下一节课应该遵循哪些教学风格和结构要求。

## 教学协议

新会话恢复课程时，必须读取：

```text
study/TEACHING-PROTOCOL.md
```

`TEACHING-PROTOCOL.md` 定义老师的讲课方式、结构、节奏、理解检查、收尾规则和博客沉淀规则。

任何新会话不能只根据 `CURRENT.md` 简单输出课程摘要，而必须恢复同一套教学执行方式。

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

如果完成了完整知识点并需要沉淀博文，则更新：

```text
study/blogs/README.md
study/blogs/drafts/<blog-slug>.md
```

## Handoff 命名规则

```text
study/handoffs/0001-pi-agent-loop.md
study/handoffs/0002-agent-message-vs-llm-message.md
```

## 新会话恢复流程

新会话进入学习工作区时，按顺序阅读：

1. `study/README.md`
2. `study/MASTER-PLAN.md`
3. `study/TEACHING-PROTOCOL.md`
4. `study/MISSION.md`
5. `study/CURRENT.md`
6. `study/PROGRESS.md`
7. 最新的 `study/handoffs/*.md`
8. 必要时读取 `study/NOTES.md`
9. 如果要写博文，读取 `study/blogs/README.md` 与 `study/blogs/WRITING-GUIDELINES.md`

然后再进入当前课程或下一节课。

## 新会话最低行为要求

即使用户在新会话只说“继续下一节课”，老师也必须主动读取上述文件，尤其是 `TEACHING-PROTOCOL.md`，不能因为提示词短而降低教学质量。

## 何时写 Learning Record

Handoff 每节课都写；Learning Record 不一定每节课都写。

只有满足以下条件之一，才写 Learning Record：

- 用户能用自己的话解释一个非显然概念。
- 用户完成了练习并通过复盘。
- 用户纠正了一个重要误解。
- 学习使命发生变化。

Learning Record 不是课程日志，也不是流水账。
