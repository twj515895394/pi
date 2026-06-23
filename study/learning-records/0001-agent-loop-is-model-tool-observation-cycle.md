# Agent Loop 是模型-工具-观察结果闭环

用户已经能够解释 Pi Agent Loop 的主链路：外部输入先统一成 `AgentMessage[]`，每次 LLM 调用前再转换为模型可理解的 `Message[]`；模型产生 `toolCall` 后，工具结果不会直接返回给用户，而是包装成 `toolResult message` 回灌到上下文，让模型继续推理、继续调用工具或生成最终回答。

Evidence: 用户正确回答了 `normalizePromptInput`、`currentContext.messages` 与 `newMessages`、`toolResult message` 回灌、`transformContext` 与 `convertToLlm` 分工，以及事件驱动状态更新的检查问题；已纠正 `uiStatus` 过滤阶段和工具事件命名两个小误区。

Implications: 下一课可以进入 `AgentMessage vs LLM Message` 的设计动机，不需要再从普通 tool calling 基础开始讲。