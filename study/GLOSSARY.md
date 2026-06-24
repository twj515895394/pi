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

**Tool Call**：
assistant message content 中由模型提出的工具调用请求，包含工具名、toolCallId 和参数；它不是可信命令，必须经过 Runtime 校验和治理。
_应避免_：直接函数调用、可信命令

**ToolResult message**：
工具执行结果被包装后的消息，使用 `role: "toolResult"` 回到上下文，使 LLM 能基于工具观察结果继续推理。
_应避免_：工具直接返回、函数返回值

**transformContext**：
LLM 调用前对 `AgentMessage[]` 做上下文层面的处理，例如裁剪、压缩或注入外部上下文。
_应避免_：消息格式转换

**convertToLlm**：
LLM 调用前将 `AgentMessage[]` 转成模型可理解的 `Message[]`，并过滤、脱敏、格式化或转换内部消息。
_应避免_：上下文压缩、消息追加

**beforeToolCall**：
工具执行前的治理钩子，在参数校验后、真正执行前运行，适合做权限、审批、风控、白名单和危险操作拦截。
_应避免_：结果脱敏、输出压缩

**afterToolCall**：
工具执行后的治理钩子，在生成 toolResult message 前运行，适合做脱敏、压缩、错误标准化、审计补充、结果改写和 terminate 控制。
_应避免_：执行前权限判断

**prepareArguments**：
工具自定义的参数预处理逻辑，用于整理、兼容、修正模型输出的 raw arguments。
_应避免_：schema 校验

**validateToolArguments**：
基于工具 schema 对准备后的参数做正式校验。
_应避免_：参数兼容修正

**tool_execution_start / update / end**：
工具执行生命周期事件，分别表示工具开始、过程更新和结束，通常用于 UI 状态展示与审计。
_应避免_：toolcallupdate、toolcallend

**Tool Permission Layer**：
企业 Agent 中用于控制工具是否可执行的权限与风控层，通常检查用户身份、工具权限、操作类型、资源归属、参数风险、审批状态和审计信息。
_应避免_：简单 if 判断、单一黑名单

**CustomAgentMessages**：
Pi 允许应用通过 TypeScript declaration merging 扩展的自定义消息集合，使 AgentMessage 能承载应用层消息。
_应避免_：普通聊天消息

**内部领域模型**：
系统内部用于表达完整事实、状态和业务语义的模型，不等同于外部接口请求结构。
_应避免_：接口 DTO、API 请求对象

**接口 DTO**：
面向外部接口的数据传输结构，只表达对方接口需要且允许消费的信息。
_应避免_：内部领域模型
