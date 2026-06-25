# 课程清单

本文件是 `MASTER-PLAN.md` 的课程清单视图。

`MASTER-PLAN.md` 是唯一权威学习主计划；本文件只维护课程编号、主题、阶段归属、状态和主要产出。

## 课程总览

| 课次 | 阶段 | 主题 | 状态 | 主要产出 |
|---|---|---|---|---|
| 0001 | 阶段 1：Agent Runtime 最小闭环 | Pi Agent Loop 源码精读 | 已完成 | Agent Loop 调用链图 |
| 0002 | 阶段 1：Agent Runtime 最小闭环 | AgentMessage vs LLM Message | 已完成 | 消息模型对照表 |
| 0003 | 阶段 1：Agent Runtime 最小闭环 | Tool Call 执行链路 | 已完成 | Tool Call 生命周期图 |
| 0004 | 阶段 2：Harness Runtime 控制层 | AgentHarness 是 Runtime 控制层 | 已完成 | Harness 职责地图 |
| 0005 | 阶段 2：Harness Runtime 控制层 | Session Tree 与 Compaction | 未开始 | Session Tree 数据模型图、Compaction 流程图 |
| 0006 | 阶段 3：Extension / Skills / Resource 能力扩展 | Extension System | 未开始 | Extension 生命周期图 |
| 0007 | 阶段 5：企业级设计与 Capstone | 企业级 Agent Harness 设计 | 未开始 | Java / Dify / Pi 对照架构 |
| 0008 | 阶段 3：Extension / Skills / Resource 能力扩展 | Coding Tools | 未开始 | 工具风险矩阵 |
| 0009 | 阶段 4：运行模式与平台集成 | SDK / RPC / CLI 模式 | 未开始 | 多运行模式对照图 |
| 0010 | 阶段 3：Extension / Skills / Resource 能力扩展 | ResourceLoader / Skills / Prompt Templates | 未开始 | 资源加载与能力注入图 |
| 0011 | 阶段 4：运行模式与平台集成 | Provider Layer | 未开始 | Provider 请求治理方案 |
| 0012 | 阶段 4：运行模式与平台集成 | Evaluation / Observability | 未开始 | Agent 可观测性设计 |
| 0013 | 阶段 5：企业级设计与 Capstone | Capstone Project | 未开始 | 企业级 Agent Harness 方案 |
| 0014 | 阶段 5：企业级设计与 Capstone | AI Coding Agent 横向对比 | 未开始 | Pi / Codex / Claude Code / Dify 对比 |
| 0015 | 阶段 5：企业级设计与 Capstone | 阶段复盘与下一步 | 未开始 | 学习复盘与下一阶段规划 |

## 当前执行位置

```text
已完成：0001 - 0004
下一课：0005 Session Tree 与 Compaction
```

## 动态补课规则

允许根据学习情况插入小课，但必须服务主线阶段目标，并在补课后回到主线。

| 补课类型 | 例子 | 服务阶段 |
|---|---|---|
| 复习课 | AgentHarness Runtime 再拆解 | 当前阶段 |
| 源码小课 | branch summary 细读 | 阶段 2 |
| 设计练习 | Tool Audit Extension | 阶段 3 |
| 企业案例课 | 清关 Agent Harness 权限设计 | 阶段 5 |

## 第一阶段练习建议

原计划中“先完成 0001-0004，再开始第一个 extension 练习”。

当前已完成 0001-0004，但为了让 Harness 长期状态主线更完整，建议先完成 0005 Session Tree 与 Compaction，再进入第一个 extension 练习。