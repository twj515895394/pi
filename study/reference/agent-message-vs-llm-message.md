# AgentMessage vs LLM Message

本参考资料沉淀第二课 `0002：AgentMessage vs LLM Message` 的核心结论。

## 一句话结论

`AgentMessage` 是 Pi Agent 内部的完整消息账本，服务 Agent Loop、Harness、Session、UI、Extension、审计和业务状态；`LLM Message` 是每次模型调用前由 `convertToLlm` 生成的模型可消费消息，是面向 LLM Provider 的接口 DTO。

## 核心区别

| 维度 | AgentMessage | LLM Message |
|---|---|---|
| 所在层级 | Agent Runtime 内部领域模型 | LLM Provider 接口协议 |
| 范围 | 更大，包含标准消息和自定义消息 | 更小，只包含模型能理解的消息 |
| 使用者 | Agent / Harness / UI / Session / Extension | LLM API |
| 是否可扩展 | 可以通过 `CustomAgentMessages` 扩展 | 受模型协议约束 |
| 是否一定进入 LLM | 不一定 | 是 |
| 典型处理 | 存储、展示、审计、转换、压缩 | 请求模型 |

## 源码定义

`AgentMessage` 在基础层是：

```ts
export type AgentMessage = Message | CustomAgentMessages[keyof CustomAgentMessages];
```

这表示它是标准 LLM message 与应用自定义消息的联合类型。

基础 `defaultConvertToLlm` 只保留：

```text
user
assistant
toolResult
```

Harness 层通过 declaration merging 扩展了：

```text
bashExecution
custom
branchSummary
compactionSummary
```

## convertToLlm 的职责

`convertToLlm` 不是简单的类型转换，而是边界裁剪和协议适配：

```text
AgentMessage[]
  -> filter / convert / redact / summarize / format
  -> Message[]
  -> LLM
```

它负责决定：

- 哪些内部消息允许进入模型上下文。
- 哪些内部消息必须过滤。
- 哪些内部消息要转换成 user message。
- 哪些内部消息需要脱敏、压缩或格式化。

## transformContext 与 convertToLlm 的边界

| 函数 | 处理对象 | 典型职责 |
|---|---|---|
| `transformContext` | `AgentMessage[]` | 裁剪历史、压缩上下文、注入外部信息、重排上下文 |
| `convertToLlm` | `AgentMessage[] -> Message[]` | 过滤 UI-only 消息、转换 custom 消息、脱敏、格式化为 LLM 协议 |

简化判断：

```text
要改变上下文本身：transformContext
要决定模型看什么：convertToLlm
```

## 消息模型对照表

| 消息类型 | 是否写 session | 是否展示用户 | 是否进入 LLM | 是否影响工具调用 |
|---|---:|---:|---:|---:|
| user | 是 | 是 | 是 | 可能 |
| assistant | 是 | 是 | 是 | 可能 |
| toolResult | 是 | 可选 | 是 | 是 |
| bashExecution | 是 | 可选 | 条件进入 | 可能 |
| custom | 是 | 由 display 决定 | 通常转换后进入 | 视内容而定 |
| branchSummary | 是 | 通常否 | 是，转换为 user message | 间接影响 |
| compactionSummary | 是 | 通常否 | 是，转换为 user message | 间接影响 |
| uiStatus | 可选 | 是 | 否 | 否 |
| auditEvent | 是 | 通常否 | 默认否 | 可用于审计 |
| permissionDecision | 是 | 可选 | 默认否 | 是 |
| businessStatus | 是 | 可选 | 条件进入，通常转换后进入 | 可能 |

## 企业 Agent 判断规则

设计一类消息是否进入 LLM 时，先问四个问题：

```text
1. 是否写入 session？
2. 是否展示给用户？
3. 是否进入 LLM？
4. 是否影响工具调用或控制流？
```

再补三个风险判断：

```text
1. 是否包含敏感信息？
2. 是否会制造上下文噪声？
3. 是否会增加不必要的 token 成本？
```

## 企业消息设计示例

### ClearanceStatusMessage

清关状态消息通常可以进入 LLM，但不建议原样进入。

更好的做法是转换成清晰、短小、可推理的上下文：

```text
当前运单 BUFFALO123 的清关状态：
- 当前节点：待上传税单
- 渠道：SA_CUSTOMS
- 税单状态：未上传
- 最近更新时间：2026-06-23 10:20
请基于该状态向用户解释下一步。
```

### AuditEventMessage

审计事件默认不进入 LLM。它服务的是审计、追踪、风控和合规。

不应进入 LLM：

```text
用户 tang 在 10:33:01 通过权限系统审批了 commandId=xxx。
```

可以给 LLM 的简化结论：

```text
该高风险命令已被策略拒绝，请不要继续沿此方向执行。
```

## 本课关键理解

1. `AgentMessage` 是内部领域模型，`LLM Message` 是接口 DTO。
2. 内部消息可以比模型消息包含更多事实、状态、审计、UI 和业务上下文。
3. 不是所有内部消息都该进入 LLM；进入前要经过过滤、转换、脱敏或压缩。
4. `branchSummary` 和 `compactionSummary` 虽然是内部消息，但代表模型继续推理所需的历史上下文，所以需要转换后进入 LLM。
5. 审计和权限消息默认不进 LLM；只有会影响模型下一步行为的简化结论才应进入。
