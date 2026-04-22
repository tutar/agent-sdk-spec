# Agent Memory Overlay

## 职责

`AgentMemoryOverlay` 定义 agent identity 或 agent type scoped durable memory binding。

它回答的是：

- durable memory 是否跟随某个 agent identity / agent type
- 同一 agent across sessions 是否共享 specialized durable memory
- agent-specific memory 绑定到哪个 backing scope

它不是：

- teammate mailbox
- per-turn worker state
- 新的 payload taxonomy

## 核心语义

`AgentMemoryOverlay` 的关键语义是：

- role memory
- specialization memory
- agent identity continuity

它复用同一 durable-memory：

- payload taxonomy
- resident entrypoint / index
- bounded recall
- write and consolidation semantics

它额外绑定：

- agent identity / agent type
- backing scope

## Backing Scope

`AgentMemoryOverlay` 推荐允许绑定到不同 backing scope，例如：

- user-scoped backing
- project-scoped backing
- local backing

也就是说，agent-scoped durable memory 与 `user / project / local` overlay 可以组合，而不是互斥。

## 与其它 overlays 的边界

- 与 [user-memory.md](user-memory.md)
  `user overlay` 关注 user ownership；`agent overlay` 关注 agent identity continuity
- 与 [project-memory.md](project-memory.md)
  `project overlay` 关注 project binding；`agent overlay` 关注 agent-specific specialization
- 与 [local-memory.md](local-memory.md)
  `local overlay` 可以作为 agent memory 的 backing scope，但不是 agent overlay 本身

## Default Local Mapping

默认本地映射中，`AgentMemoryOverlay` 常见为：

- agent-type scoped durable subtree
- 可绑定到 user / project / local backing root

规范不强制具体路径，只要求 agent identity scoped durable continuity 语义稳定。
