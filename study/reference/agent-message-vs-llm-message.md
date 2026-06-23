# AgentMessage vs LLM Message

> 状态：待课程 0002 完成后补全。

本文件用于沉淀 Pi 中内部消息模型与 LLM 消息模型的区别。

## 初始假设

```text
AgentMessage = Agent 内部完整消息模型
LLM Message = 每次模型调用前转换出的模型可理解消息
```

## 待补充

- [ ] AgentMessage 类型清单
- [ ] 哪些消息进入 session
- [ ] 哪些消息显示给用户
- [ ] 哪些消息进入 LLM
- [ ] transformContext 与 convertToLlm 的边界
- [ ] 企业 Agent 中审计事件和业务状态如何处理
