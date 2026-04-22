# Durable Memory Write And Consolidation

## 职责

`DurableMemoryWriteAndConsolidation` 定义 durable memory 的写侧：如何捕获 durable signal、写入 durable target、以及如何在后续做整理与压实。

它不同于：

- `session transcript`
  原始会话事件
- `short-term session memory`
  面向 continuity 的短期摘要
- `compact`
  面向上下文窗口治理的重组
- `durable recall`
  面向已存在 durable store 的读侧选择

## 核心结论

durable memory 的写侧至少应允许三条可区分的路径：

1. `direct write`
   - 当前主路径显式写入 durable memory
2. `turn-end extraction`
   - turn 结束后补提取 durable signal
3. `cross-session consolidation`
   - 对多个 session 和记忆存量做 merge / prune / tighten

这三条路径可以协作，但不应被混成单一不可区分操作。

## 稳定接口

推荐最小接口：

```text
MemoryExtractionRequest
  - session_ref
  - transcript_slice
  - extraction_policy?

MemoryConsolidationRequest
  - memory_scope
  - source_sessions[]
  - existing_memory_refs[]
  - consolidation_policy?

DreamConsolidationRequest
  - trigger: manual | automatic
  - memory_scope
  - source_sessions[]
  - existing_memory_refs[]
  - index_ref?
  - consolidation_policy?

DurableMemoryWriter
  - extract(request) -> extracted_memories
  - consolidate(request) -> consolidated_memories
  - commit(memories) -> memory_refs
  - dream(request) -> dream_result
```

建议补充状态接口：

```text
ConsolidationState
  - last_consolidated_at
  - in_progress
  - source_cursor?
  - lock_ref?
```

推荐标准结果对象：

```text
ConsolidationResult
  - updated_memory_refs[]
  - pruned_memory_refs[]
  - updated_index_ref?
  - summary?
  - status: completed | no_change | failed | killed
```

## 典型 durable target

默认 durable target 应建模为：

- topic memory records / files
- resident entrypoint index 的更新或压实
- taxonomy-preserving durable payload rewrite

其中：

- `memory-types/`
  定义 payload 的语义分类
- `auto-memory/topic-memory.md`
  定义默认本地 payload 形态
- `auto-memory/memory-entrypoint-index.md`
  定义默认本地 index 形态

assistant-style append-first mode 可以改变 capture path，但不改变这些 durable target 的语义。

## 默认实现映射

当前代码库里的默认写侧体系可抽象为：

- direct write
  - 主路径显式写 memory files / index
- extraction
  - turn 结束后补提取 durable signal
- dream
  - 跨 session consolidation、dedupe、stale repair、index tightening

从默认实现可以看出：

- extraction 是 session-level incremental capture
- dream / consolidation 是 cross-session rewrite and tightening
- manual 与 automatic dream 共用同一 consolidation 目标语义

## 状态与恢复语义

`ConsolidationState` 应被视为 durable coordination state，而不是纯进程内标志。

至少应满足：

- `source_cursor`
  可与 `SessionCheckpoint` 或 memory index 语义对齐
- `lock_ref`
  可用于并发去重、crash recovery 或过期判断
- `in_progress`
  不能作为唯一真源；进程崩溃后应允许重建真实状态

如果 consolidation 在运行中断：

- session transcript / checkpoint / resume 不得受损
- 未 commit 的 memory 不得被当作 durable memory 生效
- 下次重试应能基于 `source_cursor` 或等价状态继续，而不是无约束重跑

## Failure And Delay Semantics

durable memory write-side 是标准扩展，不是 session restore 前提。

因此必须保持：

- extraction 成功但 consolidate 失败时，session 正常继续
- consolidate 成功但 commit 失败时，durable memory view 不得进入部分提交状态
- background consolidation 延迟执行时，recall 可以读到旧版本 memory，但不能读到未提交 memory
- manual `/dream` 与 `autoDream` 必须共享同一成功/失败结果语义

禁止行为：

- 通过改写历史 transcript 模拟 consolidation 成功
- 用 process-local state 假装 memory 已 durable commit
- 将 consolidation 错误上升为 session 不可恢复错误，除非 durable store 本身已损坏且实现明确这样建模

## 与其它子规范的边界

- 与 [durable-memory-architecture.md](durable-memory-architecture.md)
  本页只定义 durable 写侧，不定义 durable-memory 总模型
- 与 [durable-memory-recall-pipeline.md](durable-memory-recall-pipeline.md)
  recall 是读侧；本页是写侧
- 与 [dream-consolidation.md](dream-consolidation.md)
  dream 是本页 consolidation 的后台 / cross-session 模式
- 与 [../../harness/context-engineering/instruction-markdown/README.md](../../harness/context-engineering/instruction-markdown/README.md)
  instruction markdown loading 是 harness-level context input，不是 durable store write pipeline
- 与 [durable-memory-scopes-and-overlays.md](durable-memory-scopes-and-overlays.md)
  scope 可以影响写入根和共享边界，但不重新定义写侧职责
- 与 [scopes/README.md](scopes/README.md)
  `user / project / team / agent / local` overlays 可以改变 write destination、visibility 与 sync surface，但不改变 consolidation 的核心职责

## Default Local Mapping

auto-memory 下的默认写入/整理路径见 [auto-memory/auto-memory-write-paths.md](auto-memory/auto-memory-write-paths.md)。

topic memory 作为 auto-memory consolidation 的典型 durable target，见 [auto-memory/topic-memory.md](auto-memory/topic-memory.md)。

不同 durable memory types 的 staleness 与 consolidation 特征见 [memory-types/README.md](memory-types/README.md)。

## 规范结论

- durable memory write-side 应作为 session 域内独立能力存在
- direct write、extraction 与 consolidation 应分为三条可区分路径
- `manual /dream` 与 `autoDream` 应被视为同一 consolidation 能力的两种触发方式
- consolidation 可以后台执行，但状态与锁语义应显式建模
- durable memory write-side 不等于 compact，也不等于 verification
- `ConsolidationResult`、`ConsolidationState` 与 `DurableMemoryRecord` 应共同形成可恢复的长期记忆语义
- assistant agent 这类长期运行 profile 可以采用 append-first / consolidate-later 的 operating mode，但不改变 durable memory 的归属与 consolidation 语义；profile-specific 运行形态见 [../../harness/agent-profiles/assistant-agent.md](../../harness/agent-profiles/assistant-agent.md)，dream-specific trigger/runtime 见 [dream-consolidation.md](dream-consolidation.md)
