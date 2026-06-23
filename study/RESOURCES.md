# Pi Agent Harness 学习资源

本文件记录学习 Agent Harness 所需的高信任资源。资源应服务于当前使命：通过 Pi 项目学习底层 Agent Runtime / Harness 工程。

## 知识 (Knowledge)

- [Pi 项目 README](../README.md)
  项目入口，说明 Pi Agent Harness 的整体定位、包结构、权限与容器化策略、开发命令。适用于：确认学习主线和项目边界。

- [pi-agent-core README](../packages/agent/README.md)
  解释 AgentMessage、LLM Message、事件流、tool execution、Agent Options、low-level API。适用于：学习 Agent Loop 和 runtime 核心。

- [pi-coding-agent README](../packages/coding-agent/README.md)
  解释 Pi 作为 terminal coding harness 的使用方式、sessions、skills、extensions、SDK、RPC、设计哲学。适用于：理解底层 runtime 如何包装成真实 coding agent。

- [SDK 文档](../packages/coding-agent/docs/sdk.md)
  说明如何用 `createAgentSession()`、`AgentSession`、`AgentSessionRuntime` 编程式使用 Pi。适用于：学习如何把 Agent 能力嵌入其他应用。

- [Extensions 文档](../packages/coding-agent/docs/extensions.md)
  说明 extension 如何注册工具、命令、事件、UI、权限拦截、状态持久化。适用于：做阶段练习和理解可扩展点。

- [Compaction 文档](../packages/coding-agent/docs/compaction.md)
  说明上下文压缩、branch summarization、cut point、file tracking。适用于：学习长上下文治理。

- [Session Format 文档](../packages/coding-agent/docs/session-format.md)
  说明 JSONL session 的结构和分支机制。适用于：学习 coding agent 为什么需要树形 session。

- OpenAI Agents SDK 官方文档
  用于对照 Agent、Tools、Handoffs、Tracing 等通用概念。适用于：把 Pi 的实现与主流 Agent SDK 概念做横向比较。

- Model Context Protocol 官方规范
  用于理解 MCP 的 host / client / server、tools、resources、prompts 抽象。适用于：对比 Pi 的 extension / tool 机制与 MCP 的边界。

- Claude Code / Claude Agent SDK 官方文档
  用于对照 hooks、tools、subagents、permissions、session 等工程设计。适用于：理解 coding agent 市场需求中的常见 harness 能力。

## 智慧 (Wisdom)（社区）

- Pi 官方社区 / issue / PR
  用于观察真实 coding agent 使用问题、设计取舍和缺陷修复。

- OpenAI / Anthropic / MCP 相关开发者社区
  用于对照不同 Agent Runtime 的设计趋势。

## 缺口

- 还需要补充一组高质量文章或论文，专门解释 coding agent 的工程架构、权限模型、评测与可观测性。
- 还需要补充企业级 Agent Harness 的案例资源，尤其是与 Java 后端、内部平台、审批流、工具治理相关的资料。
- 还需要在完成第一轮源码学习后，筛选 Pi 源码中最适合复习的 reference 文档。
