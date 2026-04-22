# Durable Memory Scopes And Overlays

## 职责

`DurableMemoryScopesAndOverlays` 定义 durable memory 的 scope axis。

它回答的是：

- durable memory 绑定到哪里
- 谁可以看到和更新这条 durable memory
- 是否共享、是否同步、是否与某个 agent identity 绑定

它不回答：

- 这条 durable memory 记的是什么语义
- recall pipeline 如何做 bounded selection
- consolidation 如何 merge 或 prune payload

因此本页讨论的是 overlay / binding，不是 payload taxonomy。

## 核心结论

durable memory 至少有两条正交维度：

1. `payload taxonomy`
   - `user / feedback / project / reference`
2. `scope / overlay binding`
   - `user / project / team / agent / local`

这两条轴可以出现同名项，但语义不同。

scope 改变的是 durable memory 的：

- visibility
- sharing
- storage root or persistence binding
- sync surface
- loading policy

scope 不改变的是 durable memory 的：

- payload taxonomy
- topic payload model
- recall pipeline 的基本分层
- consolidation 的核心职责

## Overlay Family

推荐把 durable memory 的 overlay family 至少建模为：

- `user`
- `project`
- `team`
- `agent`
- `local`

这些 overlay 都不是新的 durable plane。

## 与其它子规范的边界

- 与 [durable-memory-overview.md](durable-memory-overview.md)
  overview 定义 durable memory 总模型；本页只定义 scope axis
- 与 [memory-types/README.md](memory-types/README.md)
  `memory-types/` 定义 payload taxonomy；本页定义 storage / visibility overlays
- 与 [operations/relevant-memory-selection.md](operations/relevant-memory-selection.md)
  recall 负责在 overlay 边界内选择和读取 durable memory；本页不定义 recall 算法
- 与 [operations/extract-memories.md](operations/extract-memories.md)
  extract 可以写入不同 overlay，但不拥有 overlay 定义本身
- 与 [scopes/README.md](scopes/README.md)
  各具体 overlay 页见该子目录

## 规范结论

- durable memory 必须把 `payload taxonomy` 与 `scope / overlay binding` 作为两条不同轴建模
- `team`、`agent`、`local` 以及 `user`、`project` 一类 binding 都属于 overlays，不是新的 durable plane
- 同名项如 `user`、`project` 出现在两条轴上时，必须明确消歧
