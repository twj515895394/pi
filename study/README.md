# Pi Agent Harness 学习工作区

这个目录用于记录围绕 Pi 项目的 Agent Harness 学习过程。它不是项目官方文档的替代品，而是个人学习工作区：记录为什么学、怎么学、读哪些源码、做哪些练习、何时形成学习记录。

## 学习目标

从已有的 Dify / Java + Dify 接口型 Agent 经验，升级到底层 Agent Harness 工程能力：能够读懂、设计、改造和实现可控、可观测、可扩展的 Agent Runtime。

## 目录结构

```text
study/
├── README.md                    # 学习工作区说明
├── MISSION.md                   # 学习使命：为什么学 Pi / Agent Harness
├── LEARNING-WORKFLOW.md         # 学习节奏：如何上课、读源码、练习、复盘
├── COURSE-ROADMAP.md            # 30 天路线图与课程分阶段设计
├── SOURCE-READING-MAP.md        # Pi 源码阅读地图
├── EXERCISES.md                 # 阶段性练习与验收标准
├── RESOURCES.md                 # 高信任资源清单
├── GLOSSARY.md                  # 术语表，仅在真正理解后加入术语
├── NOTES.md                     # 教学偏好、临时笔记、待办
├── course-design/               # 课程设计草案，每节课开始前先放这里
├── lessons/                     # 正式课程材料，后续以 HTML 为主
├── reference/                   # 速查表、架构图、参考文档
├── learning-records/            # 学习记录，理解被验证后再写
└── assets/                      # 课程可复用样式、图表、组件
```

## 使用原则

1. **先使命，后课程**：每节课都要服务于 `MISSION.md`。
2. **小切片学习**：每次只解决一个核心问题，例如 Agent Loop、Tool Call、Session、Compaction、Extension。
3. **源码驱动**：课程必须绑定到具体源码路径，而不是只讲抽象概念。
4. **练习验证理解**：能解释不等于掌握，能实现、能改造、能复盘才算掌握。
5. **学习记录不写流水账**：只有当一个概念真正被理解或误区被纠正时，才写入 `learning-records/`。

## 当前学习主线

```text
Agent Loop
  -> Agent State / Message Model
  -> Tool Calling / Tool Execution
  -> AgentHarness
  -> Session / Tree / Compaction
  -> Extension / Skills
  -> Coding Agent CLI / SDK / RPC
  -> 企业级 Agent Harness 设计
```

## 下一步

从 `course-design/0001-pi-agent-loop.md` 开始，进入第一课：读懂一次 `prompt()` 如何进入 Agent Loop，并最终变成模型响应和工具调用。
