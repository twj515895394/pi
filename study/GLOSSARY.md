# Pi Agent Harness 术语表

本术语表用于沉淀本学习工作区的规范语言。只有当某个术语已经被真正理解，并且能在源码阅读或练习中正确使用时，才加入正式术语表。

## 术语

**Agent Loop**：
模型、工具和消息上下文之间的运行闭环：LLM 生成 assistant message，若包含 tool call 则执行工具，并将 tool result 作为消息回灌给 LLM 继续推理。
_应避免_：AI 聊天循环、简单对话流程

**AgentMessage**：
Pi Agent 内部使用的完整消息账本，承载标准 LLM 消息以及可能的内部自定义消息；它服务 Agent Runtime，不一定直接发送给 LLM。
_应避免_：LLM 消息、聊天消息

**LLM Message**：
模型 API 能理解的消息格式，由 `convertToLlm` 从 `AgentMessage[]` 转换而来，是面向 LLM Provider 的接口 DTO。
_应避免_：内部消息、Agent 状态

**ToolResult message**：
工具执行结果被包装后的消息，使用 `role: "toolResult"` 回到上下文，使 LLM 能基于工具观察结果继续推理。
_应避免_：工具直接返回、函数返回值

**transformContext**：
LLM 调用前对 `AgentMessage[]` 做上下文层面的处理，例如裁剪、压缩或注入外部上下文。
_应避免_：消息格式转换

**convertToLlm**：
LLM 调用前将 `AgentMessage[]` 转成模型可理解的 `Message[]`，并过滤、脱敏、格式化或转换内部消息。
_应避免_：上下文压缩、消息追加

**tool_execution_start / update / end**：
工具执行生命周期事件，分别表示工具开始、过程更新和结束，通常用于 UI 状态展示与审计。
_应避免_：toolcallupdate、toolcallend

**CustomAgentMessages**：
Pi 允许应用通过 TypeScript declaration merging 扩展的自定义消息集合，使 AgentMessage 能承载应用层消息。
_应避免_：普通聊天消息

**内部领域模型**：
系统内部用于表达完整事实、状态和业务语义的模型，不等同于外部接口请求结构。
_应避免_：接口 DTO、API 请求对象

**接口 DTO**：
面向外部接口的数据传输结构，只表达对方接口需要且允许消费的信息。
_应避免_：内部领域模型
