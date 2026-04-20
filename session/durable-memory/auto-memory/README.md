# Auto Memory

## 职责

`AutoMemory` 定义 `Session.memory` 的默认 local-first durable memory operating mode。

它不是新的 memory plane，也不是新的顶层模块。

它负责收敛以下组合能力：

- resident instructions
- resident index
- bounded durable recall
- foreground durable write
- turn-end extraction
- background consolidation
- assistant-style append-first variant

它不负责：

- short-term session memory
- transcript restore
- `AGENTS.md` 一类显式 durable injection source
- team-shared durable memory 的同步协议本体

## 核心结论

`AutoMemory` 更像 durable memory 的默认 runtime，而不是单个 store。

它至少要求：

- durable memory 有稳定的 project-scoped root
- resident index 与 recalled payload 语义分离
- resident entrypoint、topic payload、header manifest 是可区分层
- direct write、extraction、dream consolidation 是不同但可协作的写入路径
- assistant-style host profile 可将写入模式切换为 append-first / consolidate-later

## 子页

- [auto-memory-runtime.md](auto-memory-runtime.md)
- [auto-memory-prompt-and-index.md](auto-memory-prompt-and-index.md)
- [memory-entrypoint-index.md](memory-entrypoint-index.md)
- [topic-memory.md](topic-memory.md)
- [frontmatter-and-header-manifest.md](frontmatter-and-header-manifest.md)
- [auto-memory-write-paths.md](auto-memory-write-paths.md)
