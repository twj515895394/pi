# 0008：Coding Tools：read / bash / edit / write

## 本课目标

理解一个 coding agent 为什么最小能力通常围绕 read、bash、edit、write 展开，并分析每类工具的职责和风险。

## 源码范围

```text
packages/coding-agent/src/core/tools/index.ts
packages/coding-agent/src/core/tools/read.ts
packages/coding-agent/src/core/tools/bash.ts
packages/coding-agent/src/core/tools/edit.ts
packages/coding-agent/src/core/tools/write.ts
packages/coding-agent/src/core/tools/grep.ts
packages/coding-agent/src/core/tools/find.ts
packages/coding-agent/src/core/tools/ls.ts
```

## 核心问题

- read / grep / find / ls 为什么属于信息获取能力？
- edit / write 为什么属于文件变更能力？
- bash 为什么既强大又危险？
- 工具描述如何影响模型调用行为？
- 如果迁移到企业 Java 平台，哪些工具应该默认只读，哪些需要审批？

## 小练习

设计一张 Coding Agent 工具风险矩阵。

## 复盘产出

```text
study/reference/coding-tools-risk-matrix.md
```
