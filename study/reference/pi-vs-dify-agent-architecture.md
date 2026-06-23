# Pi vs Dify Agent 架构对比

> 状态：待课程 0010 完成后补全。

本文件用于把已有 Dify / Java + Dify 接口经验，与 Pi 底层 Agent Harness 工程做对照。

## 初始对照

| 维度 | Dify / Java + Dify | Pi Agent Harness |
|---|---|---|
| 主要关注 | 业务编排、知识库、节点配置、平台接入 | Agent Loop、工具执行、状态、Session、Extension、Runtime |
| Agent 控制权 | 多由平台托管 | 由代码和 Harness 控制 |
| 工具治理 | 平台节点 / API 封装 | schema + hook + extension + tool result |
| 会话管理 | 平台会话 | JSONL / tree / branch / compaction |
| 扩展方式 | Workflow 节点、工具、插件 | Extension、Skill、Prompt Template、SDK、RPC |
| 学习价值 | 快速落地业务 Agent | 掌握底层 Agent 工程能力 |

## 待补充

- [ ] Dify Workflow 与 AgentLoop 的边界
- [ ] Dify 工具节点与 Pi Tool 的差异
- [ ] Dify 知识库与 Pi context/resource/skill 的差异
- [ ] 企业平台中如何同时使用 Dify 和自研 Harness
