# Session

## 职责

`Session` 是 agent runtime 的 durable state boundary。

模块总览见 [../module-overview.md](../module-overview.md)，术语归属见 [../terminology-and-ownership.md](../terminology-and-ownership.md)。

它负责：

- 标识一条可持续运行的 agent 会话
- 维护 append-only transcript 与 branch tracing
- 维护 session-exclusive short-term continuity
- 持有 restore 所需的 durable inputs 与 resume snapshot
- 维护 observer-facing lifecycle / progress state
- 为 runtime 提供可恢复的 working state

`Session` 的 6 层模型固定为：

```text
Session
├─ Transcript
├─ Short-Term Memory
├─ Working State
├─ Resume / Restore
├─ Lifecycle / Progress
└─ Sidechain / Subagent Transcript
```

## 核心边界

`Session` 拥有：

- transcript
- short-term memory
- working state
- resume / restore inputs
- lifecycle / progress state
- sidechain / subagent transcript linkage

`Session` 不拥有：

- query loop
- tool loop
- model sampling
- post-turn worker orchestration
- durable-memory store

因此必须明确：

- transcript 不是 memory
- short-term memory 不是 transcript 替代物
- lifecycle state 不是 working state
- resume 不是 transcript replay
- sidechain transcript 不是 durable memory

durable memory 见 [../durable-memory/README.md](../durable-memory/README.md)。
它是 shared package，不属于 `Session` 本体；`Session` 只提供 write-side 上游输入与 linkage。

## 稳定接口

推荐最小接口：

```text
SessionStore
  - create_session(session_metadata) -> session_id
  - append_events(session_id, events) -> session_checkpoint
  - get_session(session_id) -> session_metadata
  - get_events(session_id, selector) -> event_slice
  - get_runtime_state(session_id) -> runtime_state
  - get_working_state(session_id) -> working_state
  - wake(session_id) -> resume_snapshot
```

推荐标准对象：

```text
SessionCheckpoint
  - session_id
  - last_event_id
  - cursor
  - committed_at
```

```text
ResumeSnapshot
  - session_id
  - wake_id
  - checkpoint
  - transcript_slice
  - runtime_state
  - working_state
  - lifecycle_state?
  - short_term_memory?
  - branch_refs?
```

若实现了 session continuity 层，还应补：

```text
ShortTermMemoryStore
  - load(session_id) -> short_term_memory | null
  - update(session_id, transcript_delta, current_memory) -> updated_memory
  - get_coverage_boundary(session_id) -> transcript_cursor | null
  - wait_until_stable(session_id, timeout_ms) -> ready | timed_out
```

若实现支持 branch / sidechain，还应补：

```text
SessionBranchStore
  - create_branch(parent_session_id, branch_metadata) -> branch_ref
  - list_child_refs(session_id) -> child_refs
  - resolve_branch(ref) -> branch_handle
```

## 核心能力

宣称实现 `Session` 模块时，至少应具备：

- append-only transcript / event log
- session identity
- checkpoint / cursor
- resume snapshot
- working state restore
- structured lifecycle state
- compact boundary 后可继续恢复
- session-exclusive short-term memory
- single active harness lease
- sidechain / branch transcript linkage

## 标准扩展

推荐把下列能力作为 `Session` 标准扩展实现：

- `ShortTermSessionMemory`
  - continuity summary
  - compact-aware continuation
  - away / handoff summary
- `BranchAndSidechain`
  - subagent transcript
  - sidechain transcript
  - remote session link
- `SessionDurableMemoryLinkage`
  - transcript-backed write-side participation
  - session-local memory refs

这些扩展的重点是 durable boundary、restore 语义与外部可观察行为，而不是某一种实现算法。

## 产品级绑定关系

在当前产品关系里：

- `1 Chat = 1 Session`
- `1 Session = 1 Short-Term Memory`
- `1 Session @ any time = 1 Active Harness Lease`

同时：

- session transcript 中的 conversation content 可以成为 durable-memory write-side 上游
- 但 cross-session durable recall 不属于 `Session` 自己执行

## 默认实现映射

当前代码库中的默认 session 实现主要由这些语义层组成：

- append-only transcript
- branch / sidechain transcript
- short-term session memory
- restore sidecars
- working-state rebuild inputs
- lifecycle / progress projection

默认 `Local` 映射常见为本地 transcript 与 restore store；
默认 `Cloud` 映射常见为远端 event log 与 restore store。

两者都必须满足：

- transcript 是 durable truth source
- resume 是 restore-and-rebind
- compact 后仍可追溯 durable history
- harness 重启不应导致 session continuity 丢失

## 子页

- [transcript.md](transcript.md)
- [event-log-schema.md](event-log-schema.md)
- [resume-and-restore.md](resume-and-restore.md)
- [working-state.md](working-state.md)
- [lifecycle-state.md](lifecycle-state.md)
- [sidechain-and-subagent-transcripts.md](sidechain-and-subagent-transcripts.md)
- [short-term-memory/README.md](short-term-memory/README.md)
- [../durable-memory/README.md](../durable-memory/README.md)

## 规范结论

- `Session` 是 agent runtime 的 durable state boundary，不是 query loop 本身
- `Transcript`、`Short-Term Memory`、`WorkingState`、`ResumeSnapshot`、`SessionLifecycleState`、`SidechainTranscript` 必须作为不同层次建模
- runtime 可以追加、恢复和消费 session state，但 ownership 仍在 `Session`
- durable memory 不应再作为 `Session` 的内嵌子目录或独占子域对待
