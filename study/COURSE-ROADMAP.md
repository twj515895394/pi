# 课程路线图

这份路线图把学习拆成 5 个源码阶段、10 个课程切片和 4 个阶段练习。30 天只是参考节奏，不是硬性日程。

## 总目标

从“会使用 Dify / Java 接口集成 Agent”升级为“能理解并开发 Agent Runtime / Harness”。

## 5 个源码阶段

### 阶段 1：Agent Loop

目标：看懂 Agent 的最小运行闭环。

核心源码：

```text
packages/agent/src/agent.ts
packages/agent/src/agent-loop.ts
packages/agent/src/types.ts
```

需要掌握：

- `prompt()` 如何进入 `runAgentLoop()`。
- `AgentMessage[]` 如何变成 LLM 可理解的消息。
- 模型响应如何触发 tool call。
- tool result 为什么要回灌给模型。

### 阶段 2：Tool Calling 与工具治理

目标：理解工具不是裸函数，而是受 schema、校验、hook、错误处理、事件流治理的执行单元。

核心源码：

```text
packages/agent/src/agent-loop.ts
packages/coding-agent/src/core/tools/index.ts
packages/coding-agent/src/core/tools/read.ts
packages/coding-agent/src/core/tools/bash.ts
packages/coding-agent/src/core/tools/edit.ts
packages/coding-agent/src/core/tools/write.ts
```

需要掌握：

- tool schema 如何定义。
- 参数如何校验。
- `beforeToolCall` 和 `afterToolCall` 在哪里发挥作用。
- sequential 和 parallel tool execution 的差异。
- 工具错误如何进入 LLM 上下文。

### 阶段 3：AgentHarness 与 Runtime 控制层

目标：理解 Harness 为什么是 Agent 工程的核心能力。

核心源码：

```text
packages/agent/src/harness/agent-harness.ts
packages/agent/src/harness/types.ts
packages/agent/src/harness/messages.ts
packages/agent/src/harness/system-prompt.ts
```

需要掌握：

- Harness 如何组织 model、session、resources、tools、queues、handlers。
- TurnState 如何生成。
- Harness 如何把 hook 接入 AgentLoop。
- Harness 如何处理 pending session writes、phase、abort、settled。

### 阶段 4：Session、Tree、Compaction

目标：理解长任务、长上下文、多分支会话如何被管理。

核心源码：

```text
packages/agent/src/harness/session/*
packages/agent/src/harness/compaction/*
packages/coding-agent/src/core/session-manager.ts
packages/coding-agent/docs/compaction.md
packages/coding-agent/docs/session-format.md
```

需要掌握：

- JSONL session 为什么适合 coding agent。
- session tree 如何支持 fork / branch。
- compaction 为什么不能随便截断 tool result。
- branch summary 如何保留被离开分支的重要上下文。

### 阶段 5：Coding Agent 工程层与 Extension

目标：理解 Pi 如何把底层 Agent Runtime 包装成真实 coding agent 产品。

核心源码：

```text
packages/coding-agent/src/core/sdk.ts
packages/coding-agent/src/core/agent-session.ts
packages/coding-agent/src/core/extensions/*
packages/coding-agent/src/core/resource-loader.ts
packages/coding-agent/src/core/system-prompt.ts
packages/coding-agent/src/modes/*
```

需要掌握：

- `createAgentSession()` 如何组装 runtime。
- `AgentSession` 为什么是 interactive / print / rpc / sdk 的共享核心。
- Extension 如何注册工具、命令、事件、UI。
- Skills / prompt templates / context files 如何变成 Agent 能力。

## 10 个课程切片

| 课次 | 主题 | 产出 |
|---|---|---|
| 0001 | Pi Agent Loop：一次 prompt 如何跑起来 | 调用链图 |
| 0002 | AgentMessage vs LLM Message | 消息模型对照表 |
| 0003 | Tool Call 执行链路 | 工具生命周期图 |
| 0004 | beforeToolCall / afterToolCall | 权限拦截点说明 |
| 0005 | AgentHarness 是什么 | Harness 职责地图 |
| 0006 | Session Tree 与 JSONL | session 数据模型图 |
| 0007 | Compaction 与 Branch Summary | 上下文治理流程图 |
| 0008 | Coding Tools：read/bash/edit/write | 工具风险矩阵 |
| 0009 | Extension System | 自定义 extension 示例 |
| 0010 | 企业级 Agent Harness 设计 | Java / Dify / Pi 对照架构 |

## 4 个阶段练习

1. **Tool Audit Extension**：记录每次工具调用和结果。
2. **Bash Permission Extension**：拦截危险 bash 命令并请求确认。
3. **Plan Mode Extension**：实现 `/plan` 与 `/execute-plan`。
4. **Java Debug Skill**：把 Java 故障排查经验封装成 Skill。

## 30 天参考节奏

```text
第 1 周：Agent Loop 与 Tool Calling
第 2 周：AgentHarness 与 Session
第 3 周：Compaction、Extension、Coding Agent 工程层
第 4 周：完成练习并沉淀个人 Agent Harness 方案
```

## 验收标准

学习结束时，应能产出：

- 一张 Pi Agent Harness 架构图。
- 一张 prompt -> tool call -> tool result -> next turn 时序图。
- 一篇 Dify vs Pi Agent Harness 对比文档。
- 至少一个可运行的 Pi extension。
- 一个结合 Java 后端经验的 Agent Harness 设计方案。
