# 0005：Session Tree 与 Compaction

## 本课目标

理解 Pi 如何处理长会话、多分支和上下文压缩。

## 源码范围

```text
packages/agent/src/harness/session/*
packages/agent/src/harness/compaction/*
packages/coding-agent/src/core/session-manager.ts
packages/coding-agent/docs/session-format.md
packages/coding-agent/docs/compaction.md
```

## 核心问题

- Session 为什么是 tree，而不是简单数组？
- `id` / `parentId` 如何形成可导航历史？
- Compaction 何时触发？
- 为什么不能切在 tool result 上？
- Branch summary 解决什么问题？

## 小练习

构造一个包含 user、assistant、toolResult、compaction、branch_summary 的 session tree 示例。

## 复盘产出

```text
study/reference/session-tree-and-compaction.md
```
