# 0010：ResourceLoader / Skills / Prompt Templates

## 本课目标

理解 Pi 如何把项目上下文、技能、提示词模板、系统提示词组织成 Agent 可使用的资源。

## 源码范围

```text
packages/coding-agent/src/core/resource-loader.ts
packages/coding-agent/src/core/skills.ts
packages/coding-agent/src/core/prompt-templates.ts
packages/coding-agent/src/core/system-prompt.ts
packages/coding-agent/docs/skills.md
packages/coding-agent/docs/prompt-templates.md
```

## 核心问题

- AGENTS.md / CLAUDE.md 如何进入上下文？
- SYSTEM.md 与 APPEND_SYSTEM.md 的区别是什么？
- Skill 与 Prompt Template 的边界是什么？
- 为什么项目级资源需要 trust 决策？
- 如何把 Java Debug Skill 设计成可复用能力？

## 小练习

实现一个 Java Debug Skill，并用一个真实 Java 问题验证。

## 复盘产出

```text
study/reference/resource-loader-skills-prompts.md
```
