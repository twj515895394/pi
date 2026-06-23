# SDK / RPC / CLI 模式

> 状态：待课程 0009 完成后补全。

## 初始对照

| 模式 | 适合场景 | 主要边界 |
|---|---|---|
| interactive | 人直接使用终端 | TUI 与键盘交互 |
| print | 一次性命令 | 输入输出简单 |
| json | 事件流消费 | 机器可读事件 |
| rpc | 非 Node 进程集成 | 标准输入输出协议 |
| sdk | Node 应用内嵌 | 直接调用 TypeScript API |

## 待补充

- [ ] `AgentSessionRuntime` 职责
- [ ] session replacement 机制
- [ ] RPC framing
- [ ] Java 平台接入方案
- [ ] 与 MCP server 集成的可能路径
