# 课程路线图

本文件是 `MASTER-PLAN.md` 的阶段路线图视图。

`MASTER-PLAN.md` 是唯一权威学习主计划；本文件用于展开阶段目标、源码范围和阶段产出。

## 总目标

从“会使用 Dify / Java 接口集成 Agent”升级为“能理解并开发 Agent Runtime / Harness”。

## 阶段总览

```text
阶段 1：Agent Runtime 最小闭环
  -> 阶段 2：Harness Runtime 控制层
  -> 阶段 3：Extension / Skills / Resource 能力扩展
  -> 阶段 4：运行模式与平台集成
  -> 阶段 5：企业级 Agent Harness 设计与 Capstone
```

## 阶段 1：Agent Runtime 最小闭环

目标：看懂 Agent 的最小运行闭环。

核心源码：

```text
packages/agent/src/agent.ts
packages/agent/src/agent-loop.ts
packages/agent/src/types.ts
packages/agent/src/harness/messages.ts
```

课程：

| 课次 | 主题 | 状态 | 产出 |
|---|---|---|---|
| 0001 | Pi Agent Loop 源码精读 | 已完成 | 调用链图 |
| 0002 | AgentMessage vs LLM Message | 已完成 | 消息模型对照表 |
| 0003 | Tool Call 执行链路 | 已完成 | 工具生命周期图 |

需要掌握：

- `prompt()` 如何进入 `runAgentLoop()`。
- `AgentMessage[]` 如何变成 LLM 可理解的消息。
- 模型响应如何触发 tool call。
- tool result 为什么要回灌给模型。
- 工具为什么是受 schema、校验、hook、错误处理、事件流治理的执行单元。

## 阶段 2：Harness Runtime 控制层

目标：理解 Harness 为什么是 Agent 工程的核心运行容器。

核心源码：

```text
packages/agent/src/harness/agent-harness.ts
packages/agent/src/harness/types.ts
packages/agent/src/harness/messages.ts
packages/agent/src/harness/system-prompt.ts
packages/agent/src/harness/session/*
packages/agent/src/harness/compaction/*
```

课程：

| 课次 | 主题 | 状态 | 产出 |
|---|---|---|---|
| 0004 | AgentHarness 是 Runtime 控制层 | 已完成 | Harness 职责地图 |
| 0005 | Session Tree 与 Compaction | 未开始 | Session Tree 数据模型图、上下文治理流程图 |

需要掌握：

- Harness 如何组织 model、session、resources、tools、queues、handlers。
- TurnState 如何生成。
- Harness 如何把 hook 接入 AgentLoop。
- Harness 如何处理 pending session writes、phase、abort、settled。
- JSONL session 为什么适合 coding agent。
- session tree 如何支持 fork / branch。
- compaction 为什么不能随便截断 tool result。
- branch summary 如何保留被离开分支的重要上下文。

## 阶段 3：Extension / Skills / Resource 能力扩展

目标：理解 Pi 如何把底层 Agent Runtime 包装成真实 coding agent 产品能力。

核心源码：

```text
packages/coding-agent/src/core/extensions/*
packages/coding-agent/src/core/resource-loader.ts
packages/coding-agent/src/core/skills.ts
packages/coding-agent/src/core/prompt-templates.ts
packages/coding-agent/src/core/system-prompt.ts
packages/coding-agent/src/core/tools/*
```

课程：

| 课次 | 主题 | 状态 | 产出 |
|---|---|---|---|
| 0006 | Extension System | 未开始 | 自定义 extension 示例 |
| 0008 | Coding Tools | 未开始 | 工具风险矩阵 |
| 0010 | ResourceLoader / Skills / Prompt Templates | 未开始 | 资源加载与能力注入图 |

需要掌握：

- Extension 能注册哪些能力。
- Extension 如何监听事件。
- 自定义工具如何进入 Agent。
- 权限拦截、审计、UI 交互、状态持久化分别应该放在哪些事件里。
- AGENTS.md / CLAUDE.md / SYSTEM.md 如何进入上下文。
- Skills 如何被发现、展示和调用。
- prompt templates 和 skills 的边界是什么。

## 阶段 4：运行模式与平台集成

目标：理解 Pi 如何把同一个 Agent Runtime 暴露给 CLI、TUI、JSON、RPC、SDK 等不同入口。

核心源码：

```text
packages/coding-agent/src/core/sdk.ts
packages/coding-agent/src/core/agent-session.ts
packages/coding-agent/src/core/agent-session-runtime.ts
packages/coding-agent/src/main.ts
packages/coding-agent/src/modes/*
packages/coding-agent/docs/rpc.md
packages/coding-agent/docs/json.md
packages/coding-agent/docs/sdk.md
```

课程：

| 课次 | 主题 | 状态 | 产出 |
|---|---|---|---|
| 0009 | SDK / RPC / CLI 模式 | 未开始 | 多运行模式对照图 |
| 0011 | Provider Layer | 未开始 | Provider 请求治理方案 |
| 0012 | Evaluation / Observability | 未开始 | Agent 可观测性设计 |

需要掌握：

- `createAgentSession()` 创建了哪些服务。
- `AgentSession` 如何连接 Agent、SessionManager、SettingsManager、ResourceLoader、ExtensionRunner。
- interactive / print / rpc / sdk 为什么能共享 `AgentSession`。
- 如果把 Pi 嵌入 Java 平台，应该走 SDK、RPC 还是自定义 bridge。

## 阶段 5：企业级 Agent Harness 设计与 Capstone

目标：把源码理解转化为自己的架构设计能力。

课程：

| 课次 | 主题 | 状态 | 产出 |
|---|---|---|---|
| 0007 | 企业级 Agent Harness 设计 | 未开始 | Java / Dify / Pi 对照架构 |
| 0013 | Capstone Project | 未开始 | 企业级 Agent Harness 方案 |
| 0014 | AI Coding Agent 横向对比 | 未开始 | Pi / Codex / Claude Code / Dify 对比 |
| 0015 | 阶段复盘与下一步 | 未开始 | 学习复盘与下一阶段规划 |

需要掌握：

- 如何从 Dify 平台型 Agent 经验升级到底层 Agent Runtime 工程能力。
- 如何设计企业级 Agent Harness。
- 如何形成可对外表达的“我能开发 Agent Harness”的能力证明。

## 阶段练习

1. **Tool Audit Extension**：记录每次工具调用和结果。
2. **Bash Permission Extension**：拦截危险 bash 命令并请求确认。
3. **Plan Mode Extension**：实现 `/plan` 与 `/execute-plan`。
4. **Java Debug Skill**：把 Java 故障排查经验封装成 Skill。

## 30 天参考节奏

30 天只是参考节奏，不是硬性日程。

```text
第 1 周：Agent Runtime 最小闭环
第 2 周：Harness Runtime 控制层与 Session
第 3 周：Extension、Skills、Coding Agent 工程层
第 4 周：企业级设计、练习和 Capstone
```

## 验收标准

学习结束时，应能产出：

- 一张 Pi Agent Harness 架构图。
- 一张 prompt -> tool call -> tool result -> next turn 时序图。
- 一篇 Dify vs Pi Agent Harness 对比文档。
- 至少一个可运行的 Pi extension。
- 一个结合 Java 后端经验的 Agent Harness 设计方案。
