# 学习笔记与偏好

这里记录临时偏好、教学注意事项和后续待办。它不是学习记录，不用于证明已经掌握某个概念。

## 教学偏好

- 以中文为主。
- 讲概念时要结合源码、调用链和工程场景。
- 不喜欢泛泛讲理论，要能落到实际代码、练习和架构设计。
- 需要持续对照已有经验：Java 后端、Dify、企业 AI 平台、Coding Agent、MCP。
- 每次学习最好有明确产出，例如图、表、练习代码、复盘文档。
- 对于类似 AgentHarness、Session、Compaction 这类内容多、新概念多、环节多的课程，默认使用结构化输出：分层表格、流程图、职责边界图、设计图、小结卡片。
- 高概念密度课程应先给总览图，再分模块拆解，最后用企业场景回填；必要时可生成更美观的图片辅助理解。

## 博文写作偏好

- 每学完一段完整知识点课程，可以写对应博文，用于强化记忆和对外分享。
- 默认一课一篇；如果单课内容特别大，再按主概念篇、源码流程篇、企业设计篇拆分。
- 企业级案例不要暴露真实企业行业方向，统一使用泛化的金融行业 Agent 作为类比。
- 博文要内容完整，但不要堆源码；代码片段可以有，但不能太多。
- 博文应有结论、结构化表格或流程图、工程类比、常见误区和阶段性理解。
- 博文写作规范已沉淀到 `study/blogs/WRITING-GUIDELINES.md`。

## 当前关注点

- Agent Loop 是怎么运行的。
- Harness 在真实项目中到底承担什么职责。
- Pi 的 `pi-core` / `pi-coding-agent` 如何分层。
- 如何从 Dify 平台型 Agent 经验升级到底层 Agent Runtime 工程能力。
- 如何形成可对外表达的“我能开发 Agent Harness”的能力证明。
- 如何把学习成果沉淀成可发布的技术博文系列。

## 待办

- [x] 完成第一课：Pi Agent Loop 源码精读。
- [x] 画出 prompt -> model -> tool call -> tool result -> next turn 的时序图。
- [x] 读完 `packages/agent/src/agent-loop.ts`。
- [x] 完成第二课：AgentMessage vs LLM Message。
- [x] 完成第三课：Tool Call 执行链路。
- [x] 完成第四课：AgentHarness 是 Runtime 控制层。
- [x] 完成前四课对应 4 篇博文草稿。
- [ ] 设计 Tool Audit Extension。
- [ ] 设计 Bash Permission Extension。
- [ ] 把 Java Debug Skill 做成可复用 Skill。
