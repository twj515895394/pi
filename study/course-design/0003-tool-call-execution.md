# 0003：Tool Call 执行链路

## 本课目标

理解 Pi 中 tool call 从模型生成到工具执行、结果回灌的完整生命周期。

## 源码范围

```text
packages/agent/src/agent-loop.ts
packages/agent/src/types.ts
packages/coding-agent/src/core/tools/index.ts
packages/coding-agent/src/core/tools/read.ts
packages/coding-agent/src/core/tools/bash.ts
packages/coding-agent/src/core/tools/edit.ts
packages/coding-agent/src/core/tools/write.ts
```

## 核心问题

- 模型产生的 tool call 在源码中是什么结构？
- Pi 如何找到对应 tool？
- 参数如何 prepare 和 validate？
- sequential / parallel tool execution 如何选择？
- 工具失败后为什么要变成 toolResult，而不是只打印错误？

## 小练习

基于源码画出 Tool Call 生命周期图，并标出：

```text
find tool
prepare args
validate args
beforeToolCall
tool.execute
afterToolCall
createToolResultMessage
```

## 复盘产出

```text
study/reference/tool-call-lifecycle.md
```
