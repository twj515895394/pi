# AgentHarness 为什么不是 Agent 的简单封装？

> 系列：从 Dify 到 Agent Runtime：我的 Pi Agent Harness 源码学习笔记  
> 对应课程：0004 AgentHarness 是 Runtime 控制层  
> 状态：草稿

## 开篇：从 AgentLoop 到 AgentHarness，复杂度突然上来了

学完 Agent Loop、消息模型和 Tool Call 执行链路后，我一开始以为 `AgentHarness` 只是把 `Agent` 再包一层，方便外部调用。

但真正读到 Harness 这一层时，会发现它完全不是简单 wrapper。

它更像一个完整的运行容器：

```text
Session
Model
Tools
Resources
Queues
Hooks
Provider
Events
Compaction
Branch Summary
UI / CLI / SDK
```

这些都不是一次模型调用能解释的。

所以第四课最重要的结论是：

```text
AgentHarness 是 Runtime 控制层，不是 Agent 的简单封装。
```

## 一句话结论

```text
AgentLoop 是执行内核，Agent 是带状态的调用门面，AgentHarness 是把执行内核接入真实应用运行环境的 Runtime 控制层。
```

## 三层职责边界

| 层级 | 类比 Java 后端 | 核心职责 |
|---|---|---|
| AgentLoop | 核心工作流函数 | 当前 run 内的模型调用、toolCall 检测、工具执行、toolResult 回灌 |
| Agent | 有状态业务 Service | 保存 messages、tools、streaming 状态，并调用 AgentLoop |
| AgentHarness | 应用运行容器 / Runtime 控制层 | 管 Session、模型、工具、资源、队列、Hook、Provider、Compaction、Branch Summary 和事件 |

这张表是理解 Harness 的关键。

如果说 AgentLoop 关心的是：

```text
这一轮怎么跑？
```

那么 AgentHarness 关心的是：

```text
这个 Agent 应用怎么长期、稳定、可治理地运行？
```

## Runtime 控制层到底控制什么？

一个真实 Agent 应用里，光有模型和工具不够。还需要控制：

```text
用户请求怎么进入
本轮带哪些上下文
当前用哪个模型
启用哪些工具
工具调用前是否允许
工具结果是否要脱敏
消息怎么落库
上下文太长怎么压缩
运行中用户插话怎么排队
UI 怎么知道运行状态
Provider 请求怎么审计
```

这些能力全部塞进 AgentLoop 会让它变成巨无霸。

更合理的是：

```text
AgentLoop 保持小而纯；
AgentHarness 负责把 Runtime 能力注入进去。
```

## Harness 持有的不只是 messages

Harness 内部要管理很多状态，例如：

| 状态 | 说明 |
|---|---|
| env | 执行环境 |
| session | 会话和会话树 |
| models / model | 模型集合与当前模型 |
| thinkingLevel | 当前推理强度 |
| systemPrompt | 系统提示词 |
| streamOptions | Provider 请求参数 |
| resources | skills、promptTemplates |
| tools / activeToolNames | 全部工具与本轮启用工具 |
| phase | 当前运行阶段 |
| steerQueue / followUpQueue / nextTurnQueue | 运行中插话和后续消息队列 |
| handlers | 事件和 Hook 订阅 |
| pendingSessionWrites | 运行中暂存的 session 写入 |

这说明 Harness 管的是 Agent 应用运行环境，而不是单个函数调用。

## createTurnState：本轮运行快照

每次真正执行前，Harness 会先组装一个 TurnState。

可以理解成：

```text
本轮 Agent 运行前的完整快照
```

它包括：

```text
session context
resources
session metadata
tools / activeTools
systemPrompt
model
thinkingLevel
streamOptions
```

然后再通过 `createContext()` 把它压缩成 AgentLoop 需要的最小上下文：

```text
systemPrompt
messages
tools
```

这体现了一个很好的分层：

```text
Harness 的状态很丰富；
AgentLoop 的输入要尽量简单。
```

## createLoopConfig：真正的适配器

我觉得 `createLoopConfig()` 是理解 Harness 的核心。

它不是普通配置对象，而是一个适配器 / 注入器。

它把 Harness 的能力注入 AgentLoop：

