# Pi Agent Loop 时序图

> 状态：待第一课完成后补全。

本文件将在完成 `study/course-design/0001-pi-agent-loop.md` 后沉淀为正式参考资料。

## 目标

记录一次 prompt 在 Pi 中的完整链路：

```text
Agent.prompt()
  -> runAgentLoop()
  -> runLoop()
  -> streamAssistantResponse()
  -> executeToolCalls()
  -> createToolResultMessage()
  -> next turn / agent_end
```

## 待补充

- [ ] ASCII 时序图
- [ ] 关键函数列表
- [ ] 事件触发位置
- [ ] tool call 生命周期
- [ ] 本课 3 个关键理解
