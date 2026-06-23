# Pi 源码阅读地图

本文件用于记录学习 Pi Agent Harness 时的源码阅读顺序。原则是按 Agent 工程主线阅读，而不是按目录从头扫。

## 0. 仓库入口

```text
README.md
package.json
AGENTS.md
```

阅读目的：

- 确认项目定位：Pi Agent Harness。
- 确认 monorepo 包结构。
- 确认开发规则和测试命令。

## 1. Agent Core 最小闭环

```text
packages/agent/src/index.ts
packages/agent/src/types.ts
packages/agent/src/agent.ts
packages/agent/src/agent-loop.ts
```

阅读问题：

- `Agent` 和 `agentLoop` 的边界是什么？
- `AgentState` 保存哪些东西？
- `AgentLoopConfig` 控制哪些行为？
- 一次 run 为什么会产生 event stream？

## 2. 消息模型

```text
packages/agent/src/types.ts
packages/agent/src/harness/messages.ts
packages/coding-agent/src/core/messages.ts
```

阅读问题：

- `AgentMessage` 与 LLM `Message` 为什么要分开？
- 哪些消息应该给模型看，哪些只是 UI / session / extension 内部使用？
- `convertToLlm` 在什么边界执行？

## 3. Tool Calling

```text
packages/agent/src/agent-loop.ts
packages/coding-agent/src/core/tools/index.ts
packages/coding-agent/src/core/tools/read.ts
packages/coding-agent/src/core/tools/bash.ts
packages/coding-agent/src/core/tools/edit.ts
packages/coding-agent/src/core/tools/write.ts
packages/coding-agent/src/core/tools/file-mutation-queue.ts
```

阅读问题：

- 工具如何定义 schema？
- 参数在哪里校验？
- 工具执行失败时是 throw 还是返回错误内容？
- 文件修改为什么需要排队？
- bash 工具的风险点在哪里？

## 4. Harness 层

```text
packages/agent/src/harness/agent-harness.ts
packages/agent/src/harness/types.ts
packages/agent/src/harness/system-prompt.ts
packages/agent/src/harness/prompt-templates.ts
packages/agent/src/harness/skills.ts
```

阅读问题：

- Harness 比 AgentLoop 多承担了哪些职责？
- `createTurnState()` 组装了哪些东西？
- `createLoopConfig()` 如何把 Harness hook 接入 AgentLoop？
- `phase`、queue、pending session writes 各解决什么问题？

## 5. Session 与会话树

```text
packages/agent/src/harness/session/session.ts
packages/agent/src/harness/session/jsonl-repo.ts
packages/agent/src/harness/session/memory-repo.ts
packages/coding-agent/src/core/session-manager.ts
packages/coding-agent/docs/session-format.md
```

阅读问题：

- Session entry 有哪些类型？
- `id` / `parentId` 如何形成 tree？
- branch / fork / leaf 是怎么保存的？
- 为什么 coding agent 的历史不能只是线性 message list？

## 6. Compaction 与 Branch Summary

```text
packages/agent/src/harness/compaction/compaction.ts
packages/agent/src/harness/compaction/branch-summarization.ts
packages/coding-agent/src/core/compaction/*
packages/coding-agent/docs/compaction.md
```

阅读问题：

- 自动压缩何时触发？
- cut point 为什么要考虑 turn boundary？
- 为什么不能切在 tool result 上？
- branch summary 如何保留离开分支的信息？

## 7. Coding Agent 会话总控

```text
packages/coding-agent/src/core/sdk.ts
packages/coding-agent/src/core/agent-session.ts
packages/coding-agent/src/core/agent-session-runtime.ts
```

阅读问题：

- `createAgentSession()` 创建了哪些服务？
- `AgentSession` 如何连接 Agent、SessionManager、SettingsManager、ResourceLoader、ExtensionRunner？
- interactive / print / rpc / sdk 为什么能共享 `AgentSession`？

## 8. Extension 系统

```text
packages/coding-agent/src/core/extensions/index.ts
packages/coding-agent/src/core/extensions/types.ts
packages/coding-agent/src/core/extensions/runner.ts
packages/coding-agent/docs/extensions.md
packages/coding-agent/examples/extensions/
```

阅读问题：

- Extension 能注册哪些能力？
- Extension 如何监听事件？
- 自定义工具如何进入 Agent？
- 权限拦截、审计、UI 交互、状态持久化分别应该放在哪些事件里？

## 9. Resource Loading 与 Skills

```text
packages/coding-agent/src/core/resource-loader.ts
packages/coding-agent/src/core/skills.ts
packages/coding-agent/src/core/prompt-templates.ts
packages/coding-agent/src/core/system-prompt.ts
packages/coding-agent/docs/skills.md
packages/coding-agent/docs/prompt-templates.md
```

阅读问题：

- AGENTS.md / CLAUDE.md / SYSTEM.md 如何进入上下文？
- Skills 如何被发现、展示和调用？
- prompt templates 和 skills 的边界是什么？
- 项目级资源为什么需要 trust 机制？

## 10. 运行模式

```text
packages/coding-agent/src/main.ts
packages/coding-agent/src/modes/interactive/*
packages/coding-agent/src/modes/json.ts
packages/coding-agent/src/modes/rpc.ts
packages/coding-agent/src/modes/print.ts
packages/coding-agent/docs/rpc.md
packages/coding-agent/docs/json.md
packages/coding-agent/docs/sdk.md
```

阅读问题：

- CLI、TUI、JSON、RPC、SDK 共享了哪些核心？
- 模式层只处理 I/O，还是也影响 Agent 逻辑？
- 如果把 Pi 嵌入 Java 平台，应该走 SDK、RPC 还是自定义 bridge？

## 阅读顺序建议

```text
agent-loop.ts
  -> agent.ts
  -> harness/agent-harness.ts
  -> harness/types.ts
  -> coding-agent/core/sdk.ts
  -> coding-agent/core/agent-session.ts
  -> coding-agent/core/tools
  -> coding-agent/core/extensions
  -> session / compaction
```
