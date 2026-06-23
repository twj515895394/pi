# 当前学习状态

本文件是新会话续接学习时的第一入口。任何 Agent 进入 `study/` 工作区后，应先阅读本文件，再阅读最新 handoff。

## 当前课程

`0001`：Pi Agent Loop 源码精读

## 当前状态

进行中。

已经开始第一课，当前要读懂：

```text
Agent.prompt()
  -> runPromptMessages()
  -> createContextSnapshot()
  -> createLoopConfig()
  -> runAgentLoop()
```

## 最新 Handoff

```text
study/handoffs/0000-initial-study-setup.md
```

第一课完成前暂时没有 `0001` handoff。课程结束后必须生成：

```text
study/handoffs/0001-pi-agent-loop.md
```

## 当前源码范围

```text
packages/agent/src/agent.ts
packages/agent/src/agent-loop.ts
packages/agent/src/types.ts
packages/agent/README.md
```

## 当前问题

第一课要解决的问题：

```text
一次 prompt 如何进入 Agent Loop，并最终变成模型响应、工具调用、工具结果回灌和下一轮模型调用。
```

## 新会话续接方式

新会话中请先阅读：

1. `study/README.md`
2. `study/MISSION.md`
3. `study/CURRENT.md`
4. `study/handoffs/0000-initial-study-setup.md`
5. `study/course-design/0001-pi-agent-loop.md`

然后继续第一课。