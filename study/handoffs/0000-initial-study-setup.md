# Handoff：0000 学习工作区初始化

## 本节状态

- 状态：已完成
- 日期：2026-06-23
- 对应阶段：学习工作区初始化

## 已完成内容

已在仓库根目录创建 `study/` 学习工作区，用于持续学习 Pi Agent Harness。

已创建的核心文件包括：

```text
study/README.md
study/MISSION.md
study/LEARNING-WORKFLOW.md
study/COURSE-ROADMAP.md
study/SOURCE-READING-MAP.md
study/EXERCISES.md
study/RESOURCES.md
study/GLOSSARY.md
study/NOTES.md
study/PROGRESS.md
study/CURRENT.md
study/HANDOFF-PROTOCOL.md
```

已创建的核心目录包括：

```text
study/course-design/
study/lessons/
study/reference/
study/learning-records/
study/assets/
study/templates/
study/exercises/
study/handoffs/
```

## 当前学习主线

目标是从 Dify / Java + Dify 接口型 Agent 经验，升级到底层 Agent Harness 工程能力。

核心路线：

```text
Agent Loop
  -> AgentMessage / LLM Message
  -> Tool Call
  -> AgentHarness
  -> Session / Compaction
  -> Extension / Skills
  -> Coding Agent CLI / SDK / RPC
  -> 企业级 Agent Harness 设计
```

## 当前课程状态

下一节课是：

```text
0001：Pi Agent Loop 源码精读
```

状态：未开始。

## 下节课入口

下节课开始前阅读：

```text
study/course-design/0001-pi-agent-loop.md
study/reference/pi-agent-loop-sequence.md
```

并准备阅读源码：

```text
packages/agent/src/agent.ts
packages/agent/src/agent-loop.ts
packages/agent/src/types.ts
```

## 下节课目标

读懂一次用户输入如何进入 Pi 的 Agent Loop，并理解它如何变成模型响应、工具调用、工具结果回灌和下一轮模型调用。

## 新会话续接提示

如果新开会话，请说：

```text
继续 Pi Agent Harness 学习。请先读取 study/README.md、study/CURRENT.md 和 study/handoffs/0000-initial-study-setup.md，然后从第一课开始。
```

## 注意事项

- 每节课结束后必须写新的 handoff。
- Handoff 记录学习过程和续接状态。
- Learning Record 只在用户真正展示理解后写入。
