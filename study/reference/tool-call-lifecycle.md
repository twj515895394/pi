# Tool Call 生命周期

> 状态：待课程 0003 / 0004 完成后补全。

本文件用于沉淀 Pi 中工具调用从模型生成到执行完成的完整生命周期。

## 初始链路

```text
assistant message contains toolCall
  -> find tool by name
  -> prepareArguments
  -> validateToolArguments
  -> beforeToolCall
  -> tool.execute
  -> afterToolCall
  -> tool_execution_end
  -> createToolResultMessage
  -> append to messages
```

## 待补充

- [ ] sequential 与 parallel 工具执行差异
- [ ] 工具不存在时的错误处理
- [ ] 工具抛异常时如何回灌给模型
- [ ] terminate 机制
- [ ] beforeToolCall / afterToolCall 适合的企业治理场景
