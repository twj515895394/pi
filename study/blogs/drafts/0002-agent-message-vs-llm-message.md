# AgentMessage vs LLM Message：为什么内部消息和模型消息要分开？

> 系列：从 Dify 到 Agent Runtime：我的 Pi Agent Harness 源码学习笔记  
> 对应课程：0002 AgentMessage vs LLM Message  
> 状态：草稿

## 开篇：消息不就是 messages 吗？

刚开始看 Agent 代码时，我很容易把所有消息都叫做 `messages`。

比如：

```text
用户消息
模型回复
工具结果
系统状态
审计日志
权限判断
UI 提示
上下文摘要
```

它们好像都可以叫“消息”。但真正读 Agent Runtime 源码后，我发现这里有一个非常关键的边界：

```text
内部消息 != 模型消息
```

也就是说，Agent 内部需要保存和流转的消息，和最终发送给 LLM Provider 的消息，不应该混为一谈。

这就是 `AgentMessage` 和 `LLM Message` 的区别。

## 一句话结论

```text
AgentMessage 是内部领域模型；LLM Message 是发给模型接口的 DTO。
```

如果用 Java 后端类比：

```text
AgentMessage ≈ Domain Model
LLM Message ≈ API Request DTO
```

这个类比非常重要。后端系统里，我们不会把数据库领域对象、审计对象、页面展示对象、三方接口 DTO 全部混成一个类。Agent 里也一样。

## 二者的核心区别

| 维度 | AgentMessage | LLM Message |
|---|---|---|
| 所在层级 | Agent Runtime 内部 | LLM Provider 接口层 |
| 表达范围 | 更大 | 更小 |
| 使用者 | Agent、Harness、Session、UI、Extension、审计 | 模型 API |
| 是否可扩展 | 可以扩展自定义消息 | 受模型协议限制 |
| 是否一定进入模型 | 不一定 | 是 |
| 主要职责 | 存储、展示、审计、控制、转换 | 请求模型 |

所以，`AgentMessage` 不是单纯为了调用模型存在的。它是整个 Agent Runtime 的内部消息账本。

## 为什么 Agent 内部需要更大的消息模型？

一个真实 Agent 系统里，除了用户和模型的对话，还会有很多运行状态。

比如金融行业的风控 Agent：

```text
用户问：这笔交易为什么被拦截？
```

系统内部可能同时产生：

```text
用户消息
assistant 思考后的工具调用
交易查询结果
风控策略命中结果
权限系统判断
审计事件
UI 正在查询的状态
合规提示
最终回复
```

这些信息都可能需要被保存、展示、审计或用于后续控制。

但它们不一定都应该进入模型上下文。

例如：

```text
UI 正在查询中 -> 不应该给模型看
审计日志原文 -> 通常不应该给模型看
权限审批详情 -> 默认不应该给模型看
交易状态结论 -> 可能应该简化后给模型看
工具结果 -> 通常应该给模型看
```

这就是为什么需要内部消息模型和 LLM 消息模型分开。

## convertToLlm 是什么？

`convertToLlm` 的职责不是简单类型转换，而是一个边界裁剪过程：

```text
AgentMessage[]
  -> 过滤 / 转换 / 脱敏 / 压缩 / 格式化
  -> LLM Message[]
  -> 调用模型
```

它要回答一个问题：

```text
这一轮模型到底应该看到什么？
```

这比“把 A 类型转成 B 类型”复杂得多。

比如内部有一条权限消息：

```text
permissionDecision: 当前用户无权执行批量退款操作，原因是金额超过审批阈值。
```

它是否要进 LLM？

我的理解是：

```text
原始权限详情：不一定进
简化结论：可以进
```

给模型看的内容可以是：

```text
该批量退款操作已被权限策略拒绝，请向用户解释无法继续执行，并提示走人工审批流程。
```

这样模型知道下一步怎么回复，但不会接触不必要的内部审计细节。

## transformContext 和 convertToLlm 的边界

这两个名字容易混。

我的理解是：

| 函数 | 处理对象 | 主要问题 |
|---|---|---|
| `transformContext` | `AgentMessage[]` | 上下文本身怎么组织？ |
| `convertToLlm` | `AgentMessage[] -> LLM Message[]` | 哪些内容给模型看？ |

