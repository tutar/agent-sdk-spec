# Session

## 职责

`Session` 是 agent runtime 的 durable state boundary。

它负责：

- 标识一条可持续运行的 agent 会话
- 维护 runtime state
- 持久化 append-only transcript
- 恢复可继续运行的 working state
- 挂载 session 内的短期连续性记忆
- 串联 durable memory、subagent branch、remote session 等外部引用

它也必须跨宿主形态稳定：

- `TUI`
  通常是本地 durable transcript + restore
- `Desktop`
  通常是本地 durable store，可能与 GUI/runtime 分离
- `Cloud`
  通常是远端 event log / restore store

规范上必须明确：

- transcript 不是 memory
- compact summary 不是 transcript
- `CLAUDE.md` / memory files 不是 session 本身
- agent memory 不是主 session transcript

## 稳定接口

推荐最小接口：

```text
SessionStore
  - create_session(session_metadata) -> session_id
  - emit_event(session_id, event)
  - get_session(session_id) -> session_metadata
  - get_events(session_id, selector) -> event_slice
  - get_runtime_state(session_id) -> runtime_state
  - get_working_state(session_id) -> working_state
  - wake(session_id) -> resumed_session_handle
```

若实现了 session 内记忆层，还应补：

```text
SessionMemoryStore
  - load_short_term_memory(session_id)
  - update_short_term_memory(session_id, transcript_delta)
  - list_memory_links(session_id)
```

推荐补充：

```text
SessionLifecycleController
  - get_state(session_id) -> lifecycle_state
  - set_state(session_id, lifecycle_state, details?)
  - subscribe(session_id, listener)
```

## 默认实现

当前代码库中的默认 session 实现由这些部分组成：

- [bootstrap/state.ts](../../cc/bootstrap/state.ts)
  identity
- [utils/sessionState.ts](../../cc/utils/sessionState.ts)
  runtime state
- [utils/sessionStorage.ts](../../cc/utils/sessionStorage.ts)
  transcript + sidechain storage
- [utils/sessionRestore.ts](../../cc/utils/sessionRestore.ts)
  restore
- [services/SessionMemory/sessionMemory.ts](../../cc/services/SessionMemory/sessionMemory.ts)
  short-term session memory
- [memdir/findRelevantMemories.ts](../../cc/memdir/findRelevantMemories.ts)
  durable memory recall linkage
- [tools/AgentTool/agentMemory.ts](../../cc/tools/AgentTool/agentMemory.ts)
  agent-scoped durable memory

默认落点上：

- 主 transcript 用 append-only JSONL 持久化
- subagent 使用 sidechain transcript
- session memory 使用 session 级 continuity summary
- durable memory 通过 recall/linkage 接入，而不承担 restore 责任

## 要解决的问题

- 如何让会话在 harness 崩溃后仍可恢复
- 如何把当前 prompt window、raw transcript、working state、session memory 区分开
- 如何在长会话中保留 continuity，而不依赖全量重放 transcript
- 如何把跨会话 durable memory 接进来，但不与 session restore 混淆
- 如何兼容 sidechain transcript、subagent transcript、remote session 和 task state
- 如何让本地 session 与远端 session 共享同一语义边界

## 核心边界

- `Session`
  管 durable state、restore、memory linkage
- `Harness`
  读取 session 切片并构造当前 model input
- `ContextProvider`
  把 durable memory / project memory 注入当前 turn
- `ContextGovernance`
  决定 compact，不等于 session 自身
- `SessionLifecycleState`
  对外暴露 `idle / running / requires_action` 语义，不等于 transcript 或 UI state

session 域内的 memory 详细模型见 [memory-model.md](memory-model.md)。

## 子代理与 remote session

session 体系必须兼容：

- sidechain transcript
- subagent transcript
- teammate transcript
- remote agent session / task link

要求：

- 子对象必须可追溯到父 session
- 子对象可以独立落盘
- restore 时必须恢复引用关系

## 目录内文档

- [event-log-schema.md](event-log-schema.md)
- [lifecycle-state.md](lifecycle-state.md)
- [memory-model.md](memory-model.md)
- [memory-consolidation.md](memory-consolidation.md)
- [dream-consolidation.md](dream-consolidation.md)

## 规范结论

- `Session` 是 durable state boundary，不是 message array
- `Session` 模块主要负责 transcript、restore、working state 和 session memory linkage
- 长短期记忆的详细规范应下沉到独立文档，而不是挤在总览页里
- session 语义不应因为 TUI、Desktop、Cloud 的落盘位置不同而漂移
