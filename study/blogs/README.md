# 学习博文草稿

本目录用于把阶段课程沉淀成可发布的学习博文。

目标：

```text
一边复习和强化记忆，一边把学习成果分享给更多想了解 Agent Runtime / Agent Harness 的朋友。
```

## 写作规范

所有博文写作必须遵守：

```text
study/blogs/WRITING-GUIDELINES.md
```

特别注意：

```text
不暴露作者真实企业行业方向；
企业级类比统一使用泛化的金融行业 Agent；
代码片段可以有，但不能太多，不能把文章写成源码阅读文档。
```

## 当前博客草稿

| 编号 | 标题 | 对应课程 | 状态 |
|---|---|---|---|
| 0001 | Agent Loop：一个 Agent 到底是怎么跑起来的？ | 0001 Pi Agent Loop 源码精读 | 草稿完成 |
| 0002 | AgentMessage vs LLM Message：为什么内部消息和模型消息要分开？ | 0002 AgentMessage vs LLM Message | 草稿完成 |
| 0003 | Tool Call 不是函数调用：模型提出工具调用后，Runtime 做了什么？ | 0003 Tool Call 执行链路 | 草稿完成 |
| 0004 | AgentHarness 为什么不是 Agent 的简单封装？ | 0004 AgentHarness 是 Runtime 控制层 | 草稿完成 |
| 0005 | 为什么 Agent 的 Session 不是聊天记录：Session Tree、Compaction 与 Branch Summary | 0005 Session Tree 与 Compaction | 草稿完成 |

## 文件位置

```text
study/blogs/drafts/0001-agent-loop.md
study/blogs/drafts/0002-agent-message-vs-llm-message.md
study/blogs/drafts/0003-tool-call-lifecycle.md
study/blogs/drafts/0004-agent-harness-runtime-control.md
study/blogs/drafts/0005-session-tree-and-compaction.md
```

## 后续规则

1. 默认一课一篇。
2. 学完一段完整知识点后，可以写一篇博文强化记忆。
3. 如果某节课内容过大，可拆成多篇，但必须说明拆分原因。
4. 每篇文章应有清晰结论、结构化图表、工程类比和阶段性理解。
5. 企业案例默认使用金融行业 Agent，不暴露真实企业行业方向。