| 注入点 | Harness 能力 |
|---|---|
| convertToLlm | 消息转换 |
| transformContext | 上下文治理 |
| beforeToolCall | 工具执行前权限、审批、风控 |
| afterToolCall | 工具结果脱敏、压缩、错误标准化 |
| prepareNextTurn | flush session writes，重建 TurnState |
| getSteeringMessages | 运行中插话队列 |
| getFollowUpMessages | 后续消息队列 |

所以它解决的问题是：

```text
AgentLoop 不直接依赖 Harness，
但 Harness 可以通过配置和 Hook 影响 AgentLoop 的关键行为。
```

这就是低耦合的设计。

## Provider 请求治理

Harness 还控制模型请求边界。

在真正调用模型前后，可以有：

```text
before_provider_request
before_provider_payload
after_provider_response
```

这在企业里很重要。

金融行业 Agent 里，模型请求可能需要：

```text
注入 traceId
注入 tenantId
记录 provider response
控制 timeout / retry
增加 request metadata
审计 prompt payload
根据用户等级选择模型
```

这些都不应该写进 AgentLoop。

## Harness 事件：UI 为什么应该关心 Harness 而不是只关心 AgentLoop？

UI 不只是展示模型文本。

它还要展示：

```text
模型正在请求中
工具正在执行中
工具执行完成
队列更新
会话已保存
运行已结束
模型切换
工具集合变化
资源变化
```

AgentLoop 事件能描述底层执行过程，但 Harness 事件能描述完整应用状态。

所以如果做前端 SSE 推送，UI 更应该订阅 Harness 统一事件，而不是只盯 AgentLoop 内核。

## Session 也不是简单聊天记录

Harness 层还引出了下一个重要主题：Session Tree。

Session 里不仅有 message，还可能有：

```text
model_change
thinking_level_change
active_tools_change
compaction
branch_summary
custom_message
label
leaf
```

这说明 Session 是 Agent 的运行历史树，而不是简单聊天 transcript。

这也是为什么 Session、Compaction、Branch Summary 不适合放在 AgentLoop 里。

它们属于长期状态治理。

## 金融行业 Agent 的类比

假设做一个金融风控 Agent。

AgentLoop 只负责：

```text
模型判断用户问题
模型提出工具调用
Runtime 执行查询或分析工具
工具结果回灌给模型
模型生成回复
```

但 Harness 要负责：

```text
用户身份和权限
当前会话上下文
可用工具集合
是否允许查询该账户或交易
Provider 请求审计
工具结果脱敏
运行事件推送
消息落库
上下文压缩
模型切换记录
```

也就是说，真正企业级 Agent 的工程复杂度，大部分都在 Harness 层。

## 常见误区

### 误区 1：AgentHarness 就是 Agent 的高级封装

不准确。它不是把 Agent 包一下，而是把 AgentLoop 接入完整 Runtime 环境。

### 误区 2：权限和审计可以直接写进 Tool

工具内部可以兜底，但统一权限、审计、风控更适合挂在 Harness Hook 上。这样低耦合、可复用、可统一治理。

### 误区 3：UI 只需要模型消息流

真实产品里，UI 还需要工具状态、队列状态、保存状态、运行结束状态。这些都需要 Harness 事件体系支撑。

## 我的阶段性理解

学完第四课后，我对 Agent 工程化的理解变成了四层：

```text
AgentLoop：执行内核
Message Model：内部状态和模型接口边界
Tool Pipeline：工具安全执行流水线
AgentHarness：应用级 Runtime 控制层
```

前面三层解决“Agent 怎么跑”。

Harness 解决的是：

```text
Agent 怎么在真实应用里长期、稳定、可治理地运行。
```

这是从 Demo Agent 到企业 Agent 的关键分水岭。

## 小结

本文可以压缩成一句话：

```text
AgentHarness 不是 Agent 的简单封装，而是把 AgentLoop 接入 Session、模型、工具、权限、事件、Provider、队列和长期上下文治理的 Runtime 控制层。
```

理解 Harness 后，再去看 Session Tree、Compaction、Extension System，会更容易明白为什么 Agent 工程化不能只停留在 Prompt 和 Tool List。
