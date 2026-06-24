# AgentHarness 是 Runtime 控制层

用户已经能够解释 Pi 中 `AgentHarness` 与 `AgentLoop`、`Agent` 的职责边界：`AgentLoop` 负责当前 run 的模型-工具闭环，`Agent` 是带状态的调用门面，`AgentHarness` 是完整 Agent 应用的 Runtime 控制层，负责协调 session、模型、工具、资源、队列、hook、provider 请求、权限审计、UI 事件、compaction 和 branch summary。

Evidence: 用户正确回答了为什么 `AgentLoop` 应保持小而纯、为什么 Session/Compaction/Branch Summary 属于 Harness、为什么 `createLoopConfig()` 是把 Runtime 能力注入 AgentLoop 的适配器，并能基于企业清关 Agent 场景列出 Harness 层应承载的请求合法性、权限整合、上下文整理、工具整理、模型选择、审计日志、请求日志、安全风控、业务工具集合和配置管理等能力。

Implications: 后续可以进入 Session Tree 与 Compaction 课程，进一步学习 Harness 如何管理长期会话、上下文压缩和分支跳转。教学上应对高概念密度内容默认使用表格、流程图、设计图和小结卡片。