# Pi Agent Harness 学习主计划

本文件是 `study/` 工作区的唯一权威学习主计划。

它负责回答三个问题：

```text
1. 我们为什么学？
2. 主线怎么走？
3. 什么情况下动态加课、补课或进入练习？
```

`COURSE-ROADMAP.md`、`COURSE-LIST.md`、`SOURCE-READING-MAP.md`、`PROGRESS.md` 都应以本文件为准。

---

## 1. 学习总目标

从“会使用 Dify / Java 接口集成 Agent”升级为：

```text
能读懂 Agent Runtime / Harness 源码；
能解释 AgentLoop、Message、Tool、Harness、Session、Extension 的工程边界；
能设计并实现企业级 Agent Harness；
能把 Pi 的设计思想迁移到 Java / 企业 AI 平台 / Coding Agent 场景。
```

最终能力证明：

```text
1. 一张 Pi Agent Harness 总架构图。
2. 一张 prompt -> model -> toolCall -> toolResult -> next turn 时序图。
3. 一篇 Dify vs Pi Agent Harness 对比文档。
4. 至少一个可运行的 Pi Extension。
5. 一个结合 Java 后端经验的企业级 Agent Harness 设计方案。
```

---

## 2. 主学习路线

```text
阶段 1：Agent Runtime 最小闭环
  -> 阶段 2：Harness Runtime 控制层
  -> 阶段 3：Extension / Skills / Resource 能力扩展
  -> 阶段 4：运行模式与平台集成
  -> 阶段 5：企业级 Agent Harness 设计与 Capstone
```

---

## 3. 阶段规划

### 阶段 1：Agent Runtime 最小闭环

目标：理解一个 Agent 如何完成模型请求、工具调用、工具结果回灌和下一轮推理。

课程：

| 课次 | 主题 | 状态 | 核心产出 |
|---|---|---|---|
| 0001 | Pi Agent Loop 源码精读 | 已完成 | Agent Loop 调用链图 |
| 0002 | AgentMessage vs LLM Message | 已完成 | 消息模型对照表 |
| 0003 | Tool Call 执行链路 | 已完成 | Tool Call 生命周期图 |

阶段验收：

```text
能够讲清楚 prompt 如何进入 AgentLoop，assistant 如何产生 toolCall，toolResult 为什么必须回灌给模型。
```

---

### 阶段 2：Harness Runtime 控制层

目标：理解 AgentLoop 如何被包装成真实应用可运行、可治理、可恢复的 Runtime。

课程：

| 课次 | 主题 | 状态 | 核心产出 |
|---|---|---|---|
| 0004 | AgentHarness 是 Runtime 控制层 | 已完成 | Harness 职责地图 |
| 0005 | Session Tree 与 Compaction | 未开始 | Session Tree 数据模型图、Compaction 流程图 |

阶段验收：

```text
能够讲清楚 AgentLoop、Agent、AgentHarness 的职责边界；
能够解释 Session 为什么不是简单聊天记录；
能够解释 Compaction / Branch Summary 如何服务长上下文和分支会话。
```

---

### 阶段 3：Extension / Skills / Resource 能力扩展

目标：理解 Pi 如何把底层 Runtime 扩展成真实 Coding Agent 产品能力。

课程：

| 课次 | 主题 | 状态 | 核心产出 |
|---|---|---|---|
| 0006 | Extension System | 未开始 | Extension 生命周期图 |
| 0008 | Coding Tools | 未开始 | 工具风险矩阵 |
| 0010 | ResourceLoader / Skills / Prompt Templates | 未开始 | 资源加载与能力注入图 |

建议插入练习：

```text
Tool Audit Extension
Bash Permission Extension
Java Debug Skill
```

阶段验收：

```text
能够实现一个简单 Extension；
能够说明工具、命令、事件、UI、状态持久化分别应该挂在哪些扩展点。
```

---

### 阶段 4：运行模式与平台集成

