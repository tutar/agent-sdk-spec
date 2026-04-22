# Auto Memory Write Paths

## 职责

`AutoMemoryWritePaths` 定义 auto-memory runtime 下 durable memory 的默认写入与整理路径。

它的重点不是具体工具，而是写入语义如何分层：

- direct write
- background extraction
- background consolidation
- assistant-style append-first variant

## 核心结论

`AutoMemory` 的 durable write 不应被建模成单一路径。

推荐至少区分：

- `DirectDurableWrite`
  主交互路径中的显式 durable write
- `BackgroundExtraction`
  turn 结束后的增量 durable extraction
- `DreamConsolidation`
  cross-session durable consolidation

这些路径可以协作，但不应互相伪装成同一操作。

## 稳定语义

### 1. direct write

主 runtime 应允许模型直接创建或更新 durable memory。

典型场景：

- 用户明确要求 remember
- 当前 turn 已形成稳定 durable knowledge

默认 durable target 通常是 topic memory records，而不是 resident entrypoint 本体。

### 2. background extraction

runtime 应允许在完整语义边界之后进行增量 durable extraction。

约束：

- extraction 不应在未闭合的 tool/use chain 中间触发
- 若主路径已完成 durable write，background extraction 应允许跳过重复范围
- extraction 可以异步执行，但不得破坏 session 可继续性

### 3. dream consolidation

dream consolidation 应负责：

- merge duplicate durable memory
- prune stale or contradicted records
- rebuild or tighten the resident index

它不是普通 extraction，也不应被建模成 transcript rewrite。

dream 详细语义见 [../dream-consolidation.md](../dream-consolidation.md)。

### 4. assistant-style append-first variant

某些 long-lived host profile 可将 auto-memory 切换为 append-first variant：

- 新记忆先进入 append-only capture stream
- resident index 与 topic records 延后由 consolidation 更新
- delayed consolidation 不应破坏 recall / continuity / viewer reattach

这是一种 operating mode 变化，不是新的 durable memory plane。

## 与其它子规范的边界

- 与 [../durable-memory-write-and-consolidation.md](../durable-memory-write-and-consolidation.md)
  consolidation 是上层能力；本页只定义 auto-memory 下默认写入路径如何接入它
- 与 [../dream-consolidation.md](../dream-consolidation.md)
  dream 是 auto-memory 的标准 consolidation path，不是 auto-memory 的全部
- 与 [../../../harness/agent-profiles/assistant-agent.md](../../../harness/agent-profiles/assistant-agent.md)
  assistant agent profile 可以偏好 append-first / consolidate-later，但不拥有 durable memory 本体
- 与 [../durable-memory-scopes-and-overlays.md](../durable-memory-scopes-and-overlays.md)
  team memory / agent memory 是 scoped extension，不改变这些写入路径的上层语义
- 与 [topic-memory.md](topic-memory.md)
  direct write、background extraction、dream consolidation 的默认 durable payload target 见该页
- 与 [memory-entrypoint-index.md](memory-entrypoint-index.md)
  resident index 可以被这些路径更新，但不应被误当作 durable正文层

## Default Local Mapping

当前默认本地映射通常表现为：

- prompt-taught direct write
- turn-end background extraction
- background dream consolidation
- assistant-style append-only daily log + later dream
- topic memory files 作为 durable payload target，`MEMORY.md` 或等价物作为 resident entrypoint
