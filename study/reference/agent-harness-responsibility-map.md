# Agent Harness 职责地图

> 状态：待阶段 3 完成后补全。

本文件用于沉淀 Agent Harness 的职责边界。

## 初始假设

```text
AgentLoop = 模型调用 + 工具调用循环
Agent = 带状态的 AgentLoop 封装
AgentHarness = Runtime 控制层，负责连接 session、resources、tools、hooks、queues、compaction、provider options
CodingAgent = 面向用户和外部系统的产品层封装
```

## 待补充

- [ ] Pi 中 `AgentHarness` 的字段职责表
- [ ] `createTurnState()` 解析
- [ ] `createLoopConfig()` 解析
- [ ] Harness 与 Java 企业平台的类比
- [ ] Harness 与 Dify Workflow 的边界对比
