# 0012：Evaluation / Observability

## 本课目标

理解 Agent Harness 在真实工程中如何做可观测性、质量评估和成本治理。

## 源码范围

```text
packages/coding-agent/src/core/agent-session.ts
packages/coding-agent/src/core/session-manager.ts
packages/coding-agent/src/core/export-html/*
packages/coding-agent/src/core/timings.ts
packages/coding-agent/test/suite/*
```

## 核心问题

- Agent 事件流如何变成可观测数据？
- session 数据如何用于复盘和评测？
- token、cost、tool call、error 如何统计？
- 企业 Agent 平台需要哪些评估指标？

## 小练习

基于 session 数据设计一份 Agent 运行报告。
