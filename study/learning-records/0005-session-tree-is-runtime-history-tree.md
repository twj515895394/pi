# Session Tree 是 Agent Runtime 的运行历史树

用户已经能够区分 Session Tree、Compaction 与 Branch Summary 的职责边界：Session 不是简单 `messages[]`，而是通过 `id` / `parentId` / `leaf` 组织 branch path 的运行历史树；Compaction 处理当前 branch 太长的问题；Branch Summary 处理从 old branch 切到 new branch 时，如何把旧路径的关键经验带到新路径。这个理解解锁后续 Extension System 与企业级 Agent Harness 设计，因为后续课程可以直接把 Session 视为 Runtime 状态治理层，而不是聊天记录存储。

**Evidence**：用户明确指出“Compaction 处理的是当前 branch 太长问题，不是切换当前 branch，所以不是一个东西”，并能说明 Branch Summary 摘要的是被切换的 branch 内容并带到新的 branch。