# 0006：Extension System

## 本课目标

理解 Pi 的 extension 如何把底层 Agent Runtime 扩展成可定制的 coding agent。

## 源码范围

```text
packages/coding-agent/src/core/extensions/index.ts
packages/coding-agent/src/core/extensions/types.ts
packages/coding-agent/src/core/extensions/runner.ts
packages/coding-agent/docs/extensions.md
packages/coding-agent/examples/extensions/
```

## 核心问题

- Extension 能注册哪些能力？
- `pi.on()` 监听的事件来自哪里？
- 自定义工具如何进入 Agent？
- 审计、状态持久化、UI 交互分别应该放在哪里？
- Extension 与 MCP / Dify 插件有哪些相似点和差异？

## 小练习

实现一个工具调用审计 extension。

## 复盘产出

```text
study/reference/extension-system-map.md
```
