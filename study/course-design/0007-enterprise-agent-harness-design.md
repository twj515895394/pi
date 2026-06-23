# 0007：企业级 Agent Harness 设计

## 本课目标

把 Pi 的底层设计迁移到企业 Java / AI 平台场景，形成可讲述、可落地的 Agent Harness 架构方案。

## 对照范围

```text
Pi AgentLoop / Agent / AgentHarness / AgentSession
Dify Workflow / Chatflow / Tool / Knowledge Base
Java 企业后端 / API Gateway / 审计 / 权限 / 任务调度 / 观测
```

## 核心问题

- 哪些能力应该由 Dify 承担？
- 哪些能力应该由自研 Agent Harness 承担？
- 企业工具调用如何设计权限、审计、幂等、错误处理？
- Session、Memory、Trace、Eval 如何分层？
- 如何向团队解释“会开发 Agent Harness”的能力？

## 小练习

设计一个企业内部 Coding / 运维 / 业务流程 Agent Harness 架构图。

## 复盘产出

```text
study/reference/pi-vs-dify-agent-architecture.md
```
