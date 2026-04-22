# Auto Memory Runtime

## 职责

`AutoMemoryRuntime` 定义 durable memory 默认 operating mode 的 enable、scope、root 和 hosting 语义。

它回答的是：

- 何时整个 auto-memory runtime 被视为开启
- durable memory 默认根路径如何建模
- project-scoped default runtime 与其它 scope 的关系是什么
- `Local` 与 `Cloud` 下这个 runtime 如何保持语义一致

## 核心语义

### 1. default durable runtime

`AutoMemoryRuntime` 应被视为 durable memory 的默认 local-first runtime。

它通常负责：

- 提供 project-scoped durable root
- 提供 resident index 的默认位置
- 提供 bounded recall 的默认候选来源
- 承载 foreground write、extract、dream 的默认落点

### 2. project-scoped root

推荐最小对象：

```text
AutoDurableMemoryConfig
  - enabled
  - memory_scope: project
  - memory_root
  - index_mode
  - write_mode
  - recall_policy
```

默认情况下：

- root 应 project-scoped
- 不应按每个 turn 或每个临时 cwd 分裂
- worktree / equivalent project views 应允许共享同一 durable memory root

### 3. enable / disable gate

`AutoMemoryRuntime` 应允许被整体关闭。

关闭后至少应同时影响：

- resident auto-memory prompt
- bounded durable recall prefetch
- turn-end extraction
- background dream consolidation
- explicit remember / forget surfaces

### 4. Local-first + Cloud-compatible

- `Local`
  常见是本地 file-backed root + direct file access
- `Cloud`
  常见是远端 durable root + transport-neutral read/write bindings

差异只在 binding，不在 runtime 语义。

## 与其它页面的边界

- 与 [durable-memory-scopes-and-overlays.md](durable-memory-scopes-and-overlays.md)
  `AutoMemoryRuntime` 提供默认 project-scoped runtime；scope 本身仍由 overlays 定义
- 与 [operations/relevant-memory-selection.md](operations/relevant-memory-selection.md)
  recall 语义在上层保持 provider-neutral；本页只定义默认 runtime 如何提供 recall input
- 与 [operations/extract-memories.md](operations/extract-memories.md)
  extract 仍是独立能力；本页只定义它在默认 runtime 下的落点
- 与 [storage/README.md](storage/README.md)
  resident index、topic payload、header manifest 的默认落点见 storage 组

## 默认本地映射

当前默认本地映射通常表现为：

- project-scoped durable root
- resident index
- markdown topic payload
- turn-end extraction
- background dream consolidation

## 规范结论

- `AutoMemoryRuntime` 是默认 runtime 映射，不是新的 memory plane
- 它定义默认 root、gates 和 operating mode
- 它不改变 durable memory 的 ownership、taxonomy 或 overlay 语义
