# Tool Call 是受控 Runtime 执行流水线

用户已经能够解释 Pi 中 tool call 的完整生命周期：LLM 只能提出 `toolCall`，Agent Runtime 需要先完成工具查找、参数整理、参数校验、执行前权限/风控拦截，再执行工具；执行后还要通过 `afterToolCall` 做结果脱敏、压缩、错误标准化，最终包装成 `toolResult message` 回灌给模型。

Evidence: 用户正确回答了为什么不能直接执行 `tool.execute()`、`prepareArguments` 与 `validateToolArguments` 的区别、`beforeToolCall` block 后为什么仍要生成 error toolResult、并行执行下事件顺序和 toolResult message 顺序的区别，以及企业 Tool Permission Layer 的最小检查项。

Implications: 后续可以进入 AgentHarness 作为 Runtime 控制层的整体设计，把模型、工具、权限、上下文、事件和 UI/CLI/SDK 串起来。