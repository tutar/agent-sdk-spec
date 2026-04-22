# Durable Memory

## 职责

`Durable Memory` 定义 `Session` 域中的跨 session 可召回知识层。

它负责：

- durable memory model
- recall
- scope
- consolidation / dream
- default runtime mapping

它不负责：

- transcript restore
- session-exclusive short-term continuity

## 核心结论

durable memory 不应被建模成单一文件或单一路径。

推荐至少分为：

- durable architecture
- recall pipeline
- scopes and overlays
- write and consolidation
- dream consolidation
- default runtime (`auto-memory`)
- taxonomy (`memory-types`)

其中：

- `auto-memory`
  回答默认 durable runtime 如何运作
- `memory-types`
  回答 durable payload 按什么语义分类
- `scopes/`
  回答 durable payload 绑定到哪里、谁可见、如何共享与同步

durable-memory 子域还要求清楚区分：

- resident entrypoint/index
- topic payload
- header manifest
- scope overlay
- write-side capture 与 background consolidation

这些概念若被混写，容易导致：

- 把 dream 误写成独立 memory plane
- 把 `AGENTS.md` / `rules` 混成 durable recall
- 把 `team` / `agent` memory 写成新的 runtime
- 把 memory types 写成 runtime 而不是 payload taxonomy
- 把 `user / project` 的 payload semantics 与 overlay binding 混成一条轴

## 子页

- [durable-memory-architecture.md](durable-memory-architecture.md)
- [durable-memory-recall-pipeline.md](durable-memory-recall-pipeline.md)
- [durable-memory-scopes-and-overlays.md](durable-memory-scopes-and-overlays.md)
- [durable-memory-write-and-consolidation.md](durable-memory-write-and-consolidation.md)
- [dream-consolidation.md](dream-consolidation.md)
- [auto-memory/README.md](auto-memory/README.md)
- [memory-types/README.md](memory-types/README.md)
- [scopes/README.md](scopes/README.md)