目标：理解 Pi 如何把同一个 Agent Runtime 暴露给 CLI、TUI、JSON、RPC、SDK 等不同入口。

课程：

| 课次 | 主题 | 状态 | 核心产出 |
|---|---|---|---|
| 0009 | SDK / RPC / CLI 模式 | 未开始 | 多运行模式对照图 |
| 0011 | Provider Layer | 未开始 | Provider 请求治理方案 |
| 0012 | Evaluation / Observability | 未开始 | Agent 可观测性设计 |

阶段验收：

```text
能够判断企业平台接入 Pi 时，应该使用 SDK、RPC、JSON mode 还是自定义 bridge；
能够设计 provider 请求治理、审计和 observability 方案。
```

---

### 阶段 5：企业级 Agent Harness 设计与 Capstone

目标：把源码理解转化为自己的架构设计能力。

课程：

| 课次 | 主题 | 状态 | 核心产出 |
|---|---|---|---|
| 0007 | 企业级 Agent Harness 设计 | 未开始 | Java / Dify / Pi 对照架构 |
| 0013 | Capstone Project | 未开始 | 企业级 Agent Harness 方案 |
| 0014 | AI Coding Agent 横向对比 | 未开始 | Pi / Codex / Claude Code / Dify 对比 |
| 0015 | 阶段复盘与下一步 | 未开始 | 学习复盘与下一阶段规划 |

阶段验收：

```text
能够独立设计企业级 Agent Harness；
能够解释为什么 Dify、Pi、Coding Agent、MCP 在工程职责上不同；
能够把 Pi 的 Runtime 思想迁移到 Java / 企业 AI 平台。
```

---

## 4. 当前执行状态

当前已经完成：

```text
0001 Pi Agent Loop 源码精读
0002 AgentMessage vs LLM Message
0003 Tool Call 执行链路
0004 AgentHarness 是 Runtime 控制层
```

当前下一课：

```text
0005 Session Tree 与 Compaction
```

---

## 5. 动态补课规则

主线不变，但允许根据理解情况插入小课或练习。

| 触发情况 | 处理方式 |
|---|---|
| 主线概念没掌握 | 插入复习课 |
| 某个源码点很复杂 | 插入源码小课 |
| 概念懂了但工程感不足 | 插入设计练习 |
| 用户提出企业落地需求 | 插入企业案例课 |
| 当前阶段完成 | 进入阶段练习 |
| 阶段练习完成 | 进入下一阶段 |

动态补课必须满足：

```text
1. 不改变主线方向。
2. 不无限扩展。
3. 必须说明它服务哪个阶段目标。
4. 补课完成后要回到主线。
```

---

## 6. 高概念密度课程教学规则

对于 AgentHarness、Session、Compaction、Extension、Provider、Observability 这类内容多、新概念多、环节多的课程，默认采用：

```text
1. 先给总览图。
2. 再给分层表格。
3. 再给源码锚点。
4. 再给流程图或职责边界图。
5. 最后用企业场景回填。
6. 必要时生成更美观的图片辅助理解。
```

---

## 7. 30 天计划的定位

30 天只是参考节奏，不是硬性日程。

当前实际执行方式是：

```text
按课程切片推进；
按理解检查决定是否收尾；
按阶段验收决定是否进入练习；
按学习反馈适当插入补课或小课。
```

---

## 8. 文件职责

| 文件 | 职责 |
|---|---|
| `MASTER-PLAN.md` | 唯一权威学习主计划 |
| `COURSE-ROADMAP.md` | 阶段路线图，服务主计划 |
| `COURSE-LIST.md` | 课程清单，服务主计划 |
| `SOURCE-READING-MAP.md` | 源码阅读顺序，服务主计划 |
| `PROGRESS.md` | 当前进度看板 |
| `CURRENT.md` | 新会话续接入口 |
| `handoffs/` | 每课交接记录 |
| `reference/` | 复习用知识沉淀 |
| `learning-records/` | 通过理解检查后的学习记录 |
