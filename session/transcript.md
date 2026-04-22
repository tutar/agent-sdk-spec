# Transcript

## 职责

`Transcript` 定义 `Session` 的 append-only durable truth source。

它被 runtime 在 turn 推进过程中追加，被 resume / restore 路径读取，并为 compact boundary、branch tracing 与 durable replay 提供真源。

它不负责：

- continuity summary
- lifecycle projection
- working state
- cross-session durable memory

## 最小对象

推荐最小对象：

```text
TranscriptEntry
  - entry_id
  - session_id
  - parent_entry_id?
  - logical_parent_entry_id?
  - entry_type
  - timestamp
  - branch_ref?
  - payload
```

`entry_type` 至少应能表达：

- `user_message`
- `assistant_message`
- `system_message`
- `attachment`
- `compact_boundary`
- `branch_link`
- `resume_marker`
- `tool_result_reference`

## 稳定语义

### 1. append-only durable truth source

transcript 必须是 append-only durable history。

它记录的是已经发生过、且对 durable replay / restore 有意义的事件；
它不是 prompt window，也不是“当前模型看到的消息数组”。

### 2. transcript 不是 memory

transcript 不得被误写成：

- short-term continuity summary
- durable memory payload
- lifecycle status channel

即使 compact 后 runtime 不再直接把全部 transcript 原文送入模型，
transcript 仍然保持 durable truth-source 地位。

### 3. compact boundary 是 transcript 中的 durable 边界

compact / rewrite / summary 可以改变后续 continuation 的工作视图，
但不应替代 transcript 本身。

因此实现至少应能在 transcript 中表达：

- compact boundary
- compact 后的恢复位置
- compact 前 durable history 的可追溯性

### 4. transcript 为 restore 提供核心输入

resume / wake / restore 至少应以 transcript 作为核心输入真源之一。

恢复链可以在 transcript 之上执行：

- chain reconstruction
- branch selection
- interruption detection
- message normalization

但不得把 “恢复消息数组” 简化成 “重放 transcript 文本”。

### 5. transcript 与 sidechain 的关系

主 transcript 不是唯一会话历史。

`Session` 还可以拥有：

- sidechain transcript
- subagent transcript
- remote branch transcript

这些附属 transcript 仍属于 session 的 branch tracing 层，而不是另一个 memory store。

## 与相邻页面的边界

- 与 [event-log-schema.md](event-log-schema.md)
  本页定义 transcript 的 durable 语义；event-log-schema 定义标准对象和读写选择器
- 与 [working-state.md](working-state.md)
  transcript 是 truth source；working state 是 continuation runtime state
- 与 [short-term-memory/README.md](short-term-memory/README.md)
  transcript 是 durable history；short-term memory 是 continuity object
- 与 [sidechain-and-subagent-transcripts.md](sidechain-and-subagent-transcripts.md)
  本页只定义 transcript 的总边界；branch tracing 细节见该页

## 规范结论

- `Transcript` 必须被建模为 `Session` 的 append-only durable truth source
- transcript 不得退化成 memory、working state 或 lifecycle projection 的替代物
- compact boundary、branch linkage、resume markers 应作为 transcript 的 durable semantics 表达
