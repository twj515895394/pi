# 使命：Pi Agent Harness 工程学习

## 为什么 (Why)

我已经具备 Dify / Java + Dify 接口型 Agent 的实践经验，但这类经验主要停留在平台使用和业务编排层。接下来要通过 Pi 项目学习更底层的 Agent Harness 工程：理解 Agent Loop、工具调用、状态管理、Session、Compaction、Extension、Runtime 等设计，最终能设计和开发真实可落地的企业级 Agent 系统。

## 成功图景 (Success looks like)

- 能清楚解释 Pi 中一次 `prompt()` 从用户输入到模型响应、工具调用、工具结果回灌、最终结束的完整链路。
- 能读懂并讲清楚 `packages/agent` 中 Agent Loop、Agent、AgentHarness 的职责边界。
- 能实现至少一个 Pi extension，例如工具调用审计、bash 权限拦截、计划模式或企业工具接入。
- 能把 Pi 的底层设计迁移到自己的 Java / 企业 AI 平台经验中，形成一套可讲述的 Agent Harness 架构方案。
- 能对外区分“会用 Dify 搭 Agent”和“会设计 Agent Runtime / Harness”的能力差异。

## 约束条件 (Constraints)

- 学习要以源码为中心，避免泛泛讲概念。
- 每节课要短小、可复盘、可继续追踪。
- 学习内容要优先服务于实际工程能力，而不是纯理论。
- 练习最好能结合 Java 后端、企业 AI 平台、Coding Agent、Dify 经验。
- 学习记录只在真正理解某个概念后再写，不做活动流水账。

## 超出范围 (Out of scope)

- 暂不深入训练模型、微调、强化学习或底层推理框架。
- 暂不把重点放在 Dify 工作流配置本身。
- 暂不追求一次性读完整个 Pi 项目，而是按 Agent Harness 主线切片学习。
- 暂不优先做 UI 美化，除非它服务于理解 Agent Runtime 或 Extension 机制。
