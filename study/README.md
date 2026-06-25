# Pi Agent Harness 学习工作区

这是 Pi 项目中的个人学习工作区，用于持续学习底层 Agent Runtime / Agent Harness 工程。

任何 Agent 或新会话进入本目录后，应把本文件当作主入口。

## 这个工作区是干什么的

目标是把已有的 Dify / Java + Dify 接口型 Agent 经验，升级为能够读懂、设计、改造和实现 Agent Harness 的工程能力。

学习对象不是 Pi 的全部代码，而是围绕 Agent 工程主线切片学习：

```text
Agent Loop
  -> AgentMessage / LLM Message
  -> Tool Call
  -> AgentHarness
  -> Session / Compaction
  -> Extension / Skills
  -> Coding Agent CLI / SDK / RPC
  -> 企业级 Agent Harness 设计
```

## 学习主计划

`study/MASTER-PLAN.md` 是唯一权威学习主计划。

它定义：

```text
学习总目标
阶段路线
课程清单
阶段验收物
动态补课规则
高概念密度课程教学规则
当前执行方式
```

其他规划文件都应服务于 `MASTER-PLAN.md`：

```text
COURSE-ROADMAP.md      阶段路线图
COURSE-LIST.md         课程清单
SOURCE-READING-MAP.md  源码阅读地图
PROGRESS.md            当前进度看板
CURRENT.md             新会话续接入口
```

## 博文创作模块

学习过程中，每完成一段完整知识点课程，可以沉淀对应博文草稿，用于强化记忆和对外分享。

博客模块位于：

```text
study/blogs/
```

其中：

```text
study/blogs/README.md                 # 博文索引
study/blogs/WRITING-GUIDELINES.md     # 博文写作规范
study/blogs/drafts/                   # 草稿目录
study/blogs/published/                # 后续发布版目录，可按需创建
```

写作要求：

```text
默认一课一篇；
不暴露真实企业行业方向；
企业级类比统一使用泛化的金融行业 Agent；
代码片段可以有，但不能太多；
文章要包含结论、结构化图表、工程类比和阶段性理解。
```

## 新会话应该怎么继续

请按顺序阅读：

```text
study/README.md
study/MASTER-PLAN.md
study/MISSION.md
study/CURRENT.md
study/PROGRESS.md
最新的 study/handoffs/*.md
```

如果要继续写博文，还应阅读：

```text
study/blogs/README.md
study/blogs/WRITING-GUIDELINES.md
```

其中：

- `MASTER-PLAN.md`：唯一权威学习主计划。
- `CURRENT.md`：当前学习状态入口。
- `PROGRESS.md`：全局课程进度表。
- `handoffs/`：每节课结束后的交接文档。
- `blogs/`：学习博文草稿与写作规范。

## 目录结构

```text
study/
├── README.md                    # 工作区主入口
├── MASTER-PLAN.md               # 唯一权威学习主计划
├── MISSION.md                   # 学习使命
├── CURRENT.md                   # 当前学习状态，新会话优先读
├── PROGRESS.md                  # 全局学习进度总表
├── HANDOFF-PROTOCOL.md          # 每课 handoff 规则
├── LEARNING-WORKFLOW.md         # 学习节奏
├── COURSE-ROADMAP.md            # 阶段路线图
├── COURSE-LIST.md               # 课程清单
├── SOURCE-READING-MAP.md        # 源码阅读地图
├── EXERCISES.md                 # 阶段练习
├── RESOURCES.md                 # 高信任资源
├── GLOSSARY.md                  # 术语表，理解后再写入
├── NOTES.md                     # 偏好和临时笔记
├── blogs/                       # 学习博文草稿与写作规范
├── handoffs/                    # 每课交接文档
├── course-design/               # 课程草案
├── lessons/                     # 正式课程材料，HTML 为主
├── reference/                   # 复习用参考资料
├── learning-records/            # 被验证后的学习记录
├── templates/                   # 模板
├── exercises/                   # 练习实现与复盘
└── assets/                      # 课程复用资产
```

## 学习流程

```text
课程设计
  -> 源码精读
  -> 概念解释
  -> 小练习
  -> 理解检查
  -> 更新 reference
  -> 更新 progress/current/handoff
  -> 必要时写 learning-record
  -> 完整知识点完成后可写博客草稿
```

## 每节课结束后的强制更新

每节课结束后必须更新：

```text
study/PROGRESS.md
study/CURRENT.md
study/handoffs/<lesson-id>.md
study/reference/<对应参考资料>.md
```

只有当用户真正展示理解时，才写入：

```text
study/learning-records/<next-id>-<slug>.md
```

如果本课或本阶段要写博文，则更新：

```text
study/blogs/README.md
study/blogs/drafts/<blog-slug>.md
```

## 当前下一步

当前下一步以 `study/CURRENT.md` 为准。课程主线以 `study/MASTER-PLAN.md` 为准。