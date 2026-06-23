# 学习笔记与偏好

这里记录临时偏好、教学注意事项和后续待办。它不是学习记录，不用于证明已经掌握某个概念。

## 教学偏好

- 以中文为主。
- 讲概念时要结合源码、调用链和工程场景。
- 不喜欢泛泛讲理论，要能落到实际代码、练习和架构设计。
- 需要持续对照已有经验：Java 后端、Dify、企业 AI 平台、Coding Agent、MCP。
- 每次学习最好有明确产出，例如图、表、练习代码、复盘文档。

## 当前关注点

- Agent Loop 是怎么运行的。
- Harness 在真实项目中到底承担什么职责。
- Pi 的 `pi-core` / `pi-coding-agent` 如何分层。
- 如何从 Dify 平台型 Agent 经验升级到底层 Agent Runtime 工程能力。
- 如何形成可对外表达的“我能开发 Agent Harness”的能力证明。

## 待办

- [ ] 完成第一课：Pi Agent Loop 源码精读。
- [ ] 画出 prompt -> model -> tool call -> tool result -> next turn 的时序图。
- [ ] 读完 `packages/agent/src/agent-loop.ts`。
- [ ] 设计 Tool Audit Extension。
- [ ] 设计 Bash Permission Extension。
- [ ] 把 Java Debug Skill 做成可复用 Skill。
