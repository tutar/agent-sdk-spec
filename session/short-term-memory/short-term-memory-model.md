# Short-Term Memory Model

## 目标

本规范定义 `Session` 域中的短期记忆模型。

runtime 在 compact、resume、turn handoff 等场景消费这层能力。

这里的 short-term memory 不是：

- transcript
- prompt window
- durable memory

它是当前 session 的 continuity object。

## 核心原则

- `1 Session = 1 Short-Term Memory`
- short-term memory 服务于 continuity，不服务于跨 session recall
- short-term memory 可以消费 transcript，但不能替代 transcript
- harness 重启不应导致 session 绑定的 short-term memory 丢失

## 稳定语义

### 1. continuity object

`ShortTermSessionMemory` 应维护当前 session 的压缩连续性表示。

它典型负责：

- 当前任务目标与阶段性进展
- 尚未闭合的局部结论
- 最近关键决策、风险与约束
- compact 后仍需保留的 continuity signal

### 2. not a restore source

short-term memory 可以辅助 `resume`，但不是恢复真源。

因此：

- transcript 仍然是 durable restore 的真源
- compact boundary 后允许 short-term memory 辅助 continuation
- 不允许通过 short-term memory 取代 transcript 的可追溯性

### 3. continuity, not lifecycle

short-term memory 服务于 continuity。

它不应承担：

- lifecycle / progress projection
- observer-facing status signaling
- pending action binding restore

### 4. session-exclusive

一个 short-term memory 只能服务一个 session。

不允许：

- 多个 session 共享同一个 short-term memory
- 把 short-term memory 误当作 agent-global long-term memory

## 推荐最小接口

```text
ShortTermMemoryStore
  - load(session_id) -> short_term_memory | null
  - update(session_id, transcript_delta, current_memory) -> updated_memory
  - get_coverage_boundary(session_id) -> transcript_cursor | null
  - wait_until_stable(session_id, timeout_ms) -> ready | timed_out
```

## 与其它子规范的边界

- 与 [../../durable-memory/README.md](../../durable-memory/README.md)
  durable memory 服务跨 session recall；short-term memory 服务当前 session continuity
- 与 [../transcript.md](../transcript.md)
  transcript 是 durable truth source；short-term memory 只是 continuity layer
- 与 [continuity-summary.md](continuity-summary.md)
  continuity summary 是 short-term memory 的主要内容形态之一
- 与 [coverage-boundary-and-stability.md](coverage-boundary-and-stability.md)
  coverage boundary 与稳定性约束见该页
