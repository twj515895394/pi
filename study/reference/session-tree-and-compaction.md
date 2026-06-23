# Session Tree 与 Compaction

> 状态：待阶段 4 完成后补全。

本文件用于沉淀 Pi 中 session tree、branch、fork、compaction、branch summary 的设计。

## 初始问题

- 为什么 coding agent 的会话不能只是线性 messages？
- 为什么 session entry 需要 `id` 和 `parentId`？
- compaction 为什么要保留 recent messages？
- branch summary 如何让切换分支后仍保留离开分支的重要上下文？

## 待补充

- [ ] Session entry 类型图
- [ ] JSONL session 示例
- [ ] Tree / fork / leaf 关系图
- [ ] Compaction cut point 规则
- [ ] Branch summary 流程图
