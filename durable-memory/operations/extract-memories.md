# Extract Memories

## 职责

`ExtractMemories` 定义 turn 结束后如何从 model-visible conversation 中补提取 durable signal。

它不同于：

- `direct write`
  主路径显式 durable write
- `dream`
  cross-session consolidation
- `compact`
  上下文窗口治理
- `durable recall`
  已存在 durable store 的读侧选择

## 核心结论

extract 是 durable memory 写侧的第二条正式路径。

它的直接输入是 session transcript 中的 model-visible conversation，而不是整个 session 对象。

## 稳定接口

推荐最小接口：

```text
MemoryExtractionRequest
  - session_ref
  - transcript_slice
  - extraction_policy?

DurableMemoryExtractor
  - extract(request) -> extracted_memories
  - commit(memories) -> memory_refs
```

## 典型 durable target

默认 durable target 应建模为：

- topic memory records / files
- resident entrypoint index 的更新或压实
- taxonomy-preserving durable payload update

其中：

- `memory-types/`
  定义 payload 的语义分类
- [../storage/topic-memory.md](../storage/topic-memory.md)
  定义默认本地 payload 形态
- [../storage/memory-md.md](../storage/memory-md.md)
  定义默认本地 index 形态

## 默认实现映射

默认写侧体系可抽象为：

- direct write
  主路径显式写 memory files / index
- extract
  turn 结束后补提取 durable signal
- dream
  跨 session consolidation、dedupe、stale repair、index tightening

extract 是 session-level incremental capture。

## Failure And Delay Semantics

extract 可以异步执行，但不得破坏 session 可继续性。

因此必须保持：

- foreground turn 已结束，extract 才能开始
- direct write 已覆盖的语义范围，extract 允许跳过
- extract 失败不应破坏 transcript、checkpoint 或 resume
- extract 成功但后续 dream 延迟时，recall 可以读到旧版本 durable store，但不能读到未提交 memory

禁止行为：

- 通过改写历史 transcript 模拟 extract 成功
- 用 process-local state 假装 memory 已 durable commit
- 将 extract 错误上升为 session 不可恢复错误，除非 durable store 本身已损坏且实现明确这样建模

## 与其它子规范的边界

- 与 [../durable-memory-overview.md](../durable-memory-overview.md)
  本页只定义 write-side 中的 turn-end extraction
- 与 [relevant-memory-selection.md](relevant-memory-selection.md)
  recall 是读侧；本页是写侧
- 与 [auto-dream.md](auto-dream.md)
  dream 是 cross-session consolidation mode，不是 turn-end extraction
- 与 [../../harness/context-engineering/instruction-markdown/README.md](../../harness/context-engineering/instruction-markdown/README.md)
  instruction markdown loading 不是 durable store write pipeline
- 与 [../durable-memory-scopes-and-overlays.md](../durable-memory-scopes-and-overlays.md)
  scope 可以影响写入根和共享边界，但不重新定义写侧职责
- 与 [../scopes/README.md](../scopes/README.md)
  overlays 可以改变 write destination、visibility 与 sync surface，但不改变 extract 的核心语义

## 规范结论

- extract 是 durable memory 写侧的正式路径
- extract 的主要输入是 model-visible conversation
- extract 不等于 direct write，也不等于 dream consolidation
- extract 可以后台执行，但失败不得破坏 session restore
