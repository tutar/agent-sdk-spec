# Local Memory Overlay

## 职责

`LocalMemoryOverlay` 定义 host-local 或 workspace-local 的 private durable memory binding。

它回答的是：

- durable memory 是否仅在当前机器、当前宿主或当前 workspace 有效
- 是否不参与 team sync
- 是否只作为本地 private overlay 存在

它不是：

- `memory-types/user` 的私有化别名
- team-shared memory
- agent identity 本身

## 核心语义

`LocalMemoryOverlay` 的默认语义是：

- non-shared
- non-team-synced
- environment-bound or machine-bound
- workspace-local 或 host-local

它可以承载多种 payload type，也可以作为 `AgentMemoryOverlay` 的 backing scope。

## 与其它 overlays 的边界

- 与 [user-memory.md](user-memory.md)
  `user overlay` 强调 user ownership；`local overlay` 强调 host/workspace locality
- 与 [project-memory.md](project-memory.md)
  `project overlay` 关注 project binding；`local overlay` 关注 local-only persistence
- 与 [agent-memory.md](agent-memory.md)
  `agent overlay` 关注 agent identity；`local overlay` 可作为其 backing scope

## 与 payload taxonomy 的边界

`LocalMemoryOverlay` 不是 payload type。

因此：

- 一条 `memory-types/user` payload 可以落在 local overlay 中
- 一条 `feedback` payload 也可以落在 local overlay 中

决定它是否属于 `local` 的是 binding，而不是 payload semantics。

## Default Local Mapping

默认本地映射中，`LocalMemoryOverlay` 常见为：

- workspace-local durable subtree
- machine-local private backing root
- non-versioned private store

规范不强制具体文件树，只要求 local-only、non-shared、non-team-synced 的语义稳定。
