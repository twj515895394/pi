# Tool Call 不是函数调用：模型提出工具调用后，Runtime 做了什么？

> 系列：从 Dify 到 Agent Runtime：我的 Pi Agent Harness 源码学习笔记  
> 对应课程：0003 Tool Call 执行链路  
> 状态：草稿

## 开篇：模型说要调用工具，就能直接执行吗？

在很多 Agent Demo 里，工具调用看起来很简单：

```text
模型输出 toolCall
系统找到函数
执行函数
把结果返回
```

但真正进入 Agent Runtime 的源码后，会发现这件事远没有这么简单。

原因也很直接：

```text
LLM 产生的 toolCall 不是可信命令。
```

模型可以提出“我要调用某个工具”，但它不能决定这个工具是否真的应该执行。真正决定工具能不能执行、怎么执行、失败怎么处理、结果怎么回灌的是 Agent Runtime。

## 一句话结论

```text
LLM 只能提出 toolCall；Runtime 才决定工具能不能执行、如何执行、结果如何治理，并把它包装成 toolResult message 回灌给模型。
```

## Tool Call 的完整生命周期

可以把一次工具调用理解成下面这条流水线：

```text
assistant message contains toolCall
  -> executeToolCalls()
  -> choose sequential / parallel
  -> emit tool_execution_start
  -> prepareToolCall()
      -> find tool
      -> prepareArguments
      -> validateToolArguments
      -> beforeToolCall
  -> executePreparedToolCall()
      -> tool.execute()
      -> optional tool_execution_update
  -> finalizeExecutedToolCall()
      -> afterToolCall
  -> emit tool_execution_end
  -> createToolResultMessage()
  -> push toolResult back to messages
  -> next LLM turn
```

这条链路说明一个重点：

```text
toolCall 到 tool.execute 之间，有一整套 Runtime 管控层。
```

不是模型说执行就执行。

## 为什么不能直接 tool.execute？

因为模型输出可能存在很多问题：

```text
工具名不存在
参数格式不符合要求
参数语义不合法
用户没有权限
业务状态不允许流转
工具调用风险过高
模型误解了用户意图
工具结果可能包含敏感信息
```

如果直接执行，会把模型的不确定性直接放大成系统风险。

在金融行业 Agent 里，这个问题尤其明显。

比如用户说：

```text
帮我看看这笔交易为什么失败
```

模型可能误生成一个危险工具调用：

```text
refundTransaction(transactionId)
```

如果系统直接执行，就可能从“查询交易失败原因”变成“发起退款操作”。

所以 Runtime 必须在工具执行前加关口。

## prepareArguments 和 validateToolArguments

工具执行前首先要处理参数。

这两个概念容易混：

| 阶段 | 作用 | 类比 Java |
|---|---|---|
| `prepareArguments` | 对模型输出的 raw arguments 做整理、兼容、修正、标准化 | 参数适配器 / 参数清洗 |
| `validateToolArguments` | 按工具 schema 做正式校验，不合法就失败 | Bean Validation / Schema 校验 |

举个例子：

```text
模型给出：
transaction_id: " abc-123 "

prepareArguments：
去空格、转字段名、标准化成 transactionId

validateToolArguments：
检查 transactionId 是否符合交易 ID 格式
```

一个负责“整理”，一个负责“判定是否合法”。

## beforeToolCall：执行前的权限和风控

`beforeToolCall` 的职责很明确：

```text
决定这个工具调用能不能执行。
```

它适合做：

```text
用户权限检查
工具白名单检查
高风险操作审批
业务状态校验
参数越权检测
命令安全检查
调用频率限制
```

在金融行业 Agent 里，典型例子是：

```text
查询账户余额：低风险，可执行
查询本人交易记录：需要身份确认
修改授信额度：高风险，需要人工审批
批量退款：高风险，默认拦截
导出客户明细：敏感数据，需要合规检查
```

所以 `beforeToolCall` 不是普通拦截器，而是企业 Agent 的权限与风控入口。

## tool.execute：真正执行工具

只有工具存在、参数准备完成、参数校验通过、beforeToolCall 没有拦截后，才会进入真正执行。

可以用伪代码理解：

```ts
const prepared = prepareToolCall(toolCall);
if (prepared.blocked) {
  return createErrorToolResult(prepared.reason);
}

const result = await tool.execute(args);
return finalizeExecutedToolCall(result);
```

