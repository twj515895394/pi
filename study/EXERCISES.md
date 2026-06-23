# 阶段练习

练习的目标不是为了写功能而写功能，而是验证是否真正理解 Agent Harness 的工程设计。每个练习都应该有源码阅读、实现、测试、复盘四步。

## 练习 1：Tool Audit Extension

### 对应课程

- 0003 Tool Call 执行链路
- 0004 beforeToolCall / afterToolCall
- 0009 Extension System

### 目标

实现一个 extension，记录每次工具调用和工具结果，形成可审计的 JSONL 文件。

### 功能要求

```text
.pi/audit/tool-calls.jsonl
```

每行记录：

```json
{
  "timestamp": 0,
  "sessionId": "",
  "toolName": "read",
  "phase": "call | result",
  "input": {},
  "isError": false,
  "details": {}
}
```

### 需要理解

- `tool_call` 事件何时触发。
- `tool_result` 事件何时触发。
- 工具输入和工具结果分别能拿到哪些信息。
- 审计日志为什么不能只依赖最终 assistant message。

### 验收标准

- read / bash / edit / write 至少各能记录一次。
- 失败工具调用也能记录。
- 记录中能区分 call 与 result。
- 能说明这个 extension 在企业 Agent 平台中的价值。

---

## 练习 2：Bash Permission Extension

### 对应课程

- 0004 beforeToolCall / afterToolCall
- 0008 Coding Tools：read/bash/edit/write
- 0009 Extension System

### 目标

实现一个权限拦截 extension，对危险 bash 命令进行确认或阻断。

### 拦截规则示例

```text
rm -rf
sudo
curl ... | sh
wget ... | sh
修改 .env
删除 node_modules
删除 package-lock.json
修改 ~/.ssh
```

### 功能要求

- 监听 `tool_call`。
- 识别 bash 工具。
- 对危险命令弹出确认。
- 用户拒绝时返回 `{ block: true, reason: "..." }`。
- 将阻断事件写入 audit 日志。

### 需要理解

- 为什么权限控制应发生在工具执行之前。
- 为什么 bash 是 coding agent 中风险最高的工具。
- 规则拦截和用户确认各自适合什么场景。

### 验收标准

- 普通 `ls` / `pwd` 能执行。
- 危险命令会触发确认。
- 拒绝后工具不会执行。
- 能说明这和 Dify 工具节点权限治理的差异。

---

## 练习 3：Plan Mode Extension

### 对应课程

- 0005 AgentHarness 是什么
- 0009 Extension System
- 0010 企业级 Agent Harness 设计

### 目标

实现一个最小计划模式，让 Agent 先写计划，再由用户确认是否执行。

### 功能设想

```text
/plan <需求>
  -> 要求模型只输出计划，不调用工具
  -> 写入 PLAN.md
  -> 用户确认
  -> /execute-plan 执行计划
```

### 需要理解

- 为什么 plan mode 是执行控制问题，而不仅是 prompt 文案问题。
- 如何通过 command 实现用户主动触发。
- 如何限制或切换 active tools。
- 如何把计划写入 session 或项目文件。

### 验收标准

- `/plan` 能生成计划并写入文件。
- 未确认前不执行修改型工具。
- `/execute-plan` 能读取计划并进入执行。
- 能说明它和 Claude Code / Codex 类工具中的 plan 模式有什么相似点和差异。

---

## 练习 4：Java Debug Skill

### 对应课程

- 0002 AgentMessage vs LLM Message
- 0008 Coding Tools：read/bash/edit/write
- 0009 Extension System
- 0010 企业级 Agent Harness 设计

### 目标

把个人 Java 后端问题排查经验封装成可复用 Skill。

### Skill 场景

```text
Spring Boot 启动失败
MyBatis SQL 异常
Dify API 调用异常
HttpClient 请求异常
Redis / Kafka / MQ 连接问题
JVM 内存与 GC 问题
```

### 建议步骤

1. 先读 README、pom.xml、build.gradle、配置文件。
2. 再看日志和异常栈。
3. 再定位请求链路、配置绑定、环境变量、依赖版本。
4. 必要时执行只读命令。
5. 最后给出最小改动方案。

### 验收标准

- Skill 有明确适用场景。
- 能指导 Agent 先读信息再下结论。
- 能避免一上来乱改代码。
- 能结合至少一个真实 Java 问题做验证。

---

## 练习复盘模板

每个练习完成后，在 `study/learning-records/` 中视情况记录关键认知，而不是记录过程流水账。

复盘问题：

```text
1. 这个练习验证了哪个 Agent Harness 概念？
2. 哪个源码设计和我原来的 Dify / Java 经验最不同？
3. 如果迁移到企业 Java 平台，应该保留什么，改造什么？
4. 这个练习暴露了我还没理解的点是什么？
```
