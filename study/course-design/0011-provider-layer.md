# 0011：Provider Layer

## 本课目标

理解 Pi 如何管理模型供应商、模型选择和请求参数。

## 源码范围

```text
packages/coding-agent/src/core/model-registry.ts
packages/coding-agent/src/core/model-resolver.ts
packages/ai/src/*
```

## 核心问题

- 模型注册表负责什么？
- 模型选择如何和 session、settings 结合？
- 企业平台中如何设计多模型网关与 Agent Runtime 的边界？

## 小练习

画出模型选择流程图。