这段代码不是源码，只是帮助理解流程。

真实 Runtime 里还要处理 abort signal、stream update、异常捕获、事件通知等。

## afterToolCall：执行后的结果治理

工具执行完，并不代表结果可以原样给模型或用户看。

`afterToolCall` 负责：

```text
脱敏
压缩
错误标准化
结果改写
审计补充
是否 terminate
```

在金融行业 Agent 里，工具可能返回：

```text
客户完整手机号
身份证号
银行卡号
内部风控策略 ID
模型不应该看到的系统字段
接口异常堆栈
```

这些都应该在结果进入模型上下文前处理。

简化原则是：

```text
beforeToolCall 决定能不能执行。
afterToolCall 决定结果如何给模型和用户看。
```

## 工具失败为什么也要变成 toolResult？

这是我觉得很重要的一个设计点。

工具失败不应该只写日志，也不应该静默跳过，而是应该变成 error toolResult 回灌给模型。

因为模型需要知道：

```text
工具没有成功
失败原因是什么
是否可以换工具
是否应该向用户解释失败
是否应该停止继续尝试
```

比如：

```text
工具结果：权限不足，无法查询该交易明细。
```

模型看到后可以回复：

```text
我无法直接查询这笔交易的详细风控信息，因为当前权限不足。你可以联系有权限的审核人员，或提供可公开查询的交易状态信息。
```

如果只是后台日志记录，模型就会以为工具没有返回，可能继续胡乱尝试。

## sequential 和 parallel：为什么执行顺序也要管？

工具调用有两种执行模式：顺序和并行。

| 模式 | 行为 | 适合场景 |
|---|---|---|
| sequential | 一个工具完整执行完，再执行下一个 | 修改状态、写文件、有顺序依赖 |
| parallel | 多个工具可以并发执行 | 多个独立查询、只读检索 |

金融行业 Agent 示例：

```text
并行适合：同时查询用户基础信息、贷款申请状态、最近交易摘要。
顺序适合：先校验权限，再更新审批状态，再记录审计日志。
```

并行时还有一个细节：

```text
UI 事件可以按工具真实完成顺序展示；
toolResult message 最好按模型原始 toolCall 顺序回灌。
```

这样既能让 UI 及时反馈，又能保持模型上下文稳定。

## 企业级 Tool Permission Layer 检查清单

如果做企业 Agent，工具执行前至少要检查：

```text
用户身份：userId、角色、租户、岗位
工具权限：是否能调用该工具
操作类型：读、写、审批、删除、外部调用
资源范围：账户、交易、客户、产品、审批单
数据归属：是否有权访问这个对象
参数风险：是否越权、是否扩大范围
上下文一致性：是否符合用户原始意图
审批状态：是否需要人工确认
风控策略：黑白名单、频率限制、敏感字段
审计信息：traceId、toolCallId、sessionId、riskLevel
```

这个检查清单比单纯的 if 判断要完整得多。

## 常见误区

### 误区 1：toolCall 就是函数调用

不是。toolCall 是模型提出的请求，函数调用是 Runtime 审核通过后的执行动作。

### 误区 2：工具失败就是异常

对 Runtime 来说，工具失败也应该成为模型可观察的结果。它不只是异常，而是下一轮推理的重要上下文。

### 误区 3：权限应该写进每个工具内部

部分权限可以在工具内部兜底，但企业级权限治理更适合放在统一的 tool_call hook 或 Tool Permission Layer，便于审计、复用和扩展。

## 我的阶段性理解

第三课让我意识到：工具调用是 Agent 工程化里风险最高的环节之一。

因为模型从“说话”变成了“请求执行动作”。

一旦涉及动作，就会有：

```text
权限
审批
参数
状态
风险
审计
失败处理
结果治理
```

所以一个成熟 Agent Runtime 必须把 toolCall 变成可控流水线，而不是简单函数分发。

## 小结

本文可以压缩成一句话：

```text
Tool Call 不是函数调用，而是模型提出的执行请求；真正执行前后必须经过 Runtime 的参数、权限、风控、结果治理和消息回灌。
```

理解这一点后，再看 AgentHarness，就会更容易明白为什么真实 Agent 需要一个完整 Runtime 控制层。
