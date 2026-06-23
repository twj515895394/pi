# AgentMessage 是内部领域模型，LLM Message 是接口 DTO

用户已经能够解释 Pi 为什么区分 `AgentMessage` 和 `LLM Message`：`AgentMessage` 范围更大，服务 Agent Loop、Harness、Session、UI、Extension、审计和业务状态；`LLM Message` 范围更小，是每次模型调用前由 `convertToLlm` 生成的外部接口契约。

Evidence: 用户正确判断了 `branchSummary`、`compactionSummary`、`bashExecution`、`AuditEventMessage`、`PermissionDecisionMessage` 和 `ClearanceStatusMessage` 是否以及如何进入 LLM，并能用企业清关 Agent 的场景解释消息边界。

Implications: 后续可以进入 Tool Call 执行链路和企业权限拦截设计，不需要再反复解释内部消息与模型消息的基础边界。