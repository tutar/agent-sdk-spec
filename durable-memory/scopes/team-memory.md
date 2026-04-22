# Team Memory Overlay

## 职责

`TeamMemoryOverlay` 定义 shared team-scoped durable memory binding。

它回答的是：

- 哪些 durable payload 对项目成员共享可见
- 是否存在 team-level sync surface
- team-shared durable store 如何叠加到默认 project durable runtime 之上

它不是：

- 新的 durable plane
- 新的 payload taxonomy
- teammate mailbox 或 runtime working state

## 核心语义

`TeamMemoryOverlay` 应被视为 shared overlay。

它复用同一 durable-memory：

- payload taxonomy
- resident entrypoint / index
- bounded recall pipeline
- write and consolidation semantics

它改变的是：

- contributor set
- visibility
- sync surface
- shared write destination

## 默认共享语义

推荐默认语义：

- team-visible
- project-associated
- shared durable store
- 可带 sync semantics

实现可以采用不同同步协议，但不得把 `TeamMemoryOverlay` 重新建模成独立的 payload system。

## 与其它 overlays 的边界

- 与 [project-memory.md](project-memory.md)
  `project overlay` 关注 project binding；`team overlay` 关注 shared visibility 与 sync
- 与 [user-memory.md](user-memory.md)
  `user overlay` 是 private；`team overlay` 是 shared
- 与 [agent-memory.md](agent-memory.md)
  `agent overlay` 关注 agent identity continuity；`team overlay` 关注跨成员共享

## Default Local Mapping

默认本地映射中，`TeamMemoryOverlay` 常见为：

- project durable store 下的 shared subtree
- repo/project identity-based sync surface
- pull-before-use、delta-style push、watcher-driven propagation 的语义等价实现

规范不强制这些具体机制，但要求 shared overlay 与 sync surface 可区分。