简化判断：

```text
要改变上下文本身：transformContext
要决定模型看什么：convertToLlm
```

比如：

```text
历史太长，需要裁剪：transformContext
UI-only 消息不能给模型看：convertToLlm
工具结果太长，需要摘要后给模型：convertToLlm 或 transformContext，取决于是否改变内部账本
给本轮注入额外业务上下文：transformContext
```

## 不是所有消息都应该进入模型

这是本课最重要的工程判断。

可以用一张表判断：

| 消息类型 | 写 Session | 展示用户 | 进入 LLM | 说明 |
|---|---:|---:|---:|---|
| user | 是 | 是 | 是 | 用户意图 |
| assistant | 是 | 是 | 是 | 模型上下文 |
| toolResult | 是 | 可选 | 是 | 工具观察结果 |
| UI 状态 | 可选 | 是 | 否 | 只是展示状态 |
| 审计事件 | 是 | 否 | 默认否 | 服务追踪合规 |
| 权限决策 | 是 | 可选 | 条件进入 | 给结论，不给细节 |
| 业务状态 | 是 | 可选 | 条件进入 | 简化成可推理上下文 |
| 压缩摘要 | 是 | 通常否 | 是 | 保留长期上下文 |

这个判断方式很适合企业 Agent。

## 金融行业 Agent 的例子

假设一个信贷审批 Agent 内部有这些消息：

```text
LoanStatusMessage
AuditEventMessage
PermissionDecisionMessage
UiLoadingMessage
ToolResultMessage
```

它们的处理方式应该不同：

```text
LoanStatusMessage：可以简化后进入 LLM，用于解释审批进度。
AuditEventMessage：写入审计，不进入 LLM。
PermissionDecisionMessage：如果影响下一步行为，可以给模型一个简化结论。
UiLoadingMessage：只给 UI 展示，不进入 LLM。
ToolResultMessage：通常进入 LLM，让模型基于工具结果继续推理。
```

这就是内部消息模型的价值。

如果所有消息都直接发给 LLM，会带来几个问题：

```text
上下文噪声变多
token 成本升高
敏感信息风险增加
模型可能被无关状态误导
内部审计信息泄露给模型
```

## 少量伪代码理解

可以把转换逻辑理解成：

```ts
function convertToLlm(messages: AgentMessage[]): LlmMessage[] {
  return messages.flatMap(message => {
    if (message.role === 'user') return toUserMessage(message);
    if (message.role === 'assistant') return toAssistantMessage(message);
    if (message.role === 'toolResult') return toToolResultMessage(message);
    if (message.role === 'uiStatus') return [];
    if (message.role === 'auditEvent') return [];
    if (message.role === 'businessStatus') return summarizeForModel(message);
    return [];
  });
}
```

这段不是源码，只是表达思想：

```text
内部消息可以很多，但模型只看它该看的。
```

## 常见误区

### 误区 1：所有内部状态都给模型看，模型会更聪明

不一定。上下文不是越多越好。无关状态会制造噪声，敏感状态会制造风险。

### 误区 2：审计事件可以直接进入 LLM

通常不应该。审计事件服务的是合规、追踪和风控，不是模型推理。只有审计结论会影响模型下一步行为时，才应该转换成简化结论。

### 误区 3：AgentMessage 只是 LLM Message 的别名

不是。`AgentMessage` 是 Runtime 内部领域模型，范围比 LLM Message 大很多。

## 我的阶段性理解

第二课对我最大的启发是：Agent 工程化里，“消息”不是一个简单数组。

它至少要承担四种职责：

```text
模型推理上下文
用户可见对话
系统运行状态
审计和控制信息
```

如果这些职责都混成一种 LLM Message，系统后面会很难扩展。

所以，成熟 Agent Runtime 一定要有内部消息模型，并且在模型调用边界做转换。

## 小结

本文可以压缩成一句话：

```text
AgentMessage 是内部完整账本，LLM Message 是模型接口 DTO；真正调用模型前，必须通过 convertToLlm 决定模型应该看到什么。
```

这件事看起来只是类型设计，实际上影响的是整个 Agent 系统的安全、成本、上下文质量和可扩展性。
