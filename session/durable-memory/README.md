# Durable Memory

## 职责

`Durable Memory` 定义 `Session` 域中的跨 session 可召回知识层。

它负责：

- durable memory model
- recall
- file-backed injection
- scope
- consolidation / dream
- default runtime mapping

它不负责：

- transcript restore
- session-exclusive short-term continuity

## 核心结论

durable memory 不应被建模成单一文件或单一路径。

推荐至少分为：

- durable model
- recall
- injection
- scope
- consolidation
- default runtime (`auto-memory`)
- taxonomy (`memory-types`)

其中：

- `auto-memory`
  回答默认 durable runtime 如何运作
- `memory-types`
  回答 durable payload 按什么语义分类

## 子页

- [durable-memory-model.md](durable-memory-model.md)
- [durable-memory-recall.md](durable-memory-recall.md)
- [memory-injection.md](memory-injection.md)
- [scoped-durable-memory.md](scoped-durable-memory.md)
- [memory-consolidation.md](memory-consolidation.md)
- [dream-consolidation.md](dream-consolidation.md)
- [auto-memory/README.md](auto-memory/README.md)
- [memory-types/README.md](memory-types/README.md)
